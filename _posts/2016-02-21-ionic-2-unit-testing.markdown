---
title:  "Unit Testing an Ionic2 project"
date:   2016-02-21 11:34:23
categories: [dev]
tags: [ionic2, angular2, testing]
---

**TL;DR** - I have an Ionic 2 project on github set up with unit testing, [dive in][clicker-repo], or read on.

Ionic 2 looks very promising, allowing us to use the power of Angular 2 to create beautiful native mobile apps. As of yet, there isn't much guidance on how to set up unit testing for your Ionic 2 project, making one of the core benefits of Angular 2 redundant to early adopters.

This post explains the set up we're running and how you can incorporate it into your own project without too much pain.

The Demo App (Clicker)
----------------------

For the purposes of this post, it'll be useful to have the demo app cloned locally.

`git clone git@github.com:lathonez/clicker.git`

The app is very simple, each [clicker][analog-clicker-img] is a counter that lets the user increment some arbitrary event. The motivation (aside from learning things) was to plot the number of times my partner bitched about the weather here in Wellington.

![alt text](/images/ionic2_unit_testing/clicker_screenshot.png "Clicker App"){: .center-image }

Directory Structure
--------------------

There is some debate around where to keep unit tests. I started off with the unit tests in a separate `test/` directory, but have recently combined with the source code as per the [Angular 2 Style Guide][angular2-sg-dir]. Now I use `test/` for configs, etc purely pertaining to the test setup.


```
app/
├── app.html
├── app.spec.ts
├── app.ts
├── components
│   ├── clickerButton
│   │   ├── clickerButton.html
│   │   ├── clickerButton.spec.ts
│   │   └── clickerButton.ts
│   └── clickerForm
│       ├── clickerForm.html
│       ├── clickerForm.spec.ts
│       └── clickerForm.ts
├── models
│   ├── clicker.spec.ts
│   ├── clicker.ts
│   ├── click.spec.ts
│   └── click.ts
├── pages
│   ├── clickerList
│   │   ├── clickerList.html
│   │   ├── clickerList.scss
│   │   ├── clickerList.spec.ts
│   │   └── clickerList.ts
│   └── page2
│       ├── page2.html
│       ├── page2.scss
│       └── page2.ts
└── services
    ├── clickers.spec.ts
    ├── clickers.ts
    ├── utils.spec.ts
    └── utils.ts

test
├── app.stub.ts
├── config.ts
├── gulpfile.ts
├── ionic-angular.js
├── karma.config.js
├── test-main.js
└── testUtils.ts
```

This may not be the correct decision for your project / team. If you decide to keep the source and tests separately, you should still be able to follow this post with minimal config changes.

A simple unit test on app.ts
----------------------------

We probably haven't got much logic in `app.ts` worthy of unit testing. However, `app.ts` is the root source file, if we include it in our test suite, all other spec that we write will be included too.

`cp clicker/app/app.spec.ts myApp/app/app.spec.ts`

Modify the test cases in [app.spec.ts][app.spec.ts] to suit your application, or use the simple example below:

```javascript
import { TEST_BROWSER_PLATFORM_PROVIDERS, TEST_BROWSER_APPLICATION_PROVIDERS} from 'angular2/platform/testing/browser';
import { setBaseTestProviders } from 'angular2/testing';
import { IonicApp, Platform }   from 'ionic-angular';
import { MyApp }           from './app';

// this needs doing _once_ for the entire test suite, hence it's here
setBaseTestProviders(TEST_BROWSER_PLATFORM_PROVIDERS, TEST_BROWSER_APPLICATION_PROVIDERS);

let myApp = null;

export function main() {

  describe('MyApp', () => {

    beforeEach(function() {
      let platform = new Platform();
      myApp = new MyApp(platform);
    });

    it('initialises with two possible pages', () => {
      expect(myApp).not.toBeNull();
    });
  });
}
```

Note the [setBaseTestProviders][sbtp-docs] line, enabling us to utilise Angular 2's testing framework in our tests. More on this later.

