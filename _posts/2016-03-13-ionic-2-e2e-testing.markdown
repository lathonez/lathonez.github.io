---
title:  "End to End testing an Ionic2 project"
date:   2016-03-12 01:00:00
categories: [dev]
tags: [ionic2, angular2, testing]
---

**TL;DR** - I have an Ionic 2 project on github set up for E2E testing with [protractor][protractor-home], [dive in][clicker-repo], or read on.

The previous post in this series on [Unit Testing][blog-unit-testing] does a bit of intro that I won't repeat here. For the purposes of this post, it'll be useful to have the demo app cloned locally.

`git clone git@github.com:lathonez/clicker.git`

A simple e2e test on app.ts
----------------------------

Note we keep the tests and source code together, as per the [Angular 2 Style Guide][angular2-sg-dir].

`cp clicker/app/app.e2e.ts myApp/app/app.e2e.ts`

Modify the test cases in [app.e2e.ts][app.e2e.ts] to suit your application, or use the simple example below (works with the ionic starter app):

```javascript
describe('MyApp', () => {

  beforeEach(() => {
    browser.get('');
  });

  it('should have a title', () => {
    expect(browser.getTitle()).toEqual('Tab 1');
  });
});
```

Building the tests
-------------------

Unlike the unit tests, E2E tests are run against the Ionic development server. All we need to do is compile our E2E tests from Typescript to Javascript, as opposed to building all the source as well.

Make the following changes to your project:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>mkdir -p myApp/test
cp clicker/test/gulpfile.ts myApp/test
cp clicker/test/config.ts myApp/test</code>
</pre>
</div>

* [gulpfile.ts][gulpfile.ts]: gulp’s task definition file
* [config.ts][config.ts]: config file for this setup

This gulpfile defines several tasks which gulp will carry out for us during the test cycle. The only one we care about for E2E is `test.build.e2e`

**Install Dependencies and Typings:**

`karma` is necessary only because we are doing unit tests in the same gulpfile.

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install -g typings
npm install --save-dev chalk del gulp gulp-load-plugins gulp-inline-ng2-template gulp-tslint gulp-typescript karma run-sequence tslint ts-node</code>
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

We'll be running our E2E tests using [Protractor][protractor-home]. To get it up and runnning, we need more boilerplate config and more dev dependencies.

Copy [protractor's config][protractor.conf.js] into your project:

`cp clicker/test/protractor.conf.js myApp/test`

Install deps:

`npm install --save-dev jasmine-spec-reporter protractor`

Add the following lines to your `package.json` so we can get everything working nicely with `npm`:

```yaml
  "scripts": {
    "e2e": "gulp --gulpfile test/gulpfile.ts --cwd ./ test.build && ./node_modules/protractor/bin/protractor test/protractor.conf.js",
    "start": "ionic serve",
    "webdriver-update": "webdriver-manager update"
  }
```

Run the E2E tests:

* `npm run webdriver-update` - Update webdriver, **only necessary one time after install**
* `npm start` - Start Ionic's dev server
* `npm run e2e` - Build the tests and start protractor (in another terminal)

```
Using ChromeDriver directly...
[launcher] Running 1 instances of WebDriver
Spec started
Started

  MyApp
    ✓ should have a title
.
Executed 1 of 1 spec SUCCESS in 4 secs.

1 spec, 0 failures
Finished in 3.714 seconds
[launcher] 0 instance(s) of WebDriver still running
[launcher] chrome #1 passed
```

Contribute
----------

[Clickers][clicker-repo] is a work in progress. If you'd like to help out or have any suggestions, check the [roadmap sticky][clicker-issue-38].

Help!
-----

If you can't get any of this working in your own project, [raise an issue][clicker-issue-new] and I'll do my best to help out.

[angular2-sg-dir]:      https://github.com/mgechev/angular2-style-guide#directory-structure
[app.e2e.ts]:           https://github.com/lathonez/clicker/blob/master/app/app.e2e.ts
[blog-unit-testing]:    http://lathonez.github.io/2016/ionic-2-unit-testing/
[clicker-issue-38]:     https://github.com/lathonez/clicker/issues/38
[clicker-issue-new]:    https://github.com/lathonez/clicker/issues/new
[clicker-repo]:         http://github.com/lathonez/clicker
[config.ts]:            https://github.com/lathonez/clicker/blob/master/test/config.ts
[gulp-home]:            http://gulpjs.com/
[gulpfile.ts]:          https://github.com/lathonez/clicker/blob/master/test/gulpfile.ts
[protractor-home]:      https://angular.github.io/protractor
[protractor.conf.js]:   https://github.com/lathonez/clicker/blob/master/test/protractor.conf.js
