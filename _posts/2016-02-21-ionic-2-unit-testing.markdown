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
├── ionic-angular.js
├── karma.config.js
├── test-main.js
└── testUtils.ts
```

This may not be the correct decision for your project / team. If you decide to keep the source and tests separately, you should still be able to follow this post with minimal config changes.

A simple unit test on app.ts
----------------------------

We probably haven't got much logic in app.ts worthy of unit testing. However, app.ts is the root source file, if we include it in our test suite, all other spec that we write will be included too.

`cp clicker/app/app.spec.ts myApp/app/app.spec.ts`

Modify the test cases in [app.spec.ts][app.spec.ts] to suit your application, or use the simple example below:

```javascript
import { TEST_BROWSER_PLATFORM_PROVIDERS, TEST_BROWSER_APPLICATION_PROVIDERS} from 'angular2/platform/testing/browser';
import { setBaseTestProviders } from 'angular2/testing';
import { IonicApp, Platform }   from 'ionic-angular/index';
import { MyApp }           from '../app/app';

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
<code>cp clicker/gulpfile.ts myApp/
mkdir -p myApp/test
cp clicker/config.ts myApp/test</code>
</pre>
</div>

* [gulpfile.ts][gulpfile.ts]: gulp’s config file
* [config.ts][config.ts]: config file for this setup, containing only paths at the moment. Can be [expanded][angular2-seed-cfg] down the line.

This gulpfile defines several tasks which gulp will carry out for us during the test cycle:

1. **test.clean**: remove content from `www/build`, except `www/build/js`, which isn't used for testing
2. **test.lint**: perform static analysis on source code using `tslint` if enabled
3. **test.build.[fonts,html,sass]**: use `ionic-app-lib` to build fonts, html or sass
4. **test.build.locker**: prevent any progress until Ionic builds are completed
5. **test.build.typescript**: compile all our Typescript (both source and test), into individual Javascript files. This is done differently from Ionic which bundles everything into `app.bundle.js`
6. **test.karma**: spin up [Karma][karma-home] (more on this later)
7. **test**: combine all of the above to run the tests (more on this later)

**Install Dependencies and Typings:**

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install -g ionic-app-lib
npm install --save-dev chalk del gulp gulp-load-plugins gulp-inline-ng2-template gulp-tslint gulp-typescript karma run-sequence tslint ts-node typings</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="lowlight">
<code>typings install --ambient --save github:lathonez/typings/ionic-app-lib.d.ts
for typing in \
bluebird chalk del es6-shim express glob gulp gulp-load-plugins gulp-typescript \
gulp-util jasmine karma log4js mime minimatch node orchestrator q \
run-sequence serve-static through2 vinyl
do
./node_modules/typings/dist/bin/typings.js install $typing --save-dev --ambient --no-insight
done</code>
</pre>
</div>

You're now ready to build the tests:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>for task in \
test.clean test.lint test.build.fonts test.build.html test.build.sass test.build.locker test.build.typescript
do
node_modules/gulp/bin/gulp.js $task
done</code>
</pre>
</div>

```
[13:35:35] Requiring external module ts-node/register
[13:35:39] Using gulpfile ~/code/clicker/gulpfile.ts
[13:35:39] Starting 'test.clean'...
[13:35:39] Deleted /home/lathonez/code/clicker/www/build/components, /home/lathonez/code/clicker/www/build/pages
[13:35:39] Finished 'test.clean' after 13 ms
[13:35:39] Requiring external module ts-node/register
[13:35:42] Using gulpfile ~/code/clicker/gulpfile.ts
[13:35:42] Starting 'test.lint'...
[13:35:43] Finished 'test.lint' after 1.25 s
[13:35:44] Requiring external module ts-node/register
[13:35:47] Using gulpfile ~/code/clicker/gulpfile.ts
[13:35:47] Starting 'test.build.fonts'...

∆ Copying fonts
√ Matching patterns: node_modules/ionic-angular/fonts/**/*.+(ttf|woff|woff2)
[13:35:47] Finished 'test.build.fonts' after 12 ms
√ Fonts copied to www/build/fonts
[13:35:47] Requiring external module ts-node/register
[13:35:50] Using gulpfile ~/code/clicker/gulpfile.ts
[13:35:50] Starting 'test.build.html'...

∆ Copying HTML
√ Matching patterns: app/**/*.html
[13:35:50] Finished 'test.build.html' after 13 ms
√ HTML copied to www/build
[13:35:51] Requiring external module ts-node/register
[13:35:54] Using gulpfile ~/code/clicker/gulpfile.ts
[13:35:54] Starting 'test.build.sass'...

∆ Compiling Sass to CSS
√ Matching patterns: app/theme/app.+(ios|md).scss
[13:35:54] Finished 'test.build.sass' after 14 ms
√ Sass compilation complete
[13:35:55] Requiring external module ts-node/register
[13:35:59] Using gulpfile ~/code/clicker/gulpfile.ts
[13:35:59] Starting 'test.build.locker'...
[13:35:59] buildLocker: unlocked
[13:35:59] Finished 'test.build.locker' after 444 μs
[13:35:59] Requiring external module ts-node/register
[13:36:02] Using gulpfile ~/code/clicker/gulpfile.ts
[13:36:02] Starting 'test.build.typescript'...
[13:36:05] Finished 'test.build.typescript' after 2.88 s
```

