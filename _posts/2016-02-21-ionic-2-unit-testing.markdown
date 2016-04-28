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

![alt text](/images/ionic2_unit_testing/clicker.gif "Clicker App"){: .center-image }

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
├── gulpfile.ts
├── karma.config.js
├── testUtils.ts
└── karma-static
    ├── context.html
    └── debug.html

```

This may not be the correct decision for your project / team. If you decide to keep the source and tests separately, you should still be able to follow this post with minimal config changes.

A simple unit test on app.ts
----------------------------

We probably haven't got much logic in `app.ts` worthy of unit testing, but being the root source file it makes a sensible starting point.

`cp clicker/app/app.spec.ts myApp/app/app.spec.ts`

Modify the test cases in [app.spec.ts][app.spec.ts] to suit your application, or use the simple example below:

```javascript
// http://stackoverflow.com/questions/33332394/angular-2-typescript-cant-find-names
/// <reference path="../node_modules/angular2/typings/browser.d.ts" />

import { TEST_BROWSER_PLATFORM_PROVIDERS, TEST_BROWSER_APPLICATION_PROVIDERS} from 'angular2/platform/testing/browser';
import { setBaseTestProviders } from 'angular2/testing';
import { MyApp } from './app';

// this needs doing _once_ for the entire test suite, hence it's here
setBaseTestProviders(TEST_BROWSER_PLATFORM_PROVIDERS, TEST_BROWSER_APPLICATION_PROVIDERS);

// Mock out Ionic's platform class
class MockClass {
  public ready(): any {
    return new Promise((resolve: Function) => {
      resolve();
    });
  }
}

let myApp = null;

