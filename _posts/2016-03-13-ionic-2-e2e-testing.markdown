---
title:  "End to End testing an Ionic2 project"
date:   2016-03-15 01:00:00
categories: [dev]
tags: [ionic2, angular2, testing]
---

**TL;DR** - I have an Ionic 2 project on github set up with E2E testing, [dive in][clicker-repo], or read on.

The previous post in this series on [Unit Testing][blog-unit-testing] does a bit of intro that I won't repeat here. For the purposes of this post, it'll be useful to have the demo app cloned locally.

`git clone git@github.com:lathonez/clicker.git`

A simple e2e test on app.ts
----------------------------

Note we keep the tests with source code, as per the guidance in the [Angular 2 Style Guide][angular2-sg-dir].

`cp clicker/app/app.e2e.ts myApp/app/app.e2e.ts`

Modify the test cases in [app.e2e.ts][app.e2e.ts] to suit your application, or use the simple example below:

```javascript
describe('MyApp', () => {

  beforeEach(() => {
    browser.get('');
  });

  it('should have a title', () => {
    expect(browser.getTitle()).toEqual('MyApp Title');
  });
});
```

Building the tests
-------------------

Unlike unit tests, E2E tests will be run against our Ionic development server, which builds source code into `app.bundle.js`. All we need to do is compile our E2E tests from Typescript to Javascript, as opposed to building all the source as well.

The [gulp][gulp-home] build task we set up for building the unit tests does this for free, so we're just going to use that. It's just worth noting that the other stuff that task does (building *.ts and *.spec, and copying assets) are surplus to requirements here.

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

This gulpfile defines several tasks which gulp will carry out for us during the test cycle. The ones we care about for E2E are:

1. **test.clean**: remove content from `www/build`, except `www/build/js`, which isn't used for testing
2. **test.lint**: perform static analysis on source code using `tslint` if enabled
3. **test.build.typescript**: compile all our Typescript (both source and test), into individual Javascript files. This is done differently from Ionic which bundles everything into `app.bundle.js`

**Install Dependencies and Typings:**

`karma` is necessary only because we are doing unit tests in the same gulpfile.

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install -g typings
npm install --save-dev chalk del gulp gulp-load-plugins gulp-inline-ng2-template gulp-tap gulp-tslint gulp-typescript karma run-sequence tslint ts-node</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="lowlight">
<code>typings install --ambient --save angular-protractor bluebird chalk del express glob gulp gulp-load-plugins gulp-typescript gulp-util karma jasmine log4js mime minimatch node orchestrator q run-sequence selenium-webdriver serve-static through2 vinyl</code>
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

To get [Protractor][protractor-home] up and runnning, we need more boilerplate config and more dev dependencies.

Copy the following files into your project:

`cp clicker/test/protractor.config.js myApp/test`

* [protractor.config.ts][protractor.config.ts]: Protractor's config

**Install deps:**

`npm install --save-dev protractor selenium-webdriver`

Add the following lines to your `package.json` so we can get everything working nicely with `npm` instead of calling `gulp`:

```yaml
  "scripts": {
    "e2e": "gulp --gulpfile test/gulpfile.ts --cwd ./ test.build && ./node_modules/protractor/bin/protractor test/protractor.conf.js",
    "start": "ionic serve",
    "webdriver-start": "webdriver-manager start",
    "webdriver-update": "webdriver-manager update"
  }
```

Contribute
----------

[Clickers][clicker-repo] is a work in progress. If you'd like to help out or have any suggestions, check the [roadmap sticky][clicker-issue-38].

Help!
-----

If you can't get any of this working in your own project, [raise an issue][clicker-issue-new] and I'll do my best to help out.

[angular2-sg-dir]:      https://github.com/mgechev/angular2-style-guide#directory-structure
[blog-unit-testing]:    http://lathonez.github.io/2016/ionic-2-unit-testing/
[clicker-repo]:         http://github.com/lathonez/clicker
[app.e2e.ts]:           https://github.com/lathonez/clicker/blob/master/app/app.e2e.ts
[protractor-home]:      https://angular.github.io/protractor-home
[protractor.config.js]: https://github.com/lathonez/clicker/blob/master/test/protractor.config.js


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