To verify that the compilation has succeed as planned, inspect `www/build`. You should see the following files (each indicating that their part of the build succeeded):

```
www/build/
├── app.html
├── css
│   ├── app.md.css
├── fonts
│   ├── ionicons.ttf
└── test
    ├── app.js
    └── app.spec.js
```

Running the tests
------------------

To get [Karma][karma-home] up and runnning, we need more boilerplate config and more dev dependencies.

Copy the following files into your project:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>cp clicker/test/app.stub.ts clicker/test/ionic-angular.js clicker/test/clicker/test/karma.config.js clicker/test/test-main.js myApp/test</code>
</pre>
</div>

* [app.stub.ts][app.stub.ts]: A stub for Ionic's @App decorator.
* [ionic-angular.js][ionic-angular.js]: A stub for Ionic's [index.ts][ionic-index.ts] so we don't have to proxy their modules in Karma.
* [karma.config.js][karma.config.js]: Karma's config
* [test-main.js][test-main.js]: Main entry point for [unit test excution using RequireJS][karma-tm-docs].

**Install deps:**

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install --save-dev es6-module-loader jasmine-core karma-coverage karma-jasmine karma-mocha-reporter karma-phantomjs-launcher phantomjs-prebuilt systemjs traceur</code>
</pre>
</div>

Now we're ready to test: `node_modules/gulp/bin/gulp.js test`

