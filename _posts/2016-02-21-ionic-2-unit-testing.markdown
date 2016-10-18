---
title:  "Unit Testing an Ionic2 project"
date:   2016-10-15 01:48:23
categories: [dev]
tags: [ionic2, angular2, testing]
---

**Updated for Ionic 2 rc1 and Angular 2 Final!**

**TL;DR** - I have an Ionic 2 project on github set up with unit testing, [dive in][clicker-repo], or read on.

This blog and associated project have been around since the early days of Ionic 2, however there still isn't concrete guidance out there on how to unit test, or even a recommendend setup.

In `rc0`, Ionic ditched their current build process `gulp` and sidestepped onto `rollup`. This was frustrating for many in the community, who were hoping for a move toward `ng-cli` and `webpack`.

Rather than re-write this setup for `rollup`, I decided to get to get as close to `ng-cli` as possible, mainly so we could have a stable reference point for our Ionic 2 testing framework.

This post explains the setup and how you can incorporate it into your own project without too much pain.

Install dev dependencies
------------------------

Install the following npm dev dependencies, or simply merge our [package.json][package.json] with your own.

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install --save-dev angular-cli codelyzer jasmine-core jasmine-spec-reporter karma karma-chrome-launcher karma-cli karma-jasmine karma-mocha-reporter karma-remap-istanbul</code>
</pre>
</div>

Add the following line to the `scripts` object in your [package.json][package.json]:

```javascript
  "test": "ng test"
```

Install config files and boilerplate
------------------------------------

Into your project's root:

* [angular-cli.json][angular-cli.json]: Angular Cli's config file
* [karma.conf.js][karma.conf.js]: Karma's config file

Into your project's `./src` folder

* [mocks.ts][mocks.ts]: Mocks for Ionic classes we'll need to stub out when testing
* [pollyfills.ts][pollyfills.ts]: Pollyfills used by Angular Cli
* [test.ts][test.ts]: Main entry point for our unit tests
* [typings.d.ts][typings.d.ts]: Angular Cli's typings file simply declaring System

Add the following lines to your [tsconfig.json][tsconfig.json]:

```javascript
    "typeRoots": [
      "node_modules/@types",
      "../node_modules/@types"
    ]
```