Building the tests
-------------------

Our tests are written in Typescript, the same as our Ionic 2 source code. When we run `ionic serve`, webpack compiles the source into Javascript. It isn't going to do the same for our tests, because it doesn't (and shouldn't) know anything about them. The setup we use is heavily based on the excellent [Angular 2 Seed][angular2-seed-repo] project.

Enter [gulp][gulp-home]. Gulp will compile all of our source code and unit tests into individual files within www/build/test.

Make the following changes to your project:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>mkdir -p myApp/test
cp clicker/test/gulpfile.ts myApp/test
cp clicker/test/config.ts myApp/test</code>
</pre>
</div>

* [gulpfile.ts][gulpfile.ts]: gulp’s config file
* [config.ts][config.ts]: config file for this setup, containing only paths at the moment. Can be [expanded][angular2-seed-cfg] down the line.

This gulpfile defines several tasks which gulp will carry out for us during the test cycle:

1. **test.clean**: remove content from `www/build`, except `www/build/js`, which isn't used for testing
2. **test.lint**: perform static analysis on source code using `tslint` if enabled
3. **sass, copy.fonts, copy.html**: sourced directly from Ionic's [gulpfile.js][gulpfile.js]
4. **test.build.typescript**: compile all our Typescript (both source and test), into individual Javascript files. This is done differently from Ionic which bundles everything into `app.bundle.js`
5. **test.karma**: spin up [Karma][karma-home] (more on this later)
6. **test**: combine all of the above to run the tests (more on this later)

**Install Dependencies and Typings:**

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install -g typings
npm install --save-dev chalk del gulp gulp-load-plugins gulp-inline-ng2-template gulp-tap gulp-tslint gulp-typescript karma run-sequence tslint ts-node</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="lowlight">
<code>typings install --ambient --save bluebird chalk del es6-shim express glob gulp gulp-load-plugins gulp-typescript gulp-util jasmine karma log4js mime minimatch node orchestrator q run-sequence serve-static through2 vinyl</code>
</pre>
</div>

You're now ready to build the tests:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>node_modules/gulp/bin/gulp.js --gulpfile test/gulpfile.ts --cwd ./ test.build</code>
</pre>
</div>

```
[11:26:06] Requiring external module ts-node/register
[11:26:08] Using gulpfile ~/code/clicker/test/gulpfile.ts
[11:26:08] Starting 'test.build'...
[11:26:08] Starting 'test.lint'...
[11:26:08] Starting 'test.clean'...
[11:26:08] Deleted -
[11:26:08] Finished 'test.clean' after 10 ms
[11:26:09] Finished 'test.lint' after 1.11 s
[11:26:09] Starting 'sass'...
[11:26:09] Starting 'copy.fonts'...
[11:26:09] Starting 'copy.html'...
[11:26:09] Finished 'copy.html' after 27 ms
[11:26:09] Finished 'copy.fonts' after 31 ms
[11:26:10] Finished 'sass' after 736 ms
[11:26:10] Starting 'test.build.typescript'...
[11:26:12] Finished 'test.build.typescript' after 2.42 s
[11:26:12] Finished 'test.build' after 4.27 s
```

Running the tests
------------------

To get [Karma][karma-home] up and runnning, we need more boilerplate config and more dev dependencies.

Copy the following files into your project:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>cp clicker/test/app.stub.ts clicker/test/ionic-angular.js clicker/test/karma.config.js clicker/test/test-main.js myApp/test</code>
</pre>
</div>

* [app.stub.ts][app.stub.ts]: A stub for Ionic's @App decorator.
* [ionic-angular.js][ionic-angular.js]: A stub for Ionic's [index.ts][ionic-index.ts] so we don't have to proxy their modules in Karma.
* [karma.config.js][karma.config.js]: Karma's config
* [test-main.js][test-main.js]: Main entry point for [unit test excution using RequireJS][karma-tm-docs].

**Install deps:**

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install --save-dev es6-module-loader gulp-watch jasmine-core karma-coverage karma-jasmine karma-mocha-reporter karma-phantomjs-launcher phantomjs-prebuilt systemjs traceur</code>
</pre>
</div>

Now we're ready to test: `node_modules/gulp/bin/gulp.js --gulpfile test/gulpfile.ts --cwd ./ test`

```
> gulp --gulpfile test/gulpfile.ts --cwd ./ test

[11:28:25] Requiring external module ts-node/register
[11:28:28] Using gulpfile ~/code/clicker/test/gulpfile.ts
[11:28:28] Starting 'test'...
[11:28:28] Starting 'test.build'...
[11:28:28] Starting 'test.lint'...
[11:28:28] Starting 'test.clean'...
[11:28:28] Deleted /home/lathonez/code/myApp/www/build/app.html, /home/lathonez/code/myApp/www/build/components, /home/lathonez/code/myApp/www/build/css, /home/lathonez/code/myApp/www/build/fonts, /home/lathonez/code/myApp/www/build/pages
[11:28:28] Finished 'test.clean' after 417 ms
[11:28:29] Finished 'test.lint' after 1.14 s
[11:28:29] Starting 'sass'...
[11:28:29] Starting 'copy.fonts'...
[11:28:29] Starting 'copy.html'...
[11:28:29] Finished 'copy.html' after 26 ms
[11:28:29] Finished 'copy.fonts' after 29 ms
[11:28:30] Finished 'sass' after 847 ms
[11:28:30] Starting 'test.build.typescript'...
[11:28:32] Finished 'test.build.typescript' after 2.43 s
[11:28:32] Finished 'test.build' after 4.42 s
[11:28:32] Starting 'test.karma'...

START:
04 03 2016 16:58:40.522:INFO [karma]: Karma v0.13.21 server started at http://localhost:9876/
04 03 2016 16:58:40.528:INFO [launcher]: Starting browser PhantomJS
04 03 2016 16:58:40.800:INFO [PhantomJS 2.1.1 (Linux 0.0.0)]: Connected on socket /#T1KfJka_A-62AUmEAAAA with id 60102028
PhantomJS 2.1.1 (Linux 0.0.0) WARN: 'DEPRECATION WARNING: 'enqueueTask' is no longer supported and will be removed in next major release. Use addTask/addRepeatingTask/addMicroTask'
  MyApp
    ✔ initialises with two possible pages

Finished in 0.015 secs / 0.002 secs

SUMMARY:
✔ 1 test completed
-------------------|----------|----------|----------|----------|----------------|
File               |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
-------------------|----------|----------|----------|----------|----------------|
 test/             |       85 |    48.28 |       80 |    93.75 |                |
  app.js           |       85 |    48.28 |       80 |    93.75 |              4 |
 test/pages/page1/ |    82.35 |    48.28 |       75 |    92.31 |                |
  page1.js         |    82.35 |    48.28 |       75 |    92.31 |              4 |
 test/pages/page2/ |    82.35 |    48.28 |       75 |    92.31 |                |
  page2.js         |    82.35 |    48.28 |       75 |    92.31 |              4 |
 test/pages/page3/ |    82.35 |    48.28 |       75 |    92.31 |                |
  page3.js         |    82.35 |    48.28 |       75 |    92.31 |              4 |
 test/pages/tabs/  |    73.91 |    48.28 |       75 |    78.95 |                |
  tabs.js          |    73.91 |    48.28 |       75 |    78.95 |     4,18,19,20 |
-------------------|----------|----------|----------|----------|----------------|
All files          |    80.85 |    48.28 |    76.19 |    89.19 |                |
-------------------|----------|----------|----------|----------|----------------|
```
Add the following lines to your `package.json` so we can get everything working nicely with `npm` instead of calling `gulp`:

```yaml
  "scripts": {
    "postinstall": "typings install",
    "start": "ionic serve",
    "test": "gulp --gulpfile test/gulpfile.ts --cwd ./ test",
    "test.watch": "gulp --gulpfile test/gulpfile.ts --cwd ./ test.watch.build",
    "karma": "gulp --gulpfile test/gulpfile.ts --cwd ./ test.karma.debug"
  }
```

Now you can do `npm test`. This will also install the typings for you on `npm install` which I've found useful.

Test Coverage
--------------

At the end of the `npm test` output you'll see a coverage report table (as above). This gives a good overview, but if you're trying to figure out why your code isn't covered you'll need more.

Our Karma set up outputs [lcov][lcov-home] coverage to `./coverage` in the root folder of your app. If you browse to `/path/to/myApp/coverage/lcov-report/` in a web browser, you'll get [an overview][lcov-index-ss] of all your tested files. [Drill into one][lcov-app-ss] and you get line by line info as to what's covered.

You can also use external tools like [coveralls][clicker-coveralls] or [codecov][clicker-codecov], more on this in an upcoming post **Using [Travis][clicker-travis] with Ionic 2**.

Testing external modules
------------------------

To include external node modules in your testing you need to make a couple of additons to [karma.config.js][karma.config.js].

This example uses moment.js and is taken from the [FAQ][clicker-issue-33]. There are more complicated examples too.

First install the module and any necessary typings:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install --save-dev moment
typings install --ambient --save-dev moment
typings install --ambient --save-dev moment-node</code>
</pre>
</div>

Include it in your test somewhere:

```javascript
import * as moment from 'moment';
```

Add two lines to [karma.config.js][karma.config.js]. The first goes into the `files` array:

```javascript
{ pattern: 'node_modules/moment/moment.js', inclueded: false, watched: false }
```

The second into the `proxies` object. This is what most people forget. Karma will try to find moment at `/base/moment.js`, which doesn't exist and will give a 404. You need to proxy it to the correct location inside node modules:

```javascript
'/base/moment.js': '/base/node_modules/moment/moment.js'
```

Debugging the Tests
--------------------

Sometimes our tests fail and we get an unhelpful stack trace from phantom. This example is from the [FAQ][clicker-issue-6]:

```
FAILED TESTS:
  MyApp
    ✖ initialises with one possible page
      PhantomJS 2.0.0 (Mac OS X 0.0.0)
    TypeError: undefined is not an object (evaluating 'provider.toString') (line 180)
      at 
      at 
      at forEach ([native code])
      at 
      at 
      at 
      at 
      at 
      at /Users/groyer/Documents/workspace/helperchoice/helpizr-app-v2/test/karma/tests.config.js:41:18
```

If this happens it's usually easier to debug yourself in Chrome.

First build the tests and tell gulp to watch for changes. It'll rebuild the tests if you change anything:

`npm run test.watch`

Next start Karma. Calling it like this will invoke Chrome instead of Phantom and keep the browser open after the tests have finished. If Karma detects any changes (from the watch) it'll re-run your tests:

`npm run karma`

Chrome will pop up and run through all your tests. When this is finished, hit the [Debug][karma-debug-ss] button and another tab will open. Open the dev console and you can see the [output of all your tests][karma-console-ss], along with any errors which can be debugged as per usual.

Linting
-------

This set up fully supports linting with [tslint][tslint-home]. Linting is done before compilation; if linting fails, your code does not compile or test.

If you want to use linting, just add a [tslint.json][tslint.json] into your project. Each time your run `npm test`, your code will be fully linted.

Removing tests from the .apk
-----------------------------

In the current setup, the test build residing in `www/build/test` will be compiled into the `.apk` when `ionic build` is run.

See [here][cordova-prune-post] for a short post on how to prevent this from happening.

Contribute
----------

[Clickers][clicker-repo] is a work in progress. If you'd like to help out or have any suggestions, check the [roadmap sticky][clicker-issue-38].

Help!
-----

If you can't get any of this working in your own project, [raise an issue][clicker-issue-new] and I'll do my best to help out. Check the FAQ (directly below) first.

FAQ
---

* [Debugging unit tests example][clicker-issue-6]
* [External modules example one][clicker-issue-20]
* [External modules example two][clicker-issue-22]
* [External modules example three (moment.js)][clicker-issue-33]
* [404 on app.bundle.js][clicker-issue-29]
* [what is app.stub.ts][clicker-issue-34]
* [run a single test][clicker-issue-35]

[analog-clicker-img]: http://thumbs.dreamstime.com/thumblarge_304/1219960995H0ZkZw.jpg
[angular2-seed-cfg]:  https://github.com/mgechev/angular2-seed/blob/master/tools/config.ts
[angular2-seed-repo]: https://github.com/mgechev/angular2-seed
[angular2-sg-dir]:    https://github.com/mgechev/angular2-style-guide#directory-structure
[app.spec.ts]:        https://github.com/lathonez/clicker/blob/master/test/app.spec.ts
[app.stub.ts]:        https://github.com/lathonez/clicker/blob/master/test/app.stub.ts
[clicker-codecov]:    https://codecov.io/github/lathonez/clicker?branch=master
[clicker-coveralls]:  https://coveralls.io/github/lathonez/clicker?branch=master
[clicker-issue-20]:   https://github.com/lathonez/clicker/issues/20
[clicker-issue-22]:   https://github.com/lathonez/clicker/issues/22
[clicker-issue-29]:   https://github.com/lathonez/clicker/issues/29
[clicker-issue-34]:   https://github.com/lathonez/clicker/issues/34
[clicker-issue-33]:   https://github.com/lathonez/clicker/issues/33
[clicker-issue-35]:   https://github.com/lathonez/clicker/issues/35
[clicker-issue-38]:   https://github.com/lathonez/clicker/issues/38
[clicker-issue-6]:    https://github.com/lathonez/clicker/issues/6
[clicker-issue-new]:  https://github.com/lathonez/clicker/issues/new
[clicker-repo]:       http://github.com/lathonez/clicker
[clicker-travis]:     https://travis-ci.org/lathonez/clicker
[config.ts]:          https://github.com/lathonez/clicker/blob/master/test/config.ts
[cordova-prune-post]: http://lathonez.github.io/2016/cordova-remove-assets/
[gulp-home]:          http://gulpjs.com/
[gulp-inline-ng2]:    https://github.com/ludohenin/gulp-inline-ng2-template
[gulpfile.js]:        https://github.com/lathonez/clicker/blob/master/test/gulpfile.js
[gulpfile.ts]:        https://github.com/lathonez/clicker/blob/master/test/gulpfile.ts
[ionic-angular.js]:   https://github.com/lathonez/clicker/blob/master/test/ionic-angular.js
[ionic-index.ts]:     https://github.com/driftyco/ionic/blob/2.0/ionic/index.ts
[karma-console-ss]:   /images/ionic2_unit_testing/karma-console-screenshot.png
[karma-debug-ss]:     /images/ionic2_unit_testing/karma-debug-screenshot.png
[karma-home]:         https://karma-runner.github.io/0.13/index.html
[karma-tm-docs]:      https://karma-runner.github.io/0.8/plus/RequireJS.html
[karma.config.js]:    https://github.com/lathonez/clicker/blob/master/test/karma.config.js
[lcov-app-ss]:        /images/ionic2_unit_testing/lcov-app-screenshot.png
[lcov-home]:          http://ltp.sourceforge.net/coverage/lcov.php
[lcov-index-ss]:      /images/ionic2_unit_testing/lcov-index-screenshot.png
[sbtp-docs]:          https://angular.io/docs/js/latest/api/testing/setBaseTestProviders-function.html
[strict-typing]:      https://github.com/lathonez/clicker/blob/master/tslint.json#L80-L97
[test-main.js]:       https://github.com/lathonez/clicker/blob/master/test/test-main.js
[tslint-home]:        https://www.npmjs.com/package/tslint
[tslint.json]:        https://github.com/lathonez/clicker/blob/master/tslint.json
[typings-home]:       https://www.npmjs.com/package/typings