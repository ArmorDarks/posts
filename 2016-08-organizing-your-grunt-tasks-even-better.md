---
author: Serj Lavrin (https://github.com/ArmorDarks)
created: 2016-08-10
status: WIP
keywords: [Grunt, Node, JavaScript, tasks, configs, organizing, refactoring]
---

# Organizing Your Grunt Tasks Even Better
*This is a guest post by Serj Lavrin. Serj takes a look at more ways you can organize a Grunt configuration. For example, splitting up tasks by what job they do rather than strictly by what plugin they use. That way, for example, it may be easier to follow the follow of what happens to a CSS file. Serj has lots of tips, data, and example code for us here, so take it away Serj!*

We live in an era of [Webpack](https://webpack.js.org/) and [NPM scripts](https://blog.teamtreehouse.com/use-npm-task-runner). Good or bad, they took a lead as a building and task running tools, with bits of [Rollup](https://rollupjs.org/), [JSPM](https://jspm.io/) and [Gulp](https://gulpjs.com/).

But let's face it. Some of your older projects still using good ol' [Grunt](#). No longer it glimmers brightly but does the job well so there's little reason to touch it.

Though, from time to time you wonder is there a way to make it better, right? Then start from [Organizing Your Grunt Tasks](https://css-tricks.com/organizing-grunt-tasks/) article and come back. I'll show you more.


## Automatic Speed Daemon tasks loading

First thing first. People hate writing loading declarations for each task:


    grunt.loadNpmTasks('grunt-contrib-clean')
    grunt.loadNpmTasks('grunt-contrib-watch')
    grunt.loadNpmTasks('grunt-csso')
    grunt.loadNpmTasks('grunt-postcss')
    grunt.loadNpmTasks('grunt-sass')
    grunt.loadNpmTasks('grunt-uncss')
    
    grunt.initConfig({})

It's common to use `[load-grunt-tasks](https://github.com/sindresorhus/load-grunt-tasks)` to load all tasks automatically instead.

But what if I tell you *there is a faster way*?

Just use `[jit-grunt](https://github.com/shootaroo/jit-grunt)`! Similar to `load-grunt-tasks`, but even faster than native `grunt.loadNpmTasks`.

In large projects the difference can be striking.

Without `jit-grunt`:


    loading tasks     5.7s  ▇▇▇▇▇▇▇▇ 84%
    assemble:compile  1.1s  ▇▇ 16%
    Total 6.8s

With `jit-grunt`:


    loading tasks     111ms  ▇ 8%
    loading assemble  221ms  ▇▇ 16%
    assemble:compile   1.1s  ▇▇▇▇▇▇▇▇ 77%
    Total 1.4s

Yes, 1.4s doesn't really make it a Speed Daemon... So I kinda lied. But still, it's 6 times faster than the traditional way!

If you're curious how that's possible, read about [original issue](https://github.com/gruntjs/grunt/issues/975) which lead to creation of `jit-grunt`.

So, how to use?

First, install:


    npm install jit-grunt --save

Then replace all your load statement with a single line:


    module.exports = function (grunt) {
      // Intead of this:
      // grunt.loadNpmTasks('grunt-contrib-clean')
      // grunt.loadNpmTasks('grunt-contrib-watch')
      // grunt.loadNpmTasks('grunt-csso')
      // grunt.loadNpmTasks('grunt-postcss')
      // grunt.loadNpmTasks('grunt-sass')
      // grunt.loadNpmTasks('grunt-uncss')
    
      // Or instead of this, if you've used `load-grunt-tasks`
      // require('load-grunt-tasks')(grunt, {
      //   scope: ['devDependencies', 'dependencies'] 
      // })
    
      // Use this:
      require('jit-grunt')(grunt)
    
      grunt.initConfig({})
    }

Done!


## Better configs loading

We told Grunt how to load tasks itself, but we didn't quite finish yet!

As [Organizing Your Grunt Tasks](https://css-tricks.com/organizing-grunt-tasks/) article suggest, one of the most useful things we're trying to do here is split up a monolithic Gruntfile into smaller standalone files.
If you read the mentioned article, you'll know it's better to move all task configuration into external files.

So, intead of single `gruntfile.js` file:


    module.exports = function (grunt) {
      require('jit-grunt')(grunt)
    
      grunt.initConfig({
        clean: {/* task configuration goes here */},
        watch: {/* task configuration goes here */},
        csso: {/* task configuration goes here */},
        postcss: {/* task configuration goes here */},
        sass: {/* task configuration goes here */},
        uncss: {/* task configuration goes here */}
      })
    }

We want this:


    tasks
      ├─ postcss.js
      ├─ concat.js
      ├─ cssmin.js
      ├─ jshint.js
      ├─ jsvalidate.js
      ├─ uglify.js
      ├─ watch.js
      └─ sass.js
    gruntfile.js

But that will force us to load each external configuration into `gruntfile.js` manually, and that takes time! We need a way to load our configuration files automatically.

For that purpose people use `[load-grunt-configs](https://github.com/creynders/load-grunt-configs)`. It takes a path, grabs all configuration files there and gives us merged config object wich we use for Grunt config initializaiton.

Here how it works:


    module.exports = function (grunt) {
      require('jit-grunt')(grunt)
    
      const configs = require('load-grunt-configs')(grunt, {
        config: { src: 'tasks/*.js' }
      })
    
      grunt.initConfig(configs)
      grunt.registerTask('default', ['cssmin'])
    }

But what if I tell you that *Grunt can do the same thing natively*?

Take a look at `[grunt.task.loadTasks](http://gruntjs.com/api/grunt.task#grunt.task.loadtasks)` (or it's alias `[grunt.loadTasks](http://gruntjs.com/api/grunt#grunt.loadtasks)`).

Use it like this:


    module.exports = function (grunt) {
      require('jit-grunt')(grunt)
    
      grunt.initConfig({})
    
      // Load all your external configs.
      // It's important to use it _after_ Grunt config has been initialized,
      // otherwise it will have nothing to work with.
      grunt.loadTasks('tasks')
    
      grunt.registerTask('default', ['cssmin'])
    }

Grunt will automatically load all `js` or `coffee` configs from specified directory. Nice and clean!

But, if you'll try to use it, you'll notice it does nothing. How is that?

We still need to do one more thing.

**Making configuration loading work**

Let's look into our `gruntfile.js` code once again:


    module.exports = function (grunt) {
      require('jit-grunt')(grunt)
    
      grunt.initConfig({})
    
      grunt.loadTasks('tasks')
    
      grunt.registerTask('default', ['cssmin'])
    }

Notice, that `grunt.loadTasks` loads files from `tasks` directory, but never assigns it to our actual Grunt config.


Compare it with a way `load-grunt-configs` works:


    module.exports = function (grunt) {
      require('jit-grunt')(grunt)
    
      // 1. Load configs
      const configs = require('load-grunt-configs')(grunt, {
        config: { src: 'tasks/*.js' }
      })
    
      // 2. Assign configs
      grunt.initConfig(configs)
    
      grunt.registerTask('default', ['cssmin'])
    }

More of it, we initalize our Grunt config *before* actually loadings tasks configuration.

If you are getting a strong feeling that it will make us end up with empty Grunt config — you're totally right.

You see, on contrary to the `load-grunt-configs`, `grunt.loadTasks` just imports files into `gruntfile.js`. *It does nothing more*.

Woah! So, how do we make use of it?

Let's explore!

Create a file inside directory `tasks` named `test.js`:


    module.exports = function () {
      console.log("Hi! I'm an external task and I'm taking precious space in your console!")
    }

Let's run Grunt now:


    $ grunt

We'll see printed to the console:


    > Hi! I'm an external task and I'm taking precious space in your console!

So, upon importing `grunt.loadTasks` executes provided by files functions. That's nice, but what's use of it for us? We still can't do a thing with what we actually want — to configure our tasks.

Hold my beer!

What if I tell you that *there is a way to command our Grunt from within external configuration files*?
`grunt.loadTasks` upon importing provides current Grunt instance as a function first argument and also binds it to `this`.

Update our Gruntfile:


    module.exports = function (grunt) {
      require('jit-grunt')(grunt)
    
      grunt.initConfig({
        // Add some value to work with
        testingValue: 123
      })
    
      grunt.loadTasks('tasks')
    
      grunt.registerTask('default', ['cssmin'])
    }

And change external config file `tasks/test.js`:


    // Add `grunt` as first function argument
    module.exports = function (grunt) {
      // Now, use Grunt methods on `grunt` instance
      grunt.log.error('I am Grunt error!')
    
      // Or use them on `this` which does the same
      this.log.error('I am Grunt error too, from the same instance, but from `this`!')
    
      const config = grunt.config.get()
    
      grunt.log.ok('And here goes current config:')
      grunt.log.ok(config)
    }

Now, lets run Grunt again:


    $ grunt

And what we'll get:


    > I am Grunt error!
    > I am Grunt error too, from the same instance, but from `this`!
    > And here goes current config:
    > {
        testingValue: 123
      }

See how we accessed native Grunt methods from an external file? And even were able to retrieve current Grunt config?

Are you thinking about that too? Yeah, the full power of Grunt is already there, right at our fingertips in each file!

If you wondering why methods inside external files can affect our main Grunt instance, it is because of a *referencing*. `grunt.loadTasks` passing as `this` and `grunt` our current Grunt instance, not a copy of it. By invoking methods on that reference we're able to read and mutate our main Grunt configuration file.

Now, we need to actually configure something!

One last thing...

**This time making configuration loading work for real**

Alright, we come a long way. Our tasks are loaded automatically and faster. We learned how to load external configs with native Grunt method. But our configuration is still not quite there. They do not end up in Grunt config.

But we almost there. We learned, that in imported by `grunt.loadTasks` files we can use any Grunt instance methods. They are available on `grunt` and `this` instances.

Among many, there is a precious `[grunt.config](http://gruntjs.com/api/grunt.config#grunt.config)` method. It allows to set a value in an existing Grunt config. The main one, wich we initialized in our Gruntfile, remember?

But what's important, that way we also can define tasks configurations, just by setting them into our Grunt config. Exactly what we needed!

Let's use it!


    // tasks/test.js
    
    module.exports = function (grunt) {
      grunt.config('csso', {
        build: {
          files: { 'style.css': 'styles.css' }
        }
      })
    
      // same as
      // this.config('csso', {
      //   build: {
      //     files: { 'style.css': 'styles.css' }
      //   }
      // })
    }

Now let's update Gruntfile to log the current config. We need to see what we did, after all.


    module.exports = function (grunt) {
      require('jit-grunt')(grunt)
    
      grunt.initConfig({
        testingValue: 123
      })
    
      grunt.loadTasks('tasks')
    
      // Log our current config
      console.log(grunt.config())
    
      grunt.registerTask('default', ['cssmin'])
    }

Run Grunt:


    $ grunt

And here what console says:


    > {
        testingValue: 123,
        csso: {
          build: {
            files: {
              'style.css': 'styles.css'
            }
          }
        }
      }

`grunt.config` sets `csso` value when imported, so it's configurated and ready to run when Grunt is invoked. Perfect.

Note that if you used `load-grunt-configs` previsuly, you had code like that, where each file exports an configuration object:


    // tasks/grunt-csso.js
    
    module.exports = {
      target: {
        files: { 'style.css': 'styles.css' }
      }
    }

That needs to be changed to a function, as described above:

    // tasks/grunt-csso.js
    
    module.exports = function (grunt) {
      grunt.config('csso', {
        build: {
          files: { 'style.css': 'styles.css' }
        }
      })
    }

Now, one last thing... this time for real!


## Taking external config files to the next level

We learned a lot. Or maybe not. Load tasks, load external configuration files, define a configuration with Grunt methods... that's fine, but where's the profit?

Stay tuned and still hold my beer!

By that time you probably externalized all your tasks configuration. In a large project you probably have something like that:


    tasks
      ├─ grunt-browser-sync.js  
      ├─ grunt-cache-bust.js
      ├─ grunt-contrib-clean.js 
      ├─ grunt-contrib-copy.js  
      ├─ grunt-contrib-htmlmin.js   
      ├─ grunt-contrib-uglify.js
      ├─ grunt-contrib-watch.js 
      ├─ grunt-csso.js  
      ├─ grunt-nunjucks-2-html.js   
      ├─ grunt-postcss.js   
      ├─ grunt-processhtml.js
      ├─ grunt-responsive-image.js  
      ├─ grunt-sass.js  
      ├─ grunt-shell.js 
      ├─ grunt-sitemap-xml.js   
      ├─ grunt-size-report.js   
      ├─ grunt-spritesmith-map.mustache 
      ├─ grunt-spritesmith.js   
      ├─ grunt-standard.js  
      ├─ grunt-stylelint.js 
      ├─ grunt-tinypng.js   
      ├─ grunt-uncss.js 
      └─ grunt-webfont.js
    gruntfile.js

That keeps our Gruntfile relatively small and things seem to be well organized. But do you get a clear picture of the project just by glancing into this cold and lifeless list of tasks? What actually do they do? What's the flow?

Can you tell that Sass files going through `grunt-sass`, then `grunt-postcss:autoprefixer`, then `grunt-uncss`, and finally through `grunt-csso`? Is it obvious that the `clean` task is cleaning it, `grunt-spritesmith` is generating a Sass file which should be picked up too, and `grunt-watch` watches over all changes?

Seems like things all over the place. Like we have gone too far with splitting!

So, finally... what if tell you that *a better way would be to group configs based on features*?
Instead of a not-so-helpful list of tasks, we'll get a sensible list of features. How about that?


    tasks
      ├─ data.js 
      ├─ fonts.js 
      ├─ icons.js 
      ├─ images.js 
      ├─ misc.js 
      ├─ scripts.js 
      ├─ sprites.js 
      ├─ styles.js 
      └─ templates.js
    gruntfile.js

That tells me a story! But how could we do that?

We already learned about `grunt.config`. And believe it or not, you can use it multiple time in a single external file!

Lets see how it works:


    // tasks/styles.js
    
    module.exports = function (grunt) {
      grunt.config('sass', {
        build: {/* not important right now options */}
      })
    
      grunt.config('postcss', {
        autoprefix: {/* not important right now options */}
      })
    }

One file, multiple configuration. Quite flexible!

But there is an issue we missed.

How to deal with tasks such as `grunt-contrib-watch`? Its configuration is a whole monolithic thing with definitions for each task:


    // tasks/grunt-contrib-watch.js
    
    module.exports = function (grunt) {
      grunt.config('watch', {
        sprites: {/* not important right now options */},
        styles: {/* not important right now options */},
        templates: {/* not important right now options */}
      })
    }

We can't simply use `grunt.config` to set `watch` configuration in each file, as it will override same `watch` configuration in already imported files. And leaving it in a standalone file sounds like a bad option too — after all, we wanted to keep all related things close.

Fret not! `[grunt.config.merge](https://gruntjs.com/api/grunt.config#grunt.config.merge)` to the rescue!

While `grunt.config` explicitly sets and *overrides* any existing values in Grunt config, `grunt.config.merge` recursively merges value with already existing Grunt config. And so we get single Grunt config. Simple, but effective way of keep related things together.

An example:


    // tasks/styles.js
    
    module.exports = function (grunt) {
      grunt.config.merge({
        watch: {
          templates: {/* not important right now options */}
        }
      })
    }

    // tasks/templates.js
    
    module.exports = function (grunt) {
      grunt.config.merge({
        watch: {
          styles: {/* not important right now options */}
        }
      })
    }

Will produce single Grunt config:


    {
      watch: {
        styles: {/* not important right now options */},
        templates: {/* not important right now options */}
      }
    }

Just what we needed!

Let's apply this to the real issue — our styles-related configuration files. Replace three files we started with:


    tasks
      ├─ grunt-sass.js
      ├─ grunt-postcss.js   
      └─ grunt-contrib-watch.js


    // tasks/grunt-sass.js
    
    module.exports = function () {
      this.config('sass', {
        build: {/* not important right now options */}
      })
    }


    // tasks/grunt-postcss.js
    
    module.exports = function () {
      this.config('postcss', {
        autoprefix: {/* not important right now options */}
      })
    }


    // tasks/grunt-contrib-watch.js
    
    module.exports = function () {
      this.config('watch', {
        styles: {
          files: ['source/styles/{,**/}*.scss']
          tasks: ['sass', 'postcss:autoprefix']
        },
        templates: {
          files: ['source/templates/{,**/}*']
          tasks: ['nunjucks:build']
        }
        /* rest of watchers... */
      })
    }

With a single `tasks/styles.js`:


    module.exports = function (grunt) {
      grunt.config('sass', {
        build: {
          files: [
            {
              expand: true,
              cwd: 'source/styles',
              src: '{,**/}*.scss',
              dest: 'build/assets/styles',
              ext: '.compiled.css'
            }
          ]
        }
      })
    
      grunt.config('postcss', {
        autoprefix: {
          files: [
            {
              expand: true,
              cwd: 'build/assets/styles',
              src: '{,**/}*.compiled.css',
              dest: 'build/assets/styles',
              ext: '.prefixed.css'
            }
          ]
        }
      })
    
      // Note that we need to use `grunt.config.merge` here!
      grunt.config.merge({
        watch: {
          styles: {
            files: ['source/styles/{,**/}*.scss'],
            tasks: ['sass', 'postcss:autoprefix']
          }
        }
      })
    }

Now it's much easier to tell just by glancing into `tasks/styles.js` that styles have 3 related tasks.
I'm sure you can imagine extending this concept to other grouped tasks, like all the things you might want to do with scripts, images, or anything else. That gives us a reasonable configuration organization. Finding things will be much easier, trust me!

And that's it! The whole point of what we learned. Now you can give me my bear back and continue your regular deeds.


## Conclusion

Grunt is no longer that fresh and bright. But till the date it is straightforward and reliable tool that does it job well and with proper treatment gives even less reasons to swap it for something newer. Let's recap:


1. For tasks loading use `[jit-grunt](https://github.com/shootaroo/jit-grunt)` instead of `[load-grunt-tasks](https://github.com/sindresorhus/load-grunt-tasks)`. It's same, but insanely faster.
2. Move specific tasks configs from Gruntfile into external config files to keep things organized.
3. Use native `[grunt.task.loadTasks](http://gruntjs.com/api/grunt.task#grunt.task.loadtasks)` to load external config files. It's simple but powerful as it exposes all Grunt capabilities in all config files.
4. Finally, think about better way to organize you config files! Group them by feature or domain instead of the task itself. Use `[grunt.config.merge](https://gruntjs.com/api/grunt.config#grunt.config.merge)` to split complex tasks like `watch`.

And for sure, finally check [Grunt documentation](https://gruntjs.com/getting-started). After all those years it still worth a read.

If you like to see a real-world example, check out [Kotsu](https://github.com/LotusTM/Kotsu), a Grunt-based Starter Web Kit and Static Website Generator. There you'll find even more tricks.

Got better ideas about how to organize Grunt configs even better? Do not hesitate to share with us in the comments.

P.S. I don't drink. So I have no idea where that bear came from. Why did you hold it all the time, anyway?