```
[13:47:03] Requiring external module ts-node/register
[13:47:06] Using gulpfile ~/code/clicker/gulpfile.ts
[13:47:06] Starting 'test'...
[13:47:06] Starting 'test.clean'...
[13:47:06] Starting 'test.lint'...
[13:47:07] Deleted /home/lathonez/code/clicker/www/build/app.html, /home/lathonez/code/clicker/www/build/components, /home/lathonez/code/clicker/www/build/css, /home/lathonez/code/clicker/www/build/fonts, /home/lathonez/code/clicker/www/build/pages
[13:47:07] Finished 'test.clean' after 491 ms
[13:47:08] Finished 'test.lint' after 1.25 s
[13:47:08] Starting 'test.build.html'...

∆ Copying HTML
√ Matching patterns: app/**/*.html
[13:47:08] Finished 'test.build.html' after 11 ms
[13:47:08] Starting 'test.build.fonts'...

∆ Copying fonts
√ Matching patterns: node_modules/ionic-angular/fonts/**/*.+(ttf|woff|woff2)
[13:47:08] Finished 'test.build.fonts' after 2.07 ms
[13:47:08] Starting 'test.build.sass'...

∆ Compiling Sass to CSS
√ Matching patterns: app/theme/app.+(ios|md).scss
[13:47:08] Finished 'test.build.sass' after 3.83 ms
[13:47:08] Starting 'test.build.locker'...
[13:47:08] buildLocker: waiting for fonts build to complete
√ HTML copied to www/build
√ Fonts copied to www/build/fonts
[13:47:08] buildLocker: waiting for css build to complete
[13:47:08] buildLocker: waiting for css build to complete
[13:47:08] buildLocker: waiting for css build to complete
[13:47:09] buildLocker: waiting for css build to complete
√ Sass compilation complete
[13:47:09] buildLocker: unlocked
[13:47:09] Finished 'test.build.locker' after 968 ms
[13:47:09] Starting 'test.build.typescript'...
[13:47:12] Finished 'test.build.typescript' after 2.79 s
[13:47:12] Starting 'test.karma'...

START:
04 03 2016 13:47:13.357:INFO [karma]: Karma v0.13.21 server started at http://localhost:9876/
04 03 2016 13:47:13.365:INFO [launcher]: Starting browser PhantomJS
04 03 2016 13:47:13.637:INFO [PhantomJS 2.1.1 (Linux 0.0.0)]: Connected on socket /#cEaU3-8LQtWDI375AAAA with id 2734143
  Click
    ✔ initialises with defaults
    ✔ initialises with overrides
  Clicker
    ✔ initialises with the correct name
  ClickerApp
    ✔ initialises with two possible pages
    ✔ initialises with a root page
    ✔ initialises with an app
    ✔ opens a page
  ClickerButton
    ✔ initialises
    ✔ displays the clicker name and count
    ✔ does a click
  ClickerForm
    ✔ initialises
    ✔ passes new clicker through to service
    ✔ doesn't try to add a clicker with no name
  Clickers
    ✔ initialises with empty clickers
    ✔ creates an instance of SqlStorage
    ✔ has empty ids with no storage
    ✔ has empty clickers with no storage
    ✔ can initialise a clicker from string
    ✔ returns undefined for a bad id
    ✔ adds a new clicker with the correct name
    ✔ removes a clicker by id
    ✔ does a click
    ✔ loads IDs from storage
    ✔ loads clickers from storage
  ClickerList
    ✔ initialises
  Utils
    ✔ resets a control

Finished in 0.413 secs / 0.356 secs

SUMMARY:
✔ 26 tests completed
--------------------------------|----------|----------|----------|----------|----------------|
File                            |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
--------------------------------|----------|----------|----------|----------|----------------|
 test/                          |    89.74 |    48.39 |       80 |    94.29 |                |
  app.js                        |    89.66 |    48.28 |    85.71 |       96 |              5 |
  testUtils.js                  |       90 |       50 |    66.67 |       90 |              8 |
 test/components/clickerButton/ |       85 |    48.28 |      100 |    93.75 |                |
  clickerButton.js              |       85 |    48.28 |      100 |    93.75 |              5 |
 test/components/clickerForm/   |    90.32 |    51.61 |      100 |     96.3 |                |
  clickerForm.js                |    90.32 |    51.61 |      100 |     96.3 |              5 |
 test/models/                   |      100 |      100 |      100 |      100 |                |
  click.js                      |      100 |      100 |      100 |      100 |                |
  clicker.js                    |      100 |      100 |      100 |      100 |                |
 test/pages/clickerList/        |    86.96 |    48.28 |      100 |    94.74 |                |
  clickerList.js                |    86.96 |    48.28 |      100 |    94.74 |              5 |
 test/pages/page2/              |    82.35 |    48.28 |       75 |    92.31 |                |
  page2.js                      |    82.35 |    48.28 |       75 |    92.31 |              5 |
 test/services/                 |    92.55 |    48.39 |    86.67 |    95.35 |                |
  clickers.js                   |    94.94 |    48.39 |       96 |    97.22 |           5,67 |
  utils.js                      |       80 |      100 |       40 |    85.71 |            6,8 |
--------------------------------|----------|----------|----------|----------|----------------|
All files                       |    90.87 |       50 |    89.71 |    95.54 |                |
--------------------------------|----------|----------|----------|----------|----------------|
```
Add the following lines to your `package.json` so we can get everything working nicely with `npm` instead of calling `gulp`:

```yaml
  "scripts": {
    "postinstall": "typings install",
    "start": "ionic serve",
    "test": "gulp test"
  }
```

Now you can do `npm test`. This will also install the typings for you on `npm install` which I've found useful.

Test Coverage
--------------

At the end of the `npm test` output you'll see a coverage report table (as above). This gives a good overview, but if you're trying to figure out why your code isn't covered you'll need more.

Our Karma set up outputs [lcov][lcov-home] coverage to `./coverage` in the root folder of your app. If you browse to `/path/to/myApp/coverage/lcov-report/` in a web browser, you'll get [an overview][lcov-index-ss] of all your tested files. [Drill into one][lcov-app-ss] and you get line by line info as to what's covered.

You can also use external tools like [coveralls][clicker-coveralls] or [codecov][clicker-codecov], more on this in an upcoming post **Using [Travis][clicker-travis] with Ionic 2**.

Linting
-------

This set up fully supports linting with [tslint][tslint-home]. Linting is done before compilation; if linting fails, your code does not compile or test.

If you want to use linting, just add a [tslint.json][tslint.json] into your project. Each time your run `npm test`, your code will be fully linted.

I get the following output if I turn linting on for the Ionic Starter App:

