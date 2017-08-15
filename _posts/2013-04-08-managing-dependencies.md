---
layout: post
title: "Managing Dependencies"
permalink: managing-dependencies
category: user-guide
github: "https://github.com/ember-cli/ember-cli.github.io/blob/master/_posts/2013-04-08-managing-dependencies.md"
---

### NPM and Bower Configuration

Ember CLI supports [NPM](https://www.npmjs.com) and [Bower](http://bower.io/)
for dependency management.

A newly generated Ember CLI project only has NPM dependencies, so you will
notice a `package.json` file at the root of your project, but not a `bower.json`.
To use Bower packages, you will have to first run `bower init` to create a
`bower.json` file also at the root of the project.

NPM's `package.json` together with Bower's `bower.json` allow you to declare
the dependencies of your project.
Changes to your dependencies should be managed through these files, rather
than manually installing packages individually.

Executing `npm install` will install all of the dependencies listed in
`package.json` in one step. Similarly, executing `bower install` will install
all of the dependencies listed in `bower.json` in one step.

Ember CLI is configured to have git ignore your `bower_components` and
`node_modules` directories by default. Using the Bower and NPM configuration
files allows collaborators to fork your repo and get their dependencies installed
locally by executing `npm install` and `bower install` themselves.

Ember CLI watches `bower.json` for changes. Thus it reloads your app if you
install new dependencies via `bower install <dependencies> --save`. If you
install NPM dependencies via `npm install <dependencies> --save`, you will need
to restart your Ember CLI server session manually.

Further documentation about NPM and Bower is available at their official
documentation pages:

* [Bower](http://bower.io/)
* [NPM](https://www.npmjs.com)

Note that it is often easiest to install Ember addon dependencies using the
`ember install` command, which will save all dependencies to the correct
configuration files and run any further setup steps required.

### Compiling Assets

Ember CLI uses the [Broccoli](https://github.com/broccolijs/broccoli) assets
pipeline.

The assets manifest is located in the `ember-cli-build.js` file in your project
root (not the default `ember-cli-build.js`).

To add an asset specify the dependency in your`ember-cli-build.js` before
calling `app.toTree()`. You can only import assets that are within the
`bower_components` or `vendor`
directories. The following example scenarios illustrate how this works.

#### Javascript Assets

##### Standard Non-AMD Asset

First, provide the asset path as the first and only argument:

{% highlight javascript %}
app.import('bower_components/moment/moment.js');
{% endhighlight %}

From here you would use the package as specified by its documentation, usually
a global variable. In this case it would be:

{% highlight javascript %}
import Ember from 'ember';
/* global moment */
// No import for moment, it's a global called `moment`

// ...
var day = moment('Dec 25, 1995');
{% endhighlight %}

_Note: Don't forget to make ESLint happy by adding a `/* global MY_GLOBAL */`
to your module, or by defining it within the `predefs` section of your
`.eslintrc.js` file._

Alternatively, you could generate an ES6 shim to make the library accessible
via `import`.

First, generate the shim:

{% highlight bash %}
ember generate vendor-shim moment
{% endhighlight %}

Next, provide the vendor asset path:

{% highlight javascript %}
app.import('vendor/shims/moment.js');
{% endhighlight %}

Finally, use the package by adding the appropriate `import` statement:

{% highlight javascript %}
import moment from 'moment';

// ...
var day = moment('Dec 25, 1995');
{% endhighlight %}

##### Standard Named AMD Asset

Provide the asset path as the first argument, and the list of modules and
exports as the second:

{% highlight javascript %}
app.import('bower_components/ic-ajax/dist/named-amd/main.js');
{% endhighlight %}

To use this asset in your app, import it.
For example, with `ic-ajax`, when to use `ic.ajax.raw`:

{% highlight javascript %}
import { raw as icAjaxRaw } from 'ic-ajax';
//...
icAjaxRaw( /* ... */ );
{% endhighlight %}

##### Standard Anonymous AMD Asset

Provide the asset path as the first argument, and the desired module name
in the second:

{% highlight javascript %}
app.import('bower_components/ic-ajax/dist/amd/main.js', {
  using: [
    { transformation: 'amd', as: 'ic-ajax' }
  ]
});
{% endhighlight %}

To use this asset in your app, import it.
For example, with `ic-ajax`, when to use `ic.ajax.raw`:

{% highlight javascript %}
import { raw as icAjaxRaw } from 'ic-ajax';
//...
icAjaxRaw( /* ... */ );
{% endhighlight %}

##### Environment Specific Assets

If you need to use different assets in different environments, specify an
object as the first parameter. That object's key should be the environment
name, and the value should be the asset to use in that environment.

{% highlight javascript %}
app.import({
  development: 'bower_components/ember/ember.js',
  production:  'bower_components/ember/ember.prod.js'
});
{% endhighlight %}

If you need to import an asset in one environment but not import it or any
alternatives in other environments then you can wrap `app.import` in an `if`
statement.

{% highlight javascript %}
if (app.env === 'development') {
  app.import('vendor/ember-renderspeed/ember-renderspeed.js');
}
{% endhighlight %}

##### Customizing a built-in Asset

This is somewhat non-standard and discouraged, but suppose that due to a
requirement in your application that you need to use the full version of
Handlebars even in the production environment.  You would simply provide the
path to the `EmberApp` constructor:

{% highlight javascript %}
var app = new EmberApp({
  vendorFiles: {
    'handlebars.js': {
      production: 'bower_components/handlebars/handlebars.js'
    }
  }
});

{% endhighlight %}

Alternatively, if you want to exclude the built-in asset from being
automatically included in `vendor.js`, you can set its value to `false`:

{% highlight javascript %}
var app = new EmberApp({
  vendorFiles: {
    'handlebars.js': false
  }
});

{% endhighlight %}

_Note: The built-in assets are required dependencies needed by the environment
to run your app. If you use the above method to specifically exclude
some, you should still be including them in some other way._

##### Whitelisting and Blacklisting Assets

You can limit which dependencies in your package.json file get imported into
your Ember application by using the addons option of the EmberApp constructor. A
`whitelist` parameter allows you to restrict modules to a specific list. A
`blacklist` parameter excludes specific modules from being imported into your
app:

{% highlight javascript %}
var app = new EmberApp({
  addons: {
    blacklist: [
      'fastboot-app-server'
    ]
  }
});

{% endhighlight %}

##### Test Assets

You may have additional libraries that should only be included when running
tests (such as qunit-bdd or sinon). These can be imported into your app in your
ember-cli-build.js:

{% highlight javascript %}
// ember-cli-build.js
var EmberApp = require('ember-cli/lib/broccoli/ember-app'),
    isProduction = EmberApp.env() === 'production';

var app = new EmberApp();

if ( !isProduction ) {
    app.import( app.bowerDirectory + '/sinonjs/sinon.js', { type: 'test' } );
    app.import( app.bowerDirectory + '/sinon-qunit/lib/sinon-qunit.js', { type: 'test' } );
}

module.exports = app.toTree();
{% endhighlight %}

**Notes:**
- Be sure to pass `{ type: 'test' }` as the second argument to `app.import`.
  This will ensure that your libraries are compiled into the `test-support.js`
  file.

#### Styles

##### Static CSS

Provide the asset path as the first argument:

{% highlight javascript %}
app.import('bower_components/foundation/css/foundation.css');
{% endhighlight %}

All style assets added this way will be concatenated and output as
`/assets/vendor.css`.

##### Dynamic Styles (SCSS, LESS, etc)

The vendor trees that are provided upon instantiation are available to your
dynamic style files.  Take the following example (in `app/styles/app.scss`):

{% highlight scss %}
@import "bower_components/foundation/scss/normalize.scss";
{% endhighlight %}

#### Other Assets

##### Using app.import()

All other assets like images or fonts can also be added via `import()`. By default, they
will be copied to `dist/` as they are.

{% highlight javascript %}
app.import('bower_components/font-awesome/fonts/fontawesome-webfont.ttf');
{% endhighlight %}

This example would create the font file in `dist/font-awesome/fonts/fontawesome-webfont.ttf`.

You can also optionally tell `import()` to place the file at a different path.
The following example will copy the file to `dist/assets/fontawesome-webfont.ttf`.

{% highlight javascript %}
app.import('bower_components/font-awesome/fonts/fontawesome-webfont.ttf', {
  destDir: 'assets'
});
{% endhighlight %}

If you need to load certain dependencies before others, you can set the
`prepend` property equal to `true` on the second argument of `import()`. This
will prepend the dependency to the vendor file instead of appending it, which
is the default behavior.

{% highlight javascript %}
app.import('bower_components/es5-shim/es5-shim.js', {
  type: 'vendor',
  prepend: true
});
{% endhighlight %}

If you need some of your assets to be included into specific file you can
provide an `outputFile` option for your import:

{% highlight javascript %}
// ember-cli-build.js
app.import('vendor/dependency-1.js', { outputFile: 'assets/additional-script.js'});
app.import('vendor/dependency-2.js', { outputFile: 'assets/additional-script.js'});
{% endhighlight %}

As a result both dependencies will end up in `dist/assets/additional-script.js`
in the same order they were specified.

_Note: `outputFile` works only for javascript and css files._

##### Using broccoli-funnel

With the [broccoli-funnel](https://github.com/broccolijs/broccoli-funnel)
package, (parts of) a bower-installed package can be used as assets as-is.
First ensure that the Broccoli
package needed to build is installed:

{% highlight bash %}
npm install broccoli-funnel --save-dev
{% endhighlight %}

Add this import to the top of `ember-cli-build.js`, just below the `EmberApp` require:

{% highlight javascript %}
var Funnel = require('broccoli-funnel');
{% endhighlight %}

Within `ember-cli-build.js`, we merge assets from a bower dependency with the main app tree:

{% highlight javascript %}
module.exports = function(defaults) {

   ...

   // Copy only the relevant files. For example the WOFF-files and stylesheets for a webfont:

   var extraAssets = new Funnel('bower_components/a-lovely-webfont', {
      srcDir: '/',
      include: ['**/*.woff', '**/stylesheet.css'],
      destDir: '/assets/fonts'
   });

   // Providing additional trees to the `toTree` method will result in those
   // trees being merged in the final output.

   return app.toTree(extraAssets);

}
{% endhighlight %}

In the above example the assets from the fictive bower dependency called `a-lovely-webfont` can now
be found under `/assets/fonts/`, and might be linked to from `index.html` like so:

{% highlight html %}
<link rel="stylesheet" href="assets/fonts/lovelyfont_bold/stylesheet.css">
{% endhighlight %}

You can exclude assets from the final output in a similar fashion. For example,
to exclude all `.gitkeep` files from the final output:

{% highlight javascript %}
// Again, add this import to the top of `ember-cli-build.js`, just below the `EmberApp` require:
var Funnel = require('broccoli-funnel');

// Normal ember-cli-build contents

// Filter toTree()'s output
var filteredAssets = new Funnel(app.toTree(), {
  // Exclude gitkeeps from output
  exclude: ['**/.gitkeep']
});

// Export filtered tree
module.exports = filteredAssets;
{% endhighlight %}

_Note: [broccoli-static-compiler](https://github.com/joliss/broccoli-static-compiler) is deprecated. Use [broccoli-funnel](https://github.com/broccolijs/broccoli-funnel) instead._
