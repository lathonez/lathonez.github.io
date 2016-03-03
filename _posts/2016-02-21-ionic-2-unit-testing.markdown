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

There is some debate around where to keep unit tests. I started off with the unit tests in a separate `test/` directory, but have recently combined with the source code as per the [Angular 2 Style Guide][angular2-sg-dir]


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

test/
├── app.stub.ts
├── config.ts
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
cp clicker/config.ts myApp/test
ln -s app myApp/build</code>
</pre>
</div>

* [gulpfile.ts][gulpfile.ts]: gulp’s config file
* [config.ts][config.ts]: small config file for this setup, containing only paths at the moment. Can be [heavily expanded] down the line.
* [build symlink][build.sym]: this seems to be the cleanest way to use Ionic's build structure with [gulp-inline-ng2-template][gulp-inline-ng2]

This gulpfile defines several tasks which gulp will carry out for us during the test cycle:

1. **test.clean**: nuke anything that exists already in www/build/test
2. **test.lint**: perform static analysis on source code using tslint, if enabled (not by default if following this post)
3. **test.copyHTML**: we’ll want to have a copy of our html available to our unit tests, but our test build is wholly separate from Ionic’s webpack build. Thus we need to copy the html over into our test build directory
4. **test.compile**: compile all our Typescript (both source and test), into Javascript.
5. **test**: spin up [Karma][karma-home] and run the test (more on this later)

**Install Dependencies and Typings:**

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install --save-dev chalk del gulp gulp-load-plugins gulp-inline-ng2-template gulp-tslint gulp-typescript karma run-sequence tslint ts-node typings</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="lowlight">
<code>for typing in \
bluebird chalk del es6-shim express glob gulp gulp-load-plugins gulp-typescript \
gulp-util jasmine karma log4js mime minimatch node orchestrator q \
run-sequence serve-static through2 vinyl
do
./node_modules/typings/dist/bin/typings.js install $typing --save-dev --ambient --no-insight
done</code>
</pre>
</div>

You're now ready to compile the tests: `node_modules/gulp/bin/gulp.js test.compile test.compile`

```
[23:40:08] Using gulpfile ~/code/myApp/gulpfile.ts
[23:40:09] Starting 'test.clean'...
[23:40:09] Finished 'test.clean' after 13 ms
[23:40:09] Starting 'test.compile'...
[23:40:11] TypeScript: emit succeeded
[23:40:11] Finished 'test.compile' after 2.4 s
```

To verify that the compilation has succeed as planned, inspect `www/build/test`. You should see that `app.spec.ts` has been compiled into Javascript, along with all the source code:


```
www/build/test/
├── app.js
├── app.spec.js
└── pages
    ├── page1
    │   └── page1.js
    ├── page2
    │   └── page2.js
    ├── page3
    │   └── page3.js
    └── tabs
        └── tabs.js
```

Running the tests
------------------

To get [Karma][karma-home] up and runnning, we need more boilerplate config and more dev dependencies.

Copy the following files into your project:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>cp clicker/test/app.stub.ts clicker/test/karma.config.js clicker/test/test-main.js myApp/test</code>
</pre>
</div>

* [app.stub.ts][app.stub.ts]: A stub for Ionic's @App decorator.
* [karma.config.js][karma.config.js]: Karma's config
* [test-main.js][test-main.js]: Main entry point for [unit test excution using RequireJS][karma-tm-docs].

**Install deps:**

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install --save-dev es6-module-loader jasmine-core karma-coverage karma-jasmine karma-mocha-reporter karma-phantomjs-launcher phantomjs-prebuilt systemjs traceur</code>
</pre>
</div>

Now we're ready to test: `gulp test`

```
[22:53:59] Requiring external module ts-node/register
[22:54:01] Using gulpfile ~/code/clicker/gulpfile.ts
[22:54:01] Starting 'test'...
[22:54:01] Starting 'test.clean'...
[22:54:01] Starting 'test.lint'...
[22:54:01] Deleted /home/lathonez/code/clicker/www/build/test
[22:54:01] Finished 'test.clean' after 452 ms
[22:54:02] Finished 'test.lint' after 1.16 s
[22:54:02] Starting 'test.copyHTML'...
[22:54:02] Finished 'test.copyHTML' after 12 ms
[22:54:02] Starting 'test.build'...
[22:54:04] TypeScript: emit succeeded
[22:54:04] Finished 'test.build' after 1.88 s
[22:54:04] Starting 'startKarma'...

START:
[23:33:26] Finished 'test' after 800 ms
24 02 2016 23:33:26.416:INFO [karma]: Karma v0.13.21 server started at http://localhost:9876/
24 02 2016 23:33:26.422:INFO [launcher]: Starting browser PhantomJS
24 02 2016 23:33:26.647:INFO [PhantomJS 2.1.1 (Linux 0.0.0)]: Connected on socket /#uuQjIqLkOQKcVuk2AAAA with id 99323503
  MyApp
    ✔ initialises with two possible pages

Finished in 0.028 secs / 0.015 secs

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

karma exited with 0
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
[build.sym]:          https://github.com/lathonez/clicker/blob/master/build
[clicker-codecov]:    https://codecov.io/github/lathonez/clicker?branch=master
[clicker-coveralls]:  https://coveralls.io/github/lathonez/clicker?branch=master
[clicker-issue-new]:  https://github.com/lathonez/clicker/issues/new
[clicker-issue-6]:    https://github.com/lathonez/clicker/issues/6
[clicker-issue-20]:   https://github.com/lathonez/clicker/issues/20
[clicker-issue-22]:   https://github.com/lathonez/clicker/issues/22
[clicker-issue-29]:   https://github.com/lathonez/clicker/issues/29
[clicker-repo]:       http://github.com/lathonez/clicker
[clicker-travis]:     https://travis-ci.org/lathonez/clicker
[cordova-prune-post]: http://lathonez.github.io/2016/cordova-remove-assets/
[gulp-home]:          http://gulpjs.com/
[gulp-inline-ng2]:    https://github.com/ludohenin/gulp-inline-ng2-template
[gulpfile.ts]:        https://github.com/lathonez/clicker/blob/master/gulpfile.ts
[config.ts]:          https://github.com/lathonez/clicker/blob/master/test/config.ts
[karma-home]:         https://karma-runner.github.io/0.13/index.html
[karma.config.js]:    https://github.com/lathonez/clicker/blob/master/test/karma.config.js
[karma-tm-docs]:      https://karma-runner.github.io/0.8/plus/RequireJS.html
[lcov-app-ss]:        /images/ionic2_unit_testing/lcov-app-screenshot.png
[lcov-home]:          http://ltp.sourceforge.net/coverage/lcov.php
[lcov-index-ss]:      /images/ionic2_unit_testing/lcov-index-screenshot.png
[sbtp-docs]:          https://angular.io/docs/js/latest/api/testing/setBaseTestProviders-function.html
[strict-typing]:      https://github.com/lathonez/clicker/blob/master/tslint.json#L80-L97
[test-main.js]:       https://github.com/lathonez/clicker/blob/master/test/test-main.js
[tslint-home]:        https://www.npmjs.com/package/tslint
[tslint.json]:        https://github.com/lathonez/clicker/blob/master/tslint.json
[typings-home]:       https://www.npmjs.com/package/typings