[Why we're doing this ^][double-typing]

test.ts
-------

This file is worth exploring a little futher. I've created a function to remove a lot of the boilerplate around an Ionic testbed setup, we'll be using it in any of our unit tests that create an Angular 2 component.

The following function `configureIonicTestingModule` takes one or more of your components and sets up an Ionic test bed for them:

```javascript
  public static configureIonicTestingModule(components: Array<any>): void {
    TestBed.configureTestingModule({
      declarations: [
        ...components,
      ],
      providers: [
        {provide: App, useClass: ConfigMock},
        {provide: Config, useClass: ConfigMock},
        Form,
        {provide: Keyboard, useClass: ConfigMock},
        {provide: MenuController, useClass: ConfigMock},
        {provide: NavController, useClass: NavMock},
        {provide: Platform, useClass: PlatformMock},
        {provide: ClickersService, useClass: ClickersServiceMock},
      ],
      imports: [
        FormsModule,
        IonicModule,
        ReactiveFormsModule,
      ],
    });
  }
```

This means that instead of needing the above code in each of your spec files, you simply need:

```javascript
TestUtils.configureIonicTestingModule([ClickerForm]);
```

Your first unit test
--------------------

Pick one of your components to write a test for and create a `component-name.spec.ts` file for it.

A simple skeleton unit test file looks like this, where `Page2` is whatever component you're testing.

```javascript
import { ComponentFixture, TestBed, async } from '@angular/core/testing';
import { TestUtils }                        from '../../test';
import { Page2 }                            from './page2';

let fixture: ComponentFixture<Page2> = null;
let instance: any = null;

describe('Pages: Page2', () => {

  beforeEach(() => {
    TestUtils.configureIonicTestingModule([Page2]);
    fixture = TestBed.createComponent(Page2);
    instance = fixture.debugElement.componentInstance;
  });

  it('should create page2', async(() => {
    expect(instance).toBeTruthy();
  }));
});
```

Running the tests
-----------------

`npm test`

```
x220:~/code/clicker$ npm test
(node:11596) fs: re-evaluating native module sources is not supported. If you are using the graceful-fs module, please update it to a more recent version.

> Clicker@2.0.0 test /home/lathonez/code/clicker
> ng test

Could not start watchman; falling back to NodeWatcher for file system events.
Visit http://ember-cli.com/user-guide/#watchman for more info.
18 10 2016 19:52:32.336:WARN [karma]: No captured browser, open http://localhost:9876/
18 10 2016 19:52:32.348:INFO [karma]: Karma v1.2.0 server started at http://localhost:9876/
18 10 2016 19:52:32.349:INFO [launcher]: Launching browser Chrome with unlimited concurrency
18 10 2016 19:52:32.355:INFO [launcher]: Starting browser Chrome
18 10 2016 19:52:33.363:INFO [Chrome 53.0.2785 (Linux 0.0.0)]: Connected on socket /#6bqnoHH631bVTN-pAAAA with id 98134208

START:
Chrome 53.0.2785 (Linux 0.0.0): Executed 0 of 1 SUCCESS (0 secs / 0 secs)
  Pages: Page2
Chrome 53.0.2785 (Linux 0.0.0): Executed 1 of 1 SUCCESS (0.102 secs / 0.102 secs)

Finished in 0.102 secs / 0.102 secs

SUMMARY:
✔ 1 test completed
```

Congrats! You now have unit testing working in your Ionic 2 project.

Test Coverage
--------------

*TODO*: this is currently broken with the upgrade to `rc0`, will be fixing it.

At the end of the `npm test` output you'll see a coverage report table (as above). This gives a good overview, but if you're trying to figure out why your code isn't covered you'll need more.

This set up outputs [lcov][lcov-home] coverage to `./coverage` in the root folder of your app. If you browse to `/path/to/myApp/coverage/lcov-report/` in a web browser, you'll get [an overview][lcov-index-ss] of all your tested files. [Drill into one][lcov-app-ss] and you get line by line info as to what's covered.

You can also use external tools, I highlighy recommend [codecov][clicker-codecov].

Debugging the Tests
--------------------

Sometimes it's useful to debug our tests in the Chrome console.

Hit the [Debug][karma-debug-ss] button and another tab will open. Open the dev console and you can see the [output of all your tests][karma-console-ss], along with any errors which can be debugged as per usual.

Contribute
----------

[Clicker][clicker-repo] is a work in progress. If you'd like to help out or have any suggestions, check the [roadmap sticky][clicker-issue-38].

This blog is [on github][blog-repo], if you can improve it, have any suggestions or I've failed to keep it up to date, [raise an issue][blog-issue-new] or a PR.

Help!
-----

If you can't get any of this working in your own project, [raise an issue][clicker-issue-new] and I'll do my best to help out. Check the FAQ (directly below) first.

FAQ
---

* [run a single test][clicker-issue-35]
* [running as root][clicker-issue-111]

<div align="center"><iframe src="https://ghbtns.com/github-btn.html?user=lathonez&repo=clicker&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe></div>

[angular-cli.json]:   https://github.com/lathonez/clicker/blob/master/angular-cli.json
[blog-issue-new]:     https://github.com/lathonez/lathonez.github.io/issues/new
[blog-repo]:          https://github.com/lathonez/lathonez.github.io
[clicker-codecov]:    https://codecov.io/github/lathonez/clicker?branch=master
[clicker-issue-111]:  https://github.com/lathonez/clicker/issues/111
[clicker-issue-35]:   https://github.com/lathonez/clicker/issues/35
[clicker-issue-38]:   https://github.com/lathonez/clicker/issues/38
[clicker-issue-new]:  https://github.com/lathonez/clicker/issues/new
[clicker-repo]:       http://github.com/lathonez/clicker
[double-typing]:      https://github.com/lathonez/clicker/commit/246c28df59542ba0b3b03047a5c6e163c9844ee2
[karma-console-ss]:   /images/ionic2_unit_testing/karma-console-screenshot.png
[karma-debug-ss]:     /images/ionic2_unit_testing/karma-debug-screenshot.png
[karma.conf.js]:      https://github.com/lathonez/clicker/blob/master/karma.conf.js
[lcov-app-ss]:        /images/ionic2_unit_testing/lcov-app-screenshot.png
[lcov-home]:          http://ltp.sourceforge.net/coverage/lcov.php
[lcov-index-ss]:      /images/ionic2_unit_testing/lcov-index-screenshot.png
[mocks.ts]:           https://github.com/lathonez/clicker/blob/master/src/mocks.ts
[package.json]:       https://github.com/lathonez/clicker/blob/master/package.json
[pollyfills.ts]:      https://github.com/lathonez/clicker/blob/master/src/pollyfills.ts
[test.ts]:            https://github.com/lathonez/clicker/blob/master/src/test.ts
[tsconfig.json]:      https://github.com/lathonez/clicker/blob/master/tsconfig.json
[typings.d.ts]:       https://github.com/lathonez/clicker/blob/master/src/typings.d.ts