describe('MyApp', () => {

  beforeEach(function() {
    let platform = (<any>new MockClass());
    myApp = new MyApp(platform);
  });

  it('initialises with two possible pages', () => {
    expect(myApp).not.toBeNull();
  });
});
```

Note the [setBaseTestProviders][sbtp-docs] line, enabling us to utilise Angular 2's testing framework in our tests. See this [excellent blog post][angular2-di-testing] for more info.

Building the tests
-------------------

We'll use [gulp][gulp-home] to orchastrate the test process and [browserify][browserify-home] to transpile the unit tests and generate sourcemaps.

Ionic use both gulp and browserify already; you should have `gulpfile.js` inside the root of your app. In order to separate concerns we'll keep ours in `./test/gulpfile.ts`. Feel free to combine the two if you wish, ours will actually hook directly into Ionic's to make use of the tasks defined therein.

Make the following changes to your project:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>mkdir -p myApp/test
cp clicker/test/gulpfile.ts myApp/test
cp clicker/test/karma.config.js myApp/test
cp -r clicker/test/karma-static myApp/test</code>
</pre>
</div>

* [gulpfile.ts][gulpfile.ts]: gulp’s task definition file
* [karma.config.js][karma.config.js]: Karma's config
* [karma-static][karma-static]: Karma's HTML static with an `<ion-app></ion-app>` injected. See [here][clicker-issue-79] for more info

This gulpfile defines several tasks which gulp will carry out for us during the test cycle:

1. **clean-test**: remove content from `www/build`, except `www/build/js`, which isn't used for testing
2. **lint**: perform static analysis on source code using `tslint` if enabled
3. **karma**: spin up [Karma][karma-home]
4. **unit-test**: combine all of the above to run the tests

**Install Dependencies and Typings:**

The project's [README.md][clicker-deps] has list of dependencies and a brief description of what each is for

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install -g typings
npm install --save-dev browserify-istanbul codecov.io gulp-tslint gulp-typescript isparta jasmine-core karma karma-browserify karma-chrome-launcher karma-coverage karma-jasmine karma-mocha-reporter karma-phantomjs-launcher phantomjs-prebuilt traceur tsify ts-node tslint</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="lowlight">
<code>typings install --ambient --save jasmine node</code>
</pre>
</div>

**Patch Karma's static:**

`cp test/karma-static/*.html node_modules/karma/static`

Running the tests
------------------

`node_modules/gulp/bin/gulp.js --gulpfile test/gulpfile.ts --cwd ./ unit-test`

```
[22:20:58] Requiring external module ts-node/register
[22:21:02] sourced Ionic's gulpfile @ /home/lathonez/code/clicker/gulpfile.js
[22:22:09] Using gulpfile ~/code/clicker/test/gulpfile.ts
[22:22:09] Starting 'unit-test'...
[22:22:09] Starting 'clean-test'...
[22:22:09] Deleted /home/lathonez/code/clicker/www/build/test
[22:21:02] Starting 'unit-test'...
[22:21:02] Starting 'html'...
[22:21:02] Finished 'html' after 25 ms
[22:21:02] Starting 'karma'...

START:
28 04 2016 22:21:14.274:INFO [framework.browserify]: bundle built
28 04 2016 22:21:14.297:INFO [karma]: Karma v0.13.9 server started at http://localhost:9876/
28 04 2016 22:21:14.302:INFO [launcher]: Starting browser PhantomJS
28 04 2016 22:21:14.507:INFO [PhantomJS 2.1.1 (Linux 0.0.0)]: Connected on socket 7iyg7ThvlYkHrCl3AAAA with id 42386869
  ClickerApp
    ✔ initialises with two possible pages
    ✔ initialises with a root page
    ✔ initialises with an app
    ✔ opens a page
LOG: 'Angular 2 is running in the development mode. Call enableProdMode() to enable the production mode.'
  ClickerButton
    ✔ initialises
    ✔ displays the clicker name and count
    ✔ does a click
  ClickerForm
    ✔ initialises
    ✔ passes new clicker through to service
    ✔ doesn't try to add a clicker with no name
  Click
    ✔ initialises with defaults
    ✔ initialises with overrides
  Clicker
    ✔ initialises with the correct name
  ClickerList
    ✔ initialises
  Page2
    ✔ initialises
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
  Utils
    ✔ resets a control

Finished in 0.634 secs / 0.62 secs

SUMMARY:
✔ 27 tests completed
-------------------------------|----------|----------|----------|----------|----------------|
File                           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
-------------------------------|----------|----------|----------|----------|----------------|
 app/                          |      100 |      100 |      100 |      100 |                |
  app.ts                       |      100 |      100 |      100 |      100 |                |
 app/components/clickerButton/ |      100 |      100 |      100 |      100 |                |
  clickerButton.ts             |      100 |      100 |      100 |      100 |                |
 app/components/clickerForm/   |      100 |      100 |      100 |      100 |                |
  clickerForm.ts               |      100 |      100 |      100 |      100 |                |
 app/models/                   |      100 |      100 |      100 |      100 |                |
  click.ts                     |      100 |      100 |      100 |      100 |                |
  clicker.ts                   |      100 |      100 |      100 |      100 |                |
 app/pages/clickerList/        |      100 |      100 |      100 |      100 |                |
  clickerList.ts               |      100 |      100 |      100 |      100 |                |
 app/pages/page2/              |      100 |      100 |      100 |      100 |                |
  page2.ts                     |      100 |      100 |      100 |      100 |                |
 app/services/                 |    95.83 |    96.77 |       90 |    95.95 |                |
  clickers.ts                  |    98.77 |    96.77 |      100 |    98.39 |             35 |
  utils.ts                     |       80 |      100 |       40 |    83.33 |           8,10 |
-------------------------------|----------|----------|----------|----------|----------------|
All files                      |    98.36 |    99.45 |    95.38 |    98.18 |                |
-------------------------------|----------|----------|----------|----------|----------------|

[22:21:15] Finished 'karma' after 14 s
[22:21:15] Finished 'unit-test' after 14 s

```
Congrats! You now have unit tests working on your Ionic 2 Project!

Add the following lines to your `package.json` to get everything working nicely with `npm` instead of calling `gulp` directly:

```yaml
  "scripts": {
    "karma": "gulp --gulpfile test/gulpfile.ts --cwd ./ karma-debug",
    "postinstall": "typings install && cp test/karma-static/*.html node_modules/karma/static",
    "test": "gulp --gulpfile test/gulpfile.ts --cwd ./ unit-test",
    "watch": "gulp --gulpfile test/gulpfile.ts --cwd ./ watch-unit"
  }
```

Now you can simply run `npm test`. Also when you run `npm install` in future, it will also install the typings and patch Karma.

Test Coverage
--------------

At the end of the `npm test` output you'll see a coverage report table (as above). This gives a good overview, but if you're trying to figure out why your code isn't covered you'll need more.

This set up outputs [lcov][lcov-home] coverage to `./coverage` in the root folder of your app. If you browse to `/path/to/myApp/coverage/lcov-report/` in a web browser, you'll get [an overview][lcov-index-ss] of all your tested files. [Drill into one][lcov-app-ss] and you get line by line info as to what's covered.

You can also use external tools, I highlighy recommend [codecov][clicker-codecov].

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

`npm run watch`

Next start Karma. Calling it like this will invoke Chrome instead of Phantom and keep the browser open after the tests have finished. If Karma detects any changes (from the watch) it'll re-run your tests:

`npm run karma`

Chrome will pop up and run through all your tests. When this is finished, hit the [Debug][karma-debug-ss] button and another tab will open. Open the dev console and you can see the [output of all your tests][karma-console-ss], along with any errors which can be debugged as per usual.

Linting
-------

This set up fully supports linting with [tslint][tslint-home]. Linting is done before compilation; if linting fails, your code does not compile or test.

If you want to use linting, just add a [tslint.json][tslint.json] into your project. Each time your run `npm test` your code will be fully linted.

Removing tests from the .apk
-----------------------------

In this set up, test files residing in `www/build/test` will be bundled into the `.apk` when `ionic build` is run.

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
* [404 on app.bundle.js][clicker-issue-29]
* [what is app.stub.ts][clicker-issue-34]
* [run a single test][clicker-issue-35]
* [why do our tests need a main method (they don't anymore - for posterity)][clicker-issue-65]

<div align="center"><iframe src="https://ghbtns.com/github-btn.html?user=lathonez&repo=clicker&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe></div>

[analog-clicker-img]: http://thumbs.dreamstime.com/thumblarge_304/1219960995H0ZkZw.jpg
[angular2-di-testing]:https://developers.livechatinc.com/blog/testing-angular-2-apps-dependency-injection-and-components
[angular2-seed-cfg]:  https://github.com/mgechev/angular2-seed/blob/master/tools/config.ts
[angular2-seed-repo]: https://github.com/mgechev/angular2-seed
[angular2-sg-dir]:    https://github.com/mgechev/angular2-style-guide#directory-structure
[app.spec.ts]:        https://github.com/lathonez/clicker/blob/master/app/app.spec.ts
[app.stub.js]:        https://github.com/lathonez/clicker/blob/master/test/app.stub.js
[browserify-home]:    http://browserify.org/
[clicker-codecov]:    https://codecov.io/github/lathonez/clicker?branch=master
[clicker-coveralls]:  https://coveralls.io/github/lathonez/clicker?branch=master
[clicker-issue-6]:    https://github.com/lathonez/clicker/issues/6
[clicker-issue-29]:   https://github.com/lathonez/clicker/issues/29
[clicker-issue-34]:   https://github.com/lathonez/clicker/issues/34
[clicker-issue-35]:   https://github.com/lathonez/clicker/issues/35
[clicker-issue-38]:   https://github.com/lathonez/clicker/issues/38
[clicker-issue-65]:   https://github.com/lathonez/clicker/issues/65
[clicker-issue-79]:   https://github.com/lathonez/clicker/issues/79
[clicker-issue-new]:  https://github.com/lathonez/clicker/issues/new
[clicker-deps]:       https://github.com/lathonez/clicker#dependencies
[clicker-repo]:       http://github.com/lathonez/clicker
[clicker-travis]:     https://travis-ci.org/lathonez/clicker
[config.ts]:          https://github.com/lathonez/clicker/blob/master/test/config.ts
[cordova-prune-post]: http://lathonez.github.io/2016/cordova-remove-assets/
[gulp-home]:          http://gulpjs.com/
[gulpfile.ts]:        https://github.com/lathonez/clicker/blob/master/test/gulpfile.ts
[karma-console-ss]:   /images/ionic2_unit_testing/karma-console-screenshot.png
[karma-debug-ss]:     /images/ionic2_unit_testing/karma-debug-screenshot.png
[karma-home]:         https://karma-runner.github.io/0.13/index.html
[karma-static]:       https://github.com/lathonez/clicker/tree/master/test/karma-static
[karma-tm-docs]:      https://karma-runner.github.io/0.8/plus/RequireJS.html
[karma.config.js]:    https://github.com/lathonez/clicker/blob/master/test/karma.config.js
[lcov-app-ss]:        /images/ionic2_unit_testing/lcov-app-screenshot.png
[lcov-home]:          http://ltp.sourceforge.net/coverage/lcov.php
[lcov-index-ss]:      /images/ionic2_unit_testing/lcov-index-screenshot.png
[sbtp-docs]:          https://angular.io/docs/js/latest/api/testing/setBaseTestProviders-function.html
[tslint-home]:        https://www.npmjs.com/package/tslint
[tslint.json]:        https://github.com/lathonez/clicker/blob/master/tslint.json
[typings-home]:       https://www.npmjs.com/package/typings