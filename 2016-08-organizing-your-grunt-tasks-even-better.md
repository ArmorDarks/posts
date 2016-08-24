---
author: Serj Lavrin (https://github.com/ArmorDarks)
created: 2016-08-10
status: WIP
keywords: [Grunt, Node, JavaScript, tasks, configs, organizing, refactoring]
---


Organizing Your Grunt Tasks Even Better
=======================================

Hello <%= username %>!

This is follow up of [Organizing Your Grunt Tasks](https://css-tricks.com/organizing-grunt-tasks/) article.

After reading it, I felt there are few more useful things about tasks and configs organization to be told.

Note that this article is more for beginners or not very familiar with Grunt users. Like designers. Yes, I am talking about you, Tom. There won't be any mind blowing discoveries here for admirals of Grunt. Still, one might wish to take a glance at article, just to ensure that you did not miss anything interesting blinded by your infinite awesomeness. No one will judge you. Probably.

Let us begin.


## Loading tasks

[Organizing Your Grunt Tasks](https://css-tricks.com/organizing-grunt-tasks/) already made good coverage of how we can make our life easier with `load-grunt-tasks`, so we would never need to write this again:

```
grunt.loadNpmTasks('grunt-postcss');
grunt.loadNpmTasks('grunt-contrib-concat');
grunt.loadNpmTasks('grunt-contrib-cssmin');
grunt.loadNpmTasks('grunt-contrib-jshint');
grunt.loadNpmTasks('grunt-jsvalidate');
grunt.loadNpmTasks('grunt-contrib-uglify');
grunt.loadNpmTasks('grunt-contrib-watch');
grunt.loadNpmTasks('grunt-contrib-sass');
```

However, as usually, there is always something to be improved. In this case, it would be reasonable to replace `load-grunt-tasks` with [jit-grunt](https://github.com/shootaroo/jit-grunt).

`jit-grunt` is very similar to `load-grunt-tasks` tool. It just loads all tasks automatically too. However, `jit-grunt` doing this much faster, than `load-grunt-tasks` or even native `grunt.loadNpmTasks`. In large configs difference can be striking:

Without `jit-grunt`:

```
loading tasks     5.7s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 84%
assemble:compile  1.1s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 16%
Total 6.8s
```

With `jit-grunt`:

```
loading tasks     111ms  ▇▇▇▇▇▇▇▇▇ 8%
loading assemble  221ms  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 16%
assemble:compile   1.1s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 77%
Total 1.4s
```

If you wonder how that's possible, read about original issue, which lead to creation of `jit-grunt` [here](https://github.com/gruntjs/grunt/issues/975).

`jit-grunt` is drop-in replacement of `load-grunt-tasks`, to use it enough to install it:

``` shell
npm install jit-grunt --save
```

And require before `grunt.initConfig`:

``` js
module.exports = function (grunt) {
  require('jit-grunt')(grunt)

  grunt.initConfig({
    ...
  })
}
```

Note that we should pass `grunt` or `this` as parameter of `jit-grunt`.

For newcomers it might unobvious what exactly happening here. `jit-grunt` or `load-grunt-tasks` just load tasks and let Grunt know that they can be used in configs. It has nothing to do with config declaration itself.

In other words, you are free to choose your own ways of configs declaration and loading. Use `load-grunt-config` with `jit-grunt`, or use `load-grunt-tasks` with technics, described below. They aren't dependent.


# Taking configs beyo... no, for now let's load configs first

Once again, [Organizing Your Grunt Tasks](https://css-tricks.com/organizing-grunt-tasks/) makes good point about splitting monolithic Grunt configs into smaller standalone files, which later can be loaded by `load-grunt-configs` from specified directory.

The truth is, latest versions of Grunt can do same thing natively, and even better — with [grunt.task.loadTasks](http://gruntjs.com/api/grunt.task#grunt.task.loadtasks) (or it's alias [grunt.loadTasks](http://gruntjs.com/api/grunt#grunt.loadtasks)).

It's very easy to use. We need to specify in our `gruntfile` after `grunt.initConfig` `grunt.loadTasks` with destination to configs directory as parameter:

``` js
grunt.initConfig({
  ...
})

grunt.loadTasks('path/to/your/tasks')
```

And Grunt will automatically load all `js` or `coffee` configs from specified directory.

One of side effects is that we no longer need to load configs into variable and pass them to `grunt.initconfig`, as in case with `load-grunt-configs`. Loaded by this method configs will extend already existing config with native Grunt methods.

Instead of this:

``` js
module.exports = function(grunt) {
  var tasks = {scope: ['devDependencies', 'dependencies' ]};
  var options = {config: { src: "grunt/*.js" }};
  var configs = require('load-grunt-configs')(grunt, options);
  require('load-grunt-tasks')(grunt, tasks);
  grunt.initConfig(configs);
  grunt.registerTask('default', ['cssmin'])
}
```

We can have this:

``` js
module.exports = function (grunt) {
  require('jit-grunt')(grunt)

  // Initialize config
  grunt.initConfig({
    ...
  })

  // Load all your external configs
  grunt.loadTasks('path/to/tasks')

  // Register your set of tasks
  grunt.registerTask('default', ['cssmin'])
}
```

Fewer dependencies, less requires and cleaner code.


## Taking configs bey... well, no, let's clarify how it works first

`grunt.loadTasks` similarly to `load-grunt-configs` imports everything inside current Grunt instance from specified path.

However, some of you might ask — "But wait, I see what you did there! You are loading configs, but how do they end up in Grunt config? There is no assignments anywhere!".

You are right. This is where those approaches extremely different.

`load-grunt-configs` is more like a loader and merger of configs in JSON format. It always expects imported files to return JSON, no matter what. We have to manually get value of `load-grunt-configs` and then manually set this result to Grunt config, by declaring it on `grunt.initConfig(config)` or by extending already existing config via `grunt.config.merge(config)`.

Not to say how limiting it might be. What if we need to register additional task in imported config? Or create a new one? Or make a wrapper for existing task? Or make some complex manipulations upon current instance of Grunt?

This is where `grunt.loadTasks` comes in full glory.

You see, on contrary to `load-grunt-configs`,  `grunt.loadTasks` expects exported function, which will be executed upon loading.

For instance, if we will use `grunt.loadTasks('path/to/tasks')` and create new file `test.coffee` inside `path/to/tasks` directory:

``` coffee
module.exports = () ->
  console.log(`Hi! I'm external task and I'm taking precious space in your console!`)
```

And type `grunt` in console, we'll see

``` shell
> grunt
> Hi! I'm external task and I'm taking precious space in your console!
```

Well, that's great, but how the hell we're going to extend our config then, if it's just an executioner of functions from specified files?

One word — "magic". Sorry, wrong word. I meant "magic of passed to function Grunt instance". Actually, that isn't one word, but who cares.

`grunt.loadTasks` upon importing binds as `this` of imported function current instance of Grunt. Just to be on safe side, it also provides current Grunt instance as first argument of function, which being imported.

Let's see how it works.

In our `gruntfile`:

``` js
module.exports = function (grunt) {
  require('jit-grunt')(grunt)

  grunt.initConfig({
    testingValue: 123
  })

  grunt.loadTasks('path/to/tasks')
  grunt.registerTask('default', ['cssmin'])
}
```

With external config file `path/to/tasks/test.coffee`:

``` coffee
module.exports = (grunt) ->
  this.log.error(`I'm Grunt error!`)
  grunt.log.error(`I'm Grunt error too, from same instance, but from alias!`)
  grunt.log.ok(`And here goes current config:`)
  grunt.config.get()
```

Will yeild in console:

``` shell
> grunt
>> I'm Grunt error!
>> I'm Grunt error too, from same instance, but from alias!
>> And here goes current config:
> {
    testingValue: 123
  }
```

You see how we could access native Grunt methods from external file, and even were able to retrieve current config?

This means that full powers of Grunt are already there, right under our fingerprints. We just need to harass it to do whatever our will dictates. To bend the rules. To make perish the weak. Ops, sorry, last one from wrong article...

If you wonder, why methods inside external files affecting our main Grunt instance, it is because of powerful JavaScript feature — referencing. `grunt.loadTasks` passing as `this` or `grunt` not a copy of our main instance, but reference to it. Any changes, made upon `this` or `grunt` will also affect main instance.

So, be accurate with it, do not fall of with attempts to destroy whole little world of Grunt just by setting `this` methods to `null`.


## Taking external configs... no-no, let's cover few pitfalls first

Since `grunt.loadTasks` exposes to us whole Grunt instance in external configs and we can do with it literally whatever we want — inside those external configs we can do anything, that Grunt can do, and it will affect main instance of Grunt.

However, before we will dive into what exactly we can do, we should clarify one important thing — how not to screw whole thing up.

First of all, obvious thing — it's important to load tasks _after_ grunt config has been initialized, otherwise our operations will have nothing to change.

In other words, _wrong_:

``` js
module.exports = function (grunt) {
  grunt.loadTasks('path/to/tasks')

  require('jit-grunt')(grunt)
  grunt.initConfig({
    ...
  })
  grunt.registerTask('default', ['cssmin'])
}
```

_Right_:

``` js
module.exports = function (grunt) {
  require('jit-grunt')(grunt)
  grunt.initConfig({
    ...
  })

  grunt.loadTasks('path/to/tasks')

  grunt.registerTask('default', ['cssmin'])
}
```

Certainly, it depends on your needs and goals, and if you do not intend to change Grunt config, you can load tasks before initializing it, they will execute as expected.

Second thing is order of execution of loaded tasks. It is sequential. This means, that we cannot rely inside any external config file on value, declared inside another external config file.

Let's see example:

We have two files with configs for two tasks:

```
tasks
  ├─ grunt-browser-sync.coffee
  |    module.exports = ->
  |      @config 'browserSync',
  |        debug:
  |          ...
  |
  └─ grunt-contrib-copy.coffee
       module.exports = ->
         @config 'copy',
           misc:
             files:
               ...
```

Now, imagine we want to get something from `copy` task and use it inside `browserSync` task.

We could do this via Grunt config getter:

``` coffee
module.exports = ->
  copyFiles = @config.get('copy').misc.files
  console.log(copyFiles)

  @config 'browserSync',
    ...

```

Well, it won't work. Logging `copyFiles` will yield `undefined`. This happens, because `grunt-browser-sync` has been loaded first, and `grunt-contrib-copy` has not been even processed. Therefore, its value simply have not been written to Grunt instance yet.

Let's try same thing in `grunt-contrib-copy`:

``` coffee
module.exports = ->
  browserSyncFiles = @config.get('browserSync').debug.files
  console.log(browserSyncFiles)

  @config 'copy',
    ...

```

Suddenly it _will_ return result, because `grunt-browser-sync` has been called before `grunt-contrib-copy`, just because of... well, alphabetical order of execution.

So, while it's possible to exchange values between external configs in that way, it is highly unreliable and inadvisable.

Instead, we need to takes those shared values on upper level: for instance, define them in `gruntfile` under `grunt.initConfig()`, and retrieve in external configs via `grunt.config()`. Or implement different solution, based on your needs and environment.

Nothing special. But we had to know caveats to avoid potential pitfalls. You know. Never hurts. Let's move own. To less boring parts.


## Finally lets take external configs... wait for it... wait for it...  beyond!

At this point it should be quite clear, that we can do in our external configs files anything we want. Bound just by our own imagination. And our time. And JavaScript knowledge. And JavaScript limitations. And common sense. And performance issues. And bad practices. Well, whom I'm trying to deceive. You can't do anything with it, but you can do a lot.

Let's take few examples.


### Extending config

Let's start from most trivial task — doing same thing which `load-grunt-configs` had been doing for us before — chopping of massive config into smaller chunks in form of external files.

If in case of `load-grunt-configs` our external configs were like those:

``` js
module.exports = {
  target: {
    files: {
      'style.css': 'styles.css'
    }
  }
};
```

In case of `grunt.loadTasks` it should be like that:

``` js
module.exports = function () {
  this.config('target', {
    files: {
      'style.css': 'styles.css'
    }
  })
}
```

As you can see, we're using power of native to Grunt [config](http://gruntjs.com/api/grunt.config#grunt.config) method, which sets specified object under specified value into already existing Grunt config.

Remember, as we have learned above, `config` here available as `this` method, because `grunt.loadTasks` automatically binds current Grunt instance as `this` for us. Since it also provides same instance as first argument of function, following code will do literally same thing:

``` js
module.exports = function (grunt) {
  grunt.config('cssmin', {
    files: {
      'style.css': 'styles.css'
    }
  })
}
```

Just choose whatever flavor you prefer.

Now, we can go to main `gruntfile` and log current config:

``` js
module.exports = function (grunt) {
  require('jit-grunt')(grunt)

  grunt.initConfig({
    testingValue: 123
  })

  grunt.loadTasks('path/to/tasks')
  
  // Log our current config
  console.log(grunt.config())

  grunt.registerTask('default', ['cssmin'])
}
```

And our console output would be:

``` shell
> grunt
> {
    testingValue: 123,
    target: {
      files: {
        'style.css': 'styles.css'
      }
    }
  }
```

Perfect. Config is packed up and consumed by Grunt during tasks run.


### Merging

At some point you might find out that splitting `gruntfile` on per task basis is not enough.

When there are many config files, one per each task, it might be very hard to catch whole picture.

Consider this:

```
tasks
  ├─ grunt-browser-sync.coffee  
  ├─ grunt-cache-bust.coffee
  ├─ grunt-contrib-clean.coffee 
  ├─ grunt-contrib-copy.coffee  
  ├─ grunt-contrib-htmlmin.coffee   
  ├─ grunt-contrib-uglify.coffee
  ├─ grunt-contrib-watch.coffee 
  ├─ grunt-csso.coffee  
  ├─ grunt-nunjucks-2-html.coffee   
  ├─ grunt-postcss.coffee   
  ├─ grunt-processhtml.coffee
  ├─ grunt-responsive-image.coffee  
  ├─ grunt-sass.coffee  
  ├─ grunt-shell.coffee 
  ├─ grunt-sitemap-xml.coffee   
  ├─ grunt-size-report.coffee   
  ├─ grunt-spritesmith-map.mustache 
  ├─ grunt-spritesmith.coffee   
  ├─ grunt-standard.coffee  
  ├─ grunt-stylelint.coffee 
  ├─ grunt-tinypng.coffee   
  ├─ grunt-uncss.coffee 
  └─ grunt-webfont.coffee
```

It's uneasy to understand what flow Sass files have here.

Can you tell that Sass file goes through `grunt-sass`, then `grunt-postcss:autoprefixer`, then `grunt-uncss`, and finally through `grunt-csso`? And is it obvious that `clean` task cleaning it, `grunt-spritesmith` generating Sass file which should be picked up too, and `grunt-watch` watches over all changes?

From now on, one might say that whole idea to split Grunt config into those files seems not as brilliant as at first. Fear not. This simply indicates that we've gone too far and instead should find ~~peace~~ I mean golden middle. Not peace.

Maybe we should consider grouping of Grunt configs based on features?

Instead of gray list of tasks we will have more sensible list of features: styles, templates, images.

However, how could we do this? How to deal with `grunt-contrib-watch`, which is very recurring task and it covers large list of different files?

Fortunately, we have `grunt.config.merge`, which allows to extend already defined configs in previous files.

Let's see it on mentioned Sass files example.

We can replace those three files:

```
tasks
  |
  ├─ grunt-sass.coffee
  |    module.exports = ->
  |      @config 'sass',
  |        build:
  |          options:
  |            ...
  |
  ├─ grunt-postcss.coffee   
  |    module.exports = ->
  |      @config 'postcss',
  |        autoprefix:
  |          options:
  |            ...
  |
  └─ grunt-contrib-watch.coffee 
       module.exports = ->
         @config 'watch',
           static:
             files: ['<%= path.source.static %>/{,**/}*']
             tasks: ['newer:copy:static']
    
           ...
    
           styles:
             files: [
               '<%= path.source.styles %>/{,**/}*.scss'
               '<%= path.temp.styles %>/{,**/}*.scss'
             ]
             tasks: [
               'sass'
               'postcss:autoprefix'
             ]
```

With single `styles.coffee`:

``` coffee
module.exports = ->
  @config 'sass',
    build:
      options:
        ...
      files: [
        expand: true
        cwd: 'source/styles'
        src: '{,**/}*.scss'
        dest: 'build/assets/styles'
        ext: '.compiled.css'
      ]
      
  @config 'postcss',
    autoprefix:
      options:
        ...
      files: [
        expand: true
        cwd: 'build/assets/styles'
        src: '{,**/}*.compiled.css'
        dest: 'build/assets/styles'
        ext: '.prefixed.css'
      ]

  @config.merge
    watch:
      styles:
        files: [
          'source/styles/{,**/}*.scss'
        ]
        tasks: [
          'sass'
          'postcss:autoprefix'
        ]
```

Now it is much easier to tell just by glancing into `tasks/stykes.coffee` that our styles have 3 related tasks. We didn't transit here all other styles related tasks, but you can go on, consider it a homework. Or not. Depends on your mood.

I hope you didn't miss `this.config.merge` usage here.

While `this.config` explicitly sets config value and completely _overrides_ it, `this.config.merge` will recursively merge tasks with ones, declared in previous files.

This allows us to declare `watch` task in styles config file, and then also declare it in any other files we want too. In the end, they all will merge into single big final config.

Quite simple, but effective way of keep related things together.

Note, that merging is not exclusive for `grunt.loadTasks`. `load-grunt-configs` allows to do merging too, you can read about it [here](https://github.com/creynders/load-grunt-configs#split-multi-task-configurations). We've just reviewed native for Grunt method.


### Doing... things

Because our config files are simply exported functions with help of Node's `module.exports`, we can do some operations, which might be not that obvious.

For example, we can require any third party libs right inside our external config file and perform necessary operations.

Let's say, we want to pass some Grunt data inside Sass via `node-sass` options. Quite easy:

``` coffee
sass = require('node-sass')
{ castToSass } = require('node-sass-utils')(sass)
{ get } = require('lodash')

module.exports = ->
  @config 'sass',
    build:
      options:
        sourceMap: true
        functions:
          'grunt-data($query)': (query) =>
            query = query.getValue()
            data = @config('myData')
            return castToSass(get(data, query))
      files: [
        expand: true
        cwd: '<%= path.source.styles %>'
        src: '{,**/}*.scss'
        dest: '<%= path.build.styles %>'
        ext: '.compiled.css'
      ]
```

In fact, since config of task is just an Object, we can do with it whatever crazy comes into our head. For example, how about generating targets in loop, based on array?

``` coffee
{ join } = require('path')

module.exports = ->
  let targets = [{
      name: 'myTarget1'
      dest: 'my/dir'
    }, {
      name: 'myTarget2'
      dest: 'another/dir'
    }]

  let Task = () -> {

    targets.forEach((target) => {
      @[target.name] = {
        files: [
          expand: true
          cwd: 'source/styles'
          src: '{,**/}*.scss'
          dest: join('generic/path/', target.dest)
          ext: '.compiled.css'
        ]
      }  
    })

  }

  @config 'sass', new Task()
```

Now we can access targets as `sass:myTarget1`,  `sass:myTarget2`. We can use it, let's say, for i18n purposes, by dynamically generating targets based on locales, with appropriate configurations for each locale. Don't go too crazy with it, though. Do not overengineer your config.

Or another example — we can create completely new tasks right inside imported files:

``` coffee
module.exports = ->
  @registerMultiTask 'test', 'Log argument to console', (args...) ->
    console.log 'Your nonsensical args: ', args
```

And just call for `test` task in console:

``` shell
grunt test:myArg:myArg2:myArg3
```

We will see logged:

``` shell
> Your nonsensical args: myArg,myArg2,myArg3
```

Quite nice. Now, think up your own reason to use it. Or don't. Once again, do not overengineer your configs.


## The end

Before throwing yourself into endless refactoring and tinkering of your Grunt config based on this article, please, read this.

Yes, those technics are quite powerful and makes Grunt configs much easier to maintain, read and extend. However, you should carefully consider, does refactoring worth your time and efforts.

Consider this: swapping `load-grunt-tasks` with `jit-grunt` won't make interstellar difference, using `grunt.loadTasks` with full power of `this` instead of `load-grunt-configs` won't change anything cardinally too if your configs working and you and your teammates are ok with it. Not to mention that `load-grunt-configs` can make a lot of what `grunt.loadTasks` can do too.

However, if you are starting new project, I would suggest writing it in a proper way right from the beginning. Thankfully, it almost taking _less_ time to write, then traditional Grunt config.

Besides, external configs are very portable. It is very easy to plug  related to task file out of one project and paste it into another.

If you want to get live example, check out [Kotsu](https://github.com/LotusTM/Kotsu/tree/release/1.0.0), an Starter Web Kit and Static Website Generator, which implies described in this article technics.

So, this is it. I hope you have learned something new or reminded yourself about some cool forgotten features of Grunt. Don't make this article useless by saying you didn't :)

Got better ideas about how to organize Grunt configs even better? Do not hesitate to share with us in comments. We will definitely patent those approaches and then sell them to you for moneys. Just kidding :)

And don't be too hard on my article and don't judge me by my stupid jokes. Or judge. I don't care (please, don't, I will cry).

Thanks.

Live and propser Y.