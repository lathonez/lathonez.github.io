---
title:  "Unit Testing an Ionic2 project"
date:   2016-05-22 01:48:23
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

There is some debate around where to keep unit tests. I started off with the unit tests in a separate `test/` directory, but have since combined with the source code as per the [Angular 2 Style Guide][angular2-sg-dir]. Now I use `test/` for configs, etc purely pertaining to the test setup.

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
import { ADDITIONAL_TEST_BROWSER_PROVIDERS, TEST_BROWSER_STATIC_PLATFORM_PROVIDERS } from '@angular/platform-browser/testing/browser_static';
import { BROWSER_APP_DYNAMIC_PROVIDERS }                from '@angular/platform-browser-dynamic';
import { resetBaseTestProviders, setBaseTestProviders } from '@angular/core/testing';
import { MyApp } from './app';

// this needs doing _once_ for the entire test suite, hence it's here
resetBaseTestProviders();
setBaseTestProviders(
  TEST_BROWSER_STATIC_PLATFORM_PROVIDERS,
  [
    BROWSER_APP_DYNAMIC_PROVIDERS,
    ADDITIONAL_TEST_BROWSER_PROVIDERS,
  ]
);

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
------------------

We'll use [gulp][gulp-home] to orchastrate the test process and [browserify][browserify-home] to transpile the unit tests and generate sourcemaps.

Ionic use both gulp and browserify already; you should have `gulpfile.js` in your app's root directory. In order to separate concerns we keep ours in `./test/gulpfile.ts`

Add the following files to your project:

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
* [karma-static][karma-static]: Karma's HTML static with an `<ion-app></ion-app>` injected. See [here][clicker-issue-79] for more info.

This gulpfile defines several tasks which gulp will carry out during the unit-test cycle:

1. **lint**: perform static analysis on source code using [tslint][tslint-home]
2. **html**: hook into Ionic's gulpfile to copy html files
3. **karma**: spin up [Karma][karma-home], which calls browserify before running the tests
4. **unit-test**: sequence of the above tasks to run the tests

**Install Dependencies and Typings:**

The project's [README.md][clicker-deps] has list of dependencies and a brief description of what each is for

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install -g typings
npm install --save-dev browserify-istanbul codecov.io gulp-tslint gulp-typescript isparta jasmine-core karma karma-browserify karma-chrome-launcher karma-coverage karma-jasmine karma-mocha-reporter karma-phantomjs-launcher phantomjs-prebuilt traceur tsify ts-node tslint
typings install --save --global registry:dt/jasmine registry:dt/node</code>
</pre>
</div>

**Patch Karma's static:**

We need to have an `<ion-app></ion-app>` hook in our test runner's HTML, otherwise we'll get errors when running the tests. See [this issue][clicker-issue-79] for more information and [this one][clicker-issue-99] for what happens if you remove it.

`cp test/karma-static/*.html node_modules/karma/static`

**Linting:**

By default we're running linting with [these tslint rules][tslint.json]. You may find this fails with your project, in which case your tests wont run.

To disable linting, just remove the `lint` task from the array below in [gulpfile.ts][gulpfile.ts].

```javascript
// build unit tests, run unit tests, remap and report coverage
gulp.task('unit-test', (done: Function) => {
  runSequence(
    ['lint', 'html'],
    'karma',
    (<any>done)
  );
});
```

Running the tests
------------------

`node_modules/gulp/bin/gulp.js --gulpfile test/gulpfile.ts --cwd ./ unit-test`

```
[00:05:13] Requiring external module ts-node/register
[00:05:16] Using gulpfile ~/code/myApp/test/gulpfile.ts
[00:05:16] Starting 'unit-test'...
[00:05:16] Starting 'html'...
[00:05:16] Finished 'html' after 28 ms
[00:05:16] Starting 'karma'...

START:
30 04 2016 00:05:28.914:INFO [framework.browserify]: bundle built
30 04 2016 00:05:28.937:INFO [karma]: Karma v0.13.9 server started at http://localhost:9876/
30 04 2016 00:05:28.941:INFO [launcher]: Starting browser PhantomJS
30 04 2016 00:05:29.178:INFO [PhantomJS 2.1.1 (Linux 0.0.0)]: Connected on socket CvuugedPDP7gbjxEAAAA with id 98871511
  MyApp
    ✔ initialises with two possible pages

Finished in 0.007 secs / 0.001 secs

SUMMARY:
✔ 1 test completed
------------------|----------|----------|----------|----------|----------------|
File              |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
------------------|----------|----------|----------|----------|----------------|
 app/             |    95.45 |      100 |       80 |    90.91 |                |
  app.ts          |    95.45 |      100 |       80 |    90.91 |             17 |
 app/pages/page1/ |      100 |      100 |       75 |      100 |                |
  page1.ts        |      100 |      100 |       75 |      100 |                |
 app/pages/page2/ |      100 |      100 |       75 |      100 |                |
  page2.ts        |      100 |      100 |       75 |      100 |                |
 app/pages/page3/ |      100 |      100 |       75 |      100 |                |
  page3.ts        |      100 |      100 |       75 |      100 |                |
 app/pages/tabs/  |    86.96 |      100 |       75 |    72.73 |                |
  tabs.ts         |    86.96 |      100 |       75 |    72.73 |       13,14,15 |
------------------|----------|----------|----------|----------|----------------|
All files         |    95.83 |      100 |    76.19 |       90 |                |
------------------|----------|----------|----------|----------|----------------|

[00:05:29] Finished 'karma' after 13 s
[00:05:29] Finished 'unit-test' after 13 s
```

