---
title:  "End to End testing an Ionic2 project"
date:   2016-04-29 02:48:23
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

We need to transpile our E2E tests from Typescript to Javascript, but we don't need to build all the source as `ionic serve` will do this for us later.

Copy gulp's [task definition file][gulpfile.ts] into your project:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>mkdir -p myApp/test
cp clicker/test/gulpfile.ts myApp/test</code>
</pre>
</div>

This gulpfile defines several tasks for use during the test cycle. The only one we care about for E2E is `build-e2e`

**Install Dependencies and Typings:**

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install -g typings
npm install --save-dev del gulp gulp-typescript jasmine-spec-reporter protractor run-sequence ts-node
typings install --global --save registry:dt/angular-protractor registry:dt/jasmine registry:dt/node registry:dt/selenium-webdriver</code>
</pre>
</div>

You're now ready to build the tests:

`node_modules/gulp/bin/gulp.js --gulpfile test/gulpfile.ts --cwd ./ build-e2e`

```
[09:27:47] Requiring external module ts-node/register
[09:27:50] Using gulpfile ~/code/myApp/test/gulpfile.ts
[09:27:50] Starting 'clean-test'...
Deleted -
[09:27:50] Finished 'clean-test' after 5.43 ms
[09:27:50] Starting 'build-e2e'...
[09:27:51] Finished 'build-e2e' after 1.79 s
```

Running the tests
------------------

We'll be running our E2E tests using [Protractor][protractor-home]. Copy [protractor's config][protractor.conf.js] into your project:

`cp clicker/test/protractor.conf.js myApp/test`

Add the following lines to your `package.json` so we can get everything working nicely with `npm`:

```yaml
  "scripts": {
    "e2e": "gulp --gulpfile test/gulpfile.ts --cwd ./ build-e2e && protractor test/protractor.conf.js",
    "start": "ionic serve",
    "webdriver-update": "webdriver-manager update"
  }
```

Run the E2E tests:

* `npm run webdriver-update` - Update webdriver, **only necessary one time after install**
* `npm start` - Start Ionic's dev server
* `npm run e2e` - Build the tests and start protractor (in another terminal)

```
[09:31:41] Requiring external module ts-node/register
[09:31:44] Using gulpfile ~/code/myApp/test/gulpfile.ts
[09:31:44] Starting 'clean-test'...
Deleted /home/lathonez/code/myApp/www/build/test
[09:31:44] Finished 'clean-test' after 15 ms
[09:31:44] Starting 'build-e2e'...
[09:31:46] Finished 'build-e2e' after 1.76 s
[09:31:46] I/direct - Using ChromeDriver directly...
[09:31:46] I/launcher - Running 1 instances of WebDriver
Spec started
Started

  MyApp
    ✓ should have a title
.
Executed 1 of 1 spec SUCCESS in 1 sec.

1 spec, 0 failures
Finished in 1.385 seconds
[09:31:49] I/launcher - 0 instance(s) of WebDriver still running
[09:31:49] I/launcher - chrome #01 passed
```

Using Docker?
----------

Testing on docker (on any other headless environment), requires the use of `xvfb`, a virtual frame buffer implementing the X11 protocol.

`apt-get install xvfb`

Change the `e2e` line in `package.json` to include `xvfb-run`, ensuring protractor has access to the virtual display:

```yaml
  "scripts": {
    "e2e": "gulp --gulpfile test/gulpfile.ts --cwd ./ build-e2e && xvfb-run protractor test/protractor.conf.js",
    ...
  }
```

For more information see [issue #144](clicker-issue-114)

Contribute
----------
[Clickers][clicker-repo] is a work in progress. If you'd like to help out or have any suggestions, check the [roadmap sticky][clicker-issue-38].

This blog is [on github][blog-repo], if you can improve it, have any suggestions or I've failed to keep it up to date, [raise an issue][blog-issue-new] or a PR.

Help!
-----

If you can't get any of this working in your own project, [raise an issue][clicker-issue-new] and I'll do my best to help out.

<div align="center"><iframe src="https://ghbtns.com/github-btn.html?user=lathonez&repo=clicker&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe></div>

[angular2-sg-dir]:      https://github.com/mgechev/angular2-style-guide#directory-structure
[app.e2e.ts]:           https://github.com/lathonez/clicker/blob/master/app/app.e2e.ts
[blog-issue-new]:       https://github.com/lathonez/lathonez.github.io/issues/new
[blog-repo]:            https://github.com/lathonez/lathonez.github.io
[blog-unit-testing]:    http://lathonez.github.io/2016/ionic-2-unit-testing/
[clicker-issue-38]:     https://github.com/lathonez/clicker/issues/38
[clicker-issue-114]:    https://github.com/lathonez/clicker/issues/114
[clicker-issue-new]:    https://github.com/lathonez/clicker/issues/new
[clicker-repo]:         http://github.com/lathonez/clicker
[config.ts]:            https://github.com/lathonez/clicker/blob/master/test/config.ts
[gulp-home]:            http://gulpjs.com/
[gulpfile.ts]:          https://github.com/lathonez/clicker/blob/master/test/gulpfile.ts
[protractor-home]:      https://angular.github.io/protractor
[protractor.conf.js]:   https://github.com/lathonez/clicker/blob/master/test/protractor.conf.js