```
[00:02:00] Using gulpfile ~/code/myApp/gulpfile.ts
[00:02:00] Starting 'test.lint'...
[00:02:00] Finished 'test.lint' after 8.41 ms
[00:02:00] Starting 'test.clean'...
[00:02:00] [gulp-tslint] error (no-unused-variable) app.spec.ts[3, 10]: unused variable: 'IonicApp'
[00:02:00] [gulp-tslint] error (typedef) app.spec.ts[9, 10]: expected variable-declaration: 'myApp' to have a typedef
[00:02:00] [gulp-tslint] error (typedef) app.spec.ts[11, 22]: expected call-signature: 'main' to have a typedef
[00:02:00] [gulp-tslint] error (typedef) app.spec.ts[15, 25]: expected call-signature to have a typedef
[00:02:00] [gulp-tslint] error (typedef) app.spec.ts[16, 19]: expected variable-declaration: 'platform' to have a typedef
[00:02:00] [gulp-tslint] error (use-strict) app.spec.ts[11, 1]: missing 'use strict'
[00:02:00] [gulp-tslint] error (member-access) app.ts[13, 3]: default access modifier on member/method not allowed
[00:02:00] [gulp-tslint] error (no-consecutive-blank-lines) app.ts[7, 1]: consecutive blank lines are disallowed
[00:02:00] [gulp-tslint] error (trailing-comma) app.ts[10, 12]: missing trailing comma
[00:02:00] Finished 'test.clean' after 273 ms
[00:02:00] Starting 'test.compile'...
[00:02:00] Starting 'test.copyHTML'...
[00:02:00] Finished 'test.copyHTML' after 705 μs
[00:02:00] [gulp-tslint] error (no-consecutive-blank-lines) pages/page1/page1.ts[3, 1]: consecutive blank lines are disallowed
[00:02:00] [gulp-tslint] error (no-consecutive-blank-lines) pages/page2/page2.ts[3, 1]: consecutive blank lines are disallowed
[00:02:00] [gulp-tslint] error (no-consecutive-blank-lines) pages/page3/page3.ts[3, 1]: consecutive blank lines are disallowed
[00:02:00] [gulp-tslint] error (trailing-comma) pages/page3/page3.ts[5, 45]: missing trailing comma
[00:02:00] [gulp-tslint] error (member-access) pages/tabs/tabs.ts[13, 3]: default access modifier on member/method not allowed
[00:02:00] [gulp-tslint] error (member-access) pages/tabs/tabs.ts[14, 3]: default access modifier on member/method not allowed
[00:02:00] [gulp-tslint] error (member-access) pages/tabs/tabs.ts[15, 3]: default access modifier on member/method not allowed
[00:02:00] [gulp-tslint] error (no-consecutive-blank-lines) pages/tabs/tabs.ts[6, 1]: consecutive blank lines are disallowed
[00:02:00] [gulp-tslint] error (trailing-comma) pages/tabs/tabs.ts[8, 43]: missing trailing comma
```

Removing tests from the .apk
-----------------------------

In the current setup, the test build residing in `www/build/test` will be compiled into the `.apk` when `ionic build` is run.

See [here][cordova-prune-post] for a short post on how to prevent this from happening.

Help!
-----

If you can't get any of this working in your own project, [raise an issue][clicker-issue-new] and I'll do my best to help out.

FAQ
---

* [How to debug PhantomJS stack trace][clicker-issue-6]
* [How to test external node modules][clicker-issue-20]
* [External node modules again][clicker-issue-22]
* [404 on app.bundle.js][clicker-issue-29]

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
[clicker-issue-6]:    https://github.com/lathonez/clicker/issues/6
[clicker-issue-new]:  https://github.com/lathonez/clicker/issues/new
[clicker-repo]:       http://github.com/lathonez/clicker
[clicker-travis]:     https://travis-ci.org/lathonez/clicker
[config.ts]:          https://github.com/lathonez/clicker/blob/master/test/config.ts
[cordova-prune-post]: http://lathonez.github.io/2016/cordova-remove-assets/
[gulp-home]:          http://gulpjs.com/
[gulp-inline-ng2]:    https://github.com/ludohenin/gulp-inline-ng2-template
[gulpfile.ts]:        https://github.com/lathonez/clicker/blob/master/gulpfile.ts
[ionic-angular.js]:   https://github.com/lathonez/clicker/blob/master/test/ionic-angular.js
[ionic-index.ts]:     https://github.com/driftyco/ionic/blob/2.0/ionic/index.ts
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