Congrats! You now have unit testing working in your Ionic 2 project.

Finally, add the following lines to your `package.json` to get everything working nicely with `npm` instead of calling `gulp` directly:

```yaml
  "scripts": {
    "karma": "gulp --gulpfile test/gulpfile.ts --cwd ./ karma-debug",
    "postinstall": "typings install && cp test/karma-static/*.html node_modules/karma/static",
    "test": "gulp --gulpfile test/gulpfile.ts --cwd ./ unit-test"
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

If this happens it's usually easier to debug in a a real browser.

`npm run karma`

This will invoke Chrome instead of Phantom and keep the browser open after the tests have finished. If Karma detects any changes it'll re-run your tests.

Hit the [Debug][karma-debug-ss] button and another tab will open. Open the dev console and you can see the [output of all your tests][karma-console-ss], along with any errors which can be debugged as per usual.

Contribute
----------

[Clickers][clicker-repo] is a work in progress. If you'd like to help out or have any suggestions, check the [roadmap sticky][clicker-issue-38].

This blog is [on github][blog-repo], if you can improve it, have any suggestions or I've failed to keep it up to date, [raise an issue][blog-issue-new] or a PR.

Help!
-----

If you can't get any of this working in your own project, [raise an issue][clicker-issue-new] and I'll do my best to help out. Check the FAQ (directly below) first.

FAQ
---

* [Debugging unit tests example][clicker-issue-6]
* [404 on app.bundle.js][clicker-issue-29]
* [run a single test][clicker-issue-35]
* [why do our tests need a main method (they don't anymore - for posterity)][clicker-issue-65]

<div align="center"><iframe src="https://ghbtns.com/github-btn.html?user=lathonez&repo=clicker&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe></div>

[analog-clicker-img]: http://thumbs.dreamstime.com/thumblarge_304/1219960995H0ZkZw.jpg
[angular2-di-testing]:https://developers.livechatinc.com/blog/testing-angular-2-apps-dependency-injection-and-components
[angular2-seed-cfg]:  https://github.com/mgechev/angular2-seed/blob/master/tools/config.ts
[angular2-seed-repo]: https://github.com/mgechev/angular2-seed
[angular2-sg-dir]:    https://github.com/mgechev/angular2-style-guide#directory-structure
[app.html]:           https://github.com/lathonez/clicker/blob/master/app/app.html
[app.spec.ts]:        https://github.com/lathonez/clicker/blob/master/app/app.spec.ts
[blog-issue-new]:     https://github.com/lathonez/lathonez.github.io/issues/new
[blog-repo]:          https://github.com/lathonez/lathonez.github.io
[browserify-home]:    http://browserify.org/
[clicker-codecov]:    https://codecov.io/github/lathonez/clicker?branch=master
[clicker-coveralls]:  https://coveralls.io/github/lathonez/clicker?branch=master
[clicker-issue-6]:    https://github.com/lathonez/clicker/issues/6
[clicker-issue-29]:   https://github.com/lathonez/clicker/issues/29
[clicker-issue-35]:   https://github.com/lathonez/clicker/issues/35
[clicker-issue-38]:   https://github.com/lathonez/clicker/issues/38
[clicker-issue-65]:   https://github.com/lathonez/clicker/issues/65
[clicker-issue-79]:   https://github.com/lathonez/clicker/issues/79
[clicker-issue-99]:   https://github.com/lathonez/clicker/issues/99
[clicker-issue-new]:  https://github.com/lathonez/clicker/issues/new
[clicker-deps]:       https://github.com/lathonez/clicker#dependencies
[clicker-repo]:       http://github.com/lathonez/clicker
[clicker-travis]:     https://travis-ci.org/lathonez/clicker
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
