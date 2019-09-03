---
layout: post
title: >
    Part 1: Setting up AWS CodeBuild to build a Jekyll site
description: >
    Part 1 takes a look at the process to setup the AWS CodeDeploy build phase
    used to build and deploy this site.
---

I recently decided to migrate my site from Github pages to AWS. To ensure it
remained hassle free on the deployment side, I decided to replicate the
deployment process that Github were already implementing.

```
Code change > git commit + push > Jekyll build > deploy to webserver
```

I've split the post into two parts, mostly to make it easier for me to write.
Part 1 will look at the build phase, then part 2 will look at the deployment
phase.

## CodeCommit
AWS will happily use Github's OAuth authentication to install repo hooks to
receive push notifications; however to keep things solely within the AWS
ecosystem I decided to host in [AWS CodeCommit][codecommit].

[codecommit]: https://aws.amazon.com/codecommit/

Setting up the new git repository is pretty straight forward, the only thing to
note is that you need to [upload your ssh key][aws-ssh-key] to your IAM profile.
After uploading the key, you create the repo and add it as a new remote in your
local repo.

[aws-ssh-key]: https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-ssh-unixes.html?icmpid=docs_acc_console_connect_np

## CodeBuild
The build phase is more interesting to setup. Each build is referred to as a
project, and within each project there are subsections for configuring processes
within the project.

### Project Configuration
Consists of naming the project, an optional description, and assigning any
relevant tags; not very interesting!

### Source
Here you choose where your source code is being hosted. Some of the options
include S3, Github, Bitbucket, however the one I am using is AWS CodeCommit. As
with most AWS console configuration - you can choose the source repository from
a pre populated dropdown containing a list of existing CodeCommit repositories.

![codebuild-source](/assets/images/codebuild-source.png)

### Environment
This, and the following Buildspec section, are what we are most interested in;
we need to setup a build environment to turn the Jekyll source into a static
site. The environments are all defined in Docker images, and you are given two
main options as to the source of your build host image:

- Managed Image

    A build image that is managed by AWS. You have two options, an Amazon Linux 2
    host, or Ubuntu 18.04 LTS (recommended). Choosing the Managed Image will
    subsequently allow you to choose from a [managed runtime][managed-runtime], in
    my case I chose `docker: 18`. This results in an Ubuntu 18.04 host running
    Docker 18. From there you can run docker commands using via the buildspec file.

[managed-runtime]: https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-available.html

- Custom Image

    I've not explored this option in practice, however it just allows you to setup a
    custom build host.

**Note:** if you don't set `Privilege = True` you will not be able to pull and build
docker images on the Ubuntu host.

The last part of the Environment section is to either choose an existing role to
assign to the project, or create a new one. I chose the later as this is the
first project I've setup, and I wasn't certain of the role requirements.

![codebuild-environment](/assets/images/codebuild-environment.png)

### Buildspec
Now that the build environment has been defined, we need to tell AWS how to
install dependencies, and run the build.

Again you have a couple choices as to how to go about this. The first is to just
type in a list of commands to the input box separated by `&&`.

![codebuild-buildspec-commands](/assets/images/codebuild-buildspec-commands.png)

This isn't really scalable, or conveniently stored in version control; your
other option is to store a [buildspec file][buildspec-reference] within the root
of your source directory. The buildspec contains, amongst other things, lists of
commands that are to be run in the environment.

[buildspec-reference]: https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html

The buildspec is broken down in several main sections (phases): install,
pre_build, build, post_build, artifacts, and cache.

My `buildspec.yml` contains the following.

```yaml
version: 0.2
phases:
  install:
    runtime-versions:
      docker: 18
    commands:
      - docker --version
      - docker pull jekyll/builder:3.8
  pre_build:
    commands:
      - chown -R 1000:1000 $CODEBUILD_SRC_DIR
  build:
    commands:
      - docker run --rm --volume="$CODEBUILD_SRC_DIR:/srv/jekyll" jekyll/builder:3.8 jekyll build
  post_build:
    commands:
      - ls -la
      - cp -a .ebextensions _site/
artifacts:
  files:
    - '**/*'
  base-directory: _site/
 ```

It took me several iterations to arrive at the above, and provides build times
of roughly 12-15 seconds.

The reason for the `chown` is that jekyll runs the build with user id 1000 by
default, while the container clones the source as root.

### Artifacts
The final section in my buildspec. Artifacts define build output
directories/files that should be stored in S3 for subsequent use. If you're
using CodeDeploy Pipelines then this section isn't strictly necessary as you
will override the Artifacts configured here with one created in the pipeline.
This one is useful for testing builds outside of a pipeline though.

In the case of the Jekyll build, the static site output is stored in `_site/`,
so this is what we can't to store.

## Optimisation
As I alluded to earlier, that there are some optimisations to be made when
setting up the build. As an example, I was initially using the Ubuntu 18.04
image with the Ruby 2.6 runtime. I then installed Jekyll via `gem install jekyll
bundler`.  However this involved compiling the C libraries which took close to 3
minutes.  After switching to the Docker 18 runtime, I was able to pull down the
jekyll/builder image and use that to build the project. This got the build time
down to around 55 seconds.

The next, and one of the most important steps when using the Docker runtime, was
to enable local Docker layer caching. Now whenever I run `docker pull
jekyll/builder` within the build environment, and there is not a newer image to
pull down, docker will use the locally cached image. After making this small
change I was able to get build times down to around 12 seconds on average.

![codebuild-docker-layer-cache](/assets/images/codebuild-docker-layer-cache.png)

According to the [AWS documentation][aws-local-cache], the local cache if only
available for a limited time, and is intended to be used for quick back to back
builds of large projects.

[aws-local-cache]: https://aws.amazon.com/blogs/devops/improve-build-performance-and-save-time-using-local-caching-in-aws-codebuild/

**Part 2 in progress.**
