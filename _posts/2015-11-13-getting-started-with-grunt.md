---
layout: post
title: Getting started with Grunt
description: >
  An introduction to setting up GruntJS tasks from a beginners point of view.
  This includes installing Grunt and installing and configuring the tasks.

---

After [setting up RequireJS](/getting-started-with-requiresj/), I decided look
into automating my development workflow by using
[GruntJS](http://gruntjs.com/).  Getting up and running with Grunt is really
easy, just install the node module [node.js](https://nodejs.org/en/download/),
then run `npm i grunt --save-dev`.  Job done&hellip; congratulations.

You need to setup the project: in the project directory run `npm init`.  This
will walk you through the process of creating a `package.json` file, it
contains all of the metadata/dependencies for the project. I also added
`"private": "true"` to my package file as I didn't want to associate the
project with a Github repo. Now you can start installing some modules!

#### Grunt-replace

Initially I wanted something that could inline critical CSS into the head
automatically. After some googling, I discovered a node app called
[grunt-replace](https://github.com/outaTiME/grunt-replace) which could do
exactly that. To install `grunt-replace` run `npm install grunt-replace
--save-dev`.

Configuring grunt-replace is quite straight forward; create a `Gruntfile.js`
and add the following.

```
module.exports = function (grunt) {

  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),
    replace: {
      // We'll put more here later
    }
  });

  grunt.loadNpmTasks('grunt-replace');
  grunt.registerTask('default', ['replace']);
};
```

The grunt configuration must always be contained within `module.exports =
function (grunt) { //.. } `. The module configuration is then stored within
`grunt.initConfig({ //.. })`; here we have added a reference to the
`package.json` file and have also started the configuration for our
`grunt-replace` module.

Obviously there isn't much point configuring a module which hasn't been
loaded, that's where `grunt.loadNpmTasks('grunt-replace')` comes in. Finally
you need to register which modules will be run under which tasks. Here I've
defined two tasks, `default` which is for when you simply run `grunt`.

The specific configuration for the `grunt-replace` tasks is very straight
forward.

```
replace: {
  dist: {
    options: {
      patterns: [
        { // search pattern @@include_css_style_tag
          match: 'include_css_style_tag',
          // CSS file to replace it with
          replacement: '<%= grunt.file.read("css/style.css") %>'
        }
      ]
    },
    files: [
      {
        expand: true,
        flatten: true,
        src: ['_includes/head.html'],
        dest: '_includes/built/'
      }
    ]
  }
}
```

Using replacement patterns you can basically perform search and replace within
the file, and using `<%= grunt.file.read("css/style.css") %>` I've turned my
replacement string into an entire CSS file. Obviously I don't want to be
writing plain CSS. Which is where the sass pre-processor comes in.

#### Grunt-contrib-compass

Is a scss/compass compiler run by grunt. The configuration is super simple.

```
compass: {
  dist: {
    options: {
      sassDir: '_sass/',
      cssDir: 'css/',
      outputStyle: 'compressed'
    }
  }
}
```

#### Grunt-watch

Finally, because I'm too lazy to run the above after every change, I'm using
`grunt-watch` to automatically run `grunt-compass` and `grunt-replace` when a
watched file changes.

```
watch: {
  css: {                      // name of the first watch task
    files: ['_sass/*.scss'],  // watch files
    tasks: ['compass']        // task to run on change
  },
  replace: {                  // name of second watch task
    files: ['css/style.css'], // watch files
    tasks: ['replace']        // task to run on change
  }
}
```

The watch configuration will listen for changes in the sass file, when it
detects a change it will compile it into the css directory. The next watch
task is watching the compass output file, and will inline it into my html
whenever the css file is changed. It's possible to build a long chain of
events that are automated by watch.

Putting all of this together results in a `Gruntfile.js` that resembles the
following.

```
module.exports = function (grunt) {

  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),
    replace: {
      dist: {
        options: {
          patterns: [
            {
              match: 'include_css_style_tag',
              replacement: '<%= grunt.file.read("css/style.css") %>'
            }
          ]
        },
        files: [
          {
            expand: true,
            flatten: true,
            src: ['_includes/head.html'],
            dest: '_includes/built/'}
        ]
      }
    },
    compass: {
      dist: {
        options: {
          sassDir: '_sass/',
          cssDir: 'css/',
          outputStyle: 'compressed'
        }
      }
    },
    watch: {
      css: {
        files: ['_sass/*.scss'],
        tasks: ['compass']
      },
      replace: {
        files: ['css/style.css'],
        tasks: ['replace']
      }
    }
  });

  grunt.loadNpmTasks('grunt-replace');
  grunt.loadNpmTasks('grunt-contrib-watch');
  grunt.loadNpmTasks('grunt-contrib-compass');

  grunt.registerTask('default', ['watch']);
  grunt.registerTask('build', ['compass', 'replace']);
};
```
