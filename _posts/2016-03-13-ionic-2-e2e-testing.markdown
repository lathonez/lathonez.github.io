---
title:  "End to End testing an Ionic2 project"
date:   2016-11-04 02:48:23
categories: [dev]
tags: [ionic2, angular2, testing]
---

**TL;DR** - I have an Ionic 2 project on github set up for E2E testing with [protractor][protractor-home], [dive in][clicker-repo], or read on.

The previous post in this series on [Unit Testing][blog-unit-testing] does a bit of intro that I won't repeat here. For the purposes of this post, it'll be useful to have the demo app cloned locally.

Install dev dependencies
------------------------

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install --save-dev angular-cli jasmine-spec-reporter protractor ts-node</code>
</pre>
</div>

Install config files and boilerplate
------------------------------------

Into your project's root:

* [angular-cli.json][angular-cli.json]: Angular Cli's config file
* [protractor.conf.js][protractor.conf.js]: Protractor's config file


Into a newly created `./e2e` folder in your project's root:

* [tsconfig.json][tsconfig.json]: Angular Cli's compiler config

For the lazy:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>for file in angular-cli.json protractor.conf.js
do
  wget https://raw.githubusercontent.com/lathonez/clicker/master/${file}
done

mkdir e2e
cd e2e

for file in tsconfig.json
do
  wget https://raw.githubusercontent.com/lathonez/clicker/master/e2e/${file}
done</code>
</pre>
</div>


Modify existing Ionic config files:
-----------------------------------

Add your new e2e folder into the exclude array in Ionic's [tsconfig.json][ion.tsconfig.json]:

```yaml
  "exclude": [
    "node_modules"
  ],
```

Add the following to your [gitignore][gitignore] file:

```
# e2e
/e2e/*.js
/e2e/*.map
```

Hook ng e2e into your [package.json][package.json] scripts array:


```yaml
  "scripts": {
    "e2e": "protractor",
    "postinstall": "webdriver-manager update"
  },
```

A simple e2e test on app.ts
---------------------------

Create a simple e2e test file `./e2e/app.e2e-spec.ts` to get us going:

```javascript
import { browser, element, by } from 'protractor';

describe('MyApp', () => {

  beforeEach(() => {
    browser.get('');
  });

  it('should have a title', () => {
    expect(browser.getTitle()).toEqual('MyApp');
  });
})
```

Update the webdriver
--------------------

As we hooked `webdriver-manager update` into our `npm postinstall` script, we just need to run `npm install`. Thus this step is covered by simply installing your project (in the future):

```
x220:~/code/myApp$ npm install

...

> ionic-hello-world@ postinstall /home/lathonez/code/myApp
> webdriver-manager update

...

[16:38:03] I/update - chromedriver: setting permissions to 0755 for /home/lathonez/code/myApp/node_modules/protractor/node_modules/webdriver-manager/selenium/chromedriver_2.25
```

Running the tests
-----------------

As we hooked into our [package.json][package.json] above, we can run the tests with a simple `npm run e2e`. Don't forget to start the server first with `ionic serve`:

```
x220:~/code/myApp$ npm run e2e
(node:31496) fs: re-evaluating native module sources is not supported. If you are using the graceful-fs module, please update it to a more recent version.

> ionic-hello-world@ e2e /home/lathonez/code/myApp
> protractor

[16:40:33] I/direct - Using ChromeDriver directly...
[16:40:33] I/launcher - Running 1 instances of WebDriver
Started
Spec started
.
  MyApp
    ✓ should have a title

1 spec, 0 failures
Finished in 1.631 seconds

Executed 1 of 1 spec SUCCESS in 2 secs.
[16:40:36] I/launcher - 0 instance(s) of WebDriver still running
[16:40:36] I/launcher - chrome #01 passed
```

Using Docker?
----------

Testing on docker (or any other headless environment), requires the use of `xvfb`, a virtual frame buffer implementing the X11 protocol.

`apt-get install xvfb`

Change the `e2e` line in `package.json` to include `xvfb-run`, ensuring protractor has access to the virtual display:

```yaml
  "scripts": {
    "e2e": "xvfb-run protractor",
    ...
  }
```

For more information see [issue #144][clicker-issue-114]

Contribute
----------
[Clickers][clicker-repo] is a work in progress. If you'd like to help out or have any suggestions, check the [roadmap sticky][clicker-issue-38].

This blog is [on github][blog-repo], if you can improve it, have any suggestions or I've failed to keep it up to date, [raise an issue][blog-issue-new] or a PR.

Help!
-----

If you can't get any of this working in your own project, [raise an issue][clicker-issue-new] and I'll do my best to help out.

<div align="center"><iframe src="https://ghbtns.com/github-btn.html?user=lathonez&repo=clicker&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe></div>

[angular-cli.json]:     https://github.com/lathonez/clicker/blob/master/angular-cli.json
[blog-issue-new]:       https://github.com/lathonez/lathonez.github.io/issues/new
[blog-repo]:            https://github.com/lathonez/lathonez.github.io
[blog-unit-testing]:    http://lathonez.github.io/2016/ionic-2-unit-testing/
[clicker-issue-114]:    https://github.com/lathonez/clicker/issues/114
[clicker-issue-38]:     https://github.com/lathonez/clicker/issues/38
[clicker-issue-new]:    https://github.com/lathonez/clicker/issues/new
[clicker-repo]:         http://github.com/lathonez/clicker
[gitignore]:            https://github.com/lathonez/clicker/blob/master/.gitignore
[ion.tsconfig.json]:    https://github.com/lathonez/clicker/blob/master/tsconfig.json
[package.json]:         https://github.com/lathonez/clicker/blob/master/package.json
[protractor-home]:      https://angular.github.io/protractor
[protractor.conf.js]:   https://github.com/lathonez/clicker/blob/master/protractor.conf.js
[tsconfig.json]:        https://github.com/lathonez/clicker/blob/master/e2e/tsconfig.json
