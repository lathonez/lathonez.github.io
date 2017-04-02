---
title:  "Unit Testing an Ionic2 project"
date:   2017-04-02 01:48:23
categories: [dev]
tags: [ionic2, angular2, testing]
---

**Deprecation Notice** - I am looking to deprecate this post and associated project in favour of [Ionic's recent example repo][ionic-unit-testing-example]. Please see the [deprecation issue][clicker-issue-239] for the latest status / recommendation. You can still follow this blog and use the clicker project (@2.9.0) as a reference.

**Updated for Ionic 2.1 and Angular 2.4**

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
<code>npm install --save-dev @angular/cli @types/jasmine @types/node jasmine-core jasmine-spec-reporter karma karma-chrome-launcher karma-cli karma-jasmine karma-jasmine-html-reporter karma-coverage-istanbul-reporter</code>
</pre>
</div>

**Temporary step for Ionic 2.1.0**: Upgrade Ionic's packaged version of Angular (if you haven't already). Hopefully they will upgrade soon. You'll get a bunch of warnings at the end of your install about unmet peer dependencies. This is because we've installed newer versions of these packages than Ionic wants.

<div class="highlighter-rouge">
<pre class="lowlight">
<code>npm install --save @angular/common@2.4.0 @angular/compiler@2.4.0 @angular/compiler-cli@2.4.0 @angular/core@2.4.0 @angular/forms@2.4.0 @angular/http@2.4.0 @angular/platform-browser@2.4.0 @angular/platform-browser-dynamic@2.4.0 @angular/platform-server@2.4.0 @angular/router@3.4.0 rxjs@5.1.0 zone.js@0.7.6</code>
</pre>
</div>

Install config files and boilerplate
------------------------------------

Into your project's root:

* [.angular-cli.json][.angular-cli.json]: Angular Cli's config file
* [karma.conf.js][karma.conf.js]: Karma's config file

Into your project's `./src` folder

* [mocks.ts][mocks.ts]: Mocks for Ionic classes we'll need to stub out when testing
* [polyfills.ts][polyfills.ts]: Pollyfills used by Angular Cli
* [test.ts][test.ts]: Main entry point for our unit tests. **Remove references to ClickerServices as they won't be applicable to you**
* [tsconfig.spec.json][tsconfig.spec.json]: Angular Cli's compiler config for spec files

For the lazy:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>for file in .angular-cli.json karma.conf.js
do
  wget https://raw.githubusercontent.com/lathonez/clicker/master/${file}
done

cd src

for file in mocks.ts polyfills.ts test.ts tsconfig.spec.json
do
  wget https://raw.githubusercontent.com/lathonez/clicker/master/src/${file}
done</code>
</pre>
</div>

Modify existing Ionic config files:
-----------------------------------

Exclude test.ts and all our spec files in Ionic's [tsconfig.json][ion.tsconfig.json]:

```yaml
  "exclude": [
    "node_modules",
    "src/test.ts",
    "**/*.spec.ts"
  ],
```

Add the following line to the `scripts` object in your [package.json][package.json] (generating code coverage [breaks sourcemaps][ng-cli-sourcemaps], so we add an explicit option for it):

```javascript
  "test-coverage": "ng test --code-coverage",
  "test": "ng test"
```

test.ts
-------

[This file][test.ts] is worth exploring a little further. We've created a couple of functions to remove a lot of the boilerplate around an Ionic testbed setup, we'll be using these in any of our unit tests that create a Angular 2 components.

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

And this is just a wrapper we'll call from `beforeEach` to compile your components:

```javascript
  public static beforeEachCompiler(components: Array<any>): Promise<{fixture: any, instance: any}> {
    return TestUtils.configureIonicTestingModule(components)
      .compileComponents().then(() => {
        let fixture: any = TestBed.createComponent(components[0]);
        return {
          fixture: fixture,
          instance: fixture.debugElement.componentInstance,
        };
      });
  }
```

This means that instead of needing the above code in each of your component's spec files, you simply need:

```javascript
  beforeEach(async(() => TestUtils.beforeEachCompiler([MyComponent]).then(compiled => {
    fixture = compiled.fixture;
    instance = compiled.instance;
  })));
```

Alternatively, you can write all this inline in your spec files (as opposed to using TestUtils above):

```javascript
  let fixture = null;
  let instance = null;

  beforeEach(async(() => {

    TestBed.configureTestingModule({
      declarations: [MyComponent],
      providers: [
        App, Platform, Form, Keyboard, MenuController, NavController,
        {provide: Config, useClass: ConfigMock},
      ],
      imports: [
        FormsModule,
        IonicModule,
        ReactiveFormsModule,
      ],
    })
    .compileComponents().then(() => {
      fixture = TestBed.createComponent(MyComponent);
      instance = fixture;
      fixture.detectChanges();
    });
  }));
```

Your first unit test
--------------------

Pick one of your components to write a test for and create a `component-name.spec.ts` file for it.

A simple skeleton unit test file looks like this, where `HelloIonic` is whatever component you're testing.

```javascript
import { ComponentFixture, async } from '@angular/core/testing';
import { TestUtils }               from '../../test';
import { HelloIonicPage }          from './hello-ionic';

let fixture: ComponentFixture<HelloIonicPage> = null;
let instance: any = null;

describe('Pages: HelloIonic', () => {

  beforeEach(async(() => TestUtils.beforeEachCompiler([HelloIonicPage]).then(compiled => {
    fixture = compiled.fixture;
    instance = compiled.instance;
  })));

  it('should create the hello ionic page', async(() => {
    expect(instance).toBeTruthy();
  }));
});
```

**You'll also need to add** `./` to the templateUrl path in your component .ts file:

`templateUrl: 'hello-ionic.html'` becomes `templateUrl: './hello-ionic.html'`

Running the tests
-----------------

`npm test`

```
x220:~/code/myApp$ npm test
(node:17800) fs: re-evaluating native module sources is not supported. If you are using the graceful-fs module, please update it to a more recent version.

> ionic-hello-world@ test /home/lathonez/code/myApp
> ng test

30 10 2016 19:12:00.472:WARN [karma]: No captured browser, open http://localhost:9876/
30 10 2016 19:12:00.491:INFO [karma]: Karma v1.3.0 server started at http://localhost:9878/
30 10 2016 19:12:00.491:INFO [launcher]: Launching browser Chrome with unlimited concurrency
30 10 2016 19:12:00.546:INFO [launcher]: Starting browser Chrome
30 10 2016 19:12:01.732:INFO [Chrome 54.0.2840 (Linux 0.0.0)]: Connected on socket /#-CZqilkRyMTyIn1xAAAA with id 84944674

START:
  Pages: HelloIonic
    ✔ should create the hello ionic page

Finished in 0.776 secs / 0.77 secs

SUMMARY:
✔ 1 test completed
```

Congrats! You now have unit testing working in your Ionic 2 project.

Test Coverage
--------------

This set up outputs [lcov][lcov-home] coverage to `./coverage` in the root folder of your app. If you browse to `/path/to/myApp/coverage/lcov-report/` in a web browser, you'll get [an overview][lcov-index-ss] of all your tested files. [Drill into one][lcov-app-ss] and you get line by line info as to what's covered.

You can also use external tools, I highly recommend [codecov][clicker-codecov].

Debugging Setup
---------------

There are a few fiddly steps to get Angular CLI integrated into your Ionic project. If something isn't working for you, you have missed a step above. This setup works cross platform and is running on a closed source project with 400+ unit tests.

Before raising an issue, please follow these basic steps to debug the setup:

* check you are on node LTS
* remove and reinstall node_modules
* clone a copy of clicker and compare the relevant files with your own using a diff tool (meld)

If you have performed these steps and still have no luck, please raise an issue (see below) - note that we will probably need access to your source code to help out.

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

If you can't get any of this working in your own project - follow the Debugging Setup, [raise an issue][clicker-issue-new] and I'll do my best to help out.

If you have a general question about unit testing concepts (e.g. how can I write a unit test for `some-module`), see our [General Testing Help thread][clicker-issue-191].

<div align="center"><iframe src="https://ghbtns.com/github-btn.html?user=lathonez&repo=clicker&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe></div>

[.angular-cli.json]:   https://github.com/lathonez/clicker/blob/4b2a6fa40500c998b96c1b8e6bc281b80eae80c9/.angular-cli.json
[blog-issue-new]:     https://github.com/lathonez/lathonez.github.io/issues/new
[blog-repo]:          https://github.com/lathonez/lathonez.github.io
[clicker-codecov]:    https://codecov.io/github/lathonez/clicker?branch=master
[clicker-issue-38]:   https://github.com/lathonez/clicker/issues/38
[clicker-issue-191]:  https://github.com/lathonez/clicker/issues/191
[clicker-issue-239]:  https://github.com/lathonez/clicker/issues/239
[clicker-issue-new]:  https://github.com/lathonez/clicker/issues/new
[clicker-repo]:       http://github.com/lathonez/clicker
[ion.tsconfig.json]:  https://github.com/lathonez/clicker/blob/4b2a6fa40500c998b96c1b8e6bc281b80eae80c9/tsconfig.json
[ionic-unit-testing-example]: https://github.com/driftyco/ionic-unit-testing-example
[karma-console-ss]:   /images/ionic2_unit_testing/karma-console-screenshot.png
[karma-debug-ss]:     /images/ionic2_unit_testing/karma-debug-screenshot.png
[karma.conf.js]:      https://github.com/lathonez/clicker/blob/4b2a6fa40500c998b96c1b8e6bc281b80eae80c9/karma.conf.js
[lcov-app-ss]:        /images/ionic2_unit_testing/lcov-app-screenshot.png
[lcov-home]:          http://ltp.sourceforge.net/coverage/lcov.php
[lcov-index-ss]:      /images/ionic2_unit_testing/lcov-index-screenshot.png
[mocks.ts]:           https://github.com/lathonez/clicker/blob/4b2a6fa40500c998b96c1b8e6bc281b80eae80c9/src/mocks.ts
[ng-cli-sourcemaps]:  https://github.com/angular/angular-cli/pull/1799
[package.json]:       https://github.com/lathonez/clicker/blob/4b2a6fa40500c998b96c1b8e6bc281b80eae80c9/package.json
[polyfills.ts]:       https://github.com/lathonez/clicker/blob/4b2a6fa40500c998b96c1b8e6bc281b80eae80c9/src/polyfills.ts
[so-ask]:             http://stackoverflow.com/questions/ask
[test.ts]:            https://github.com/lathonez/clicker/blob/4b2a6fa40500c998b96c1b8e6bc281b80eae80c9/src/test.ts
[tsconfig.json]:      https://github.com/lathonez/clicker/blob/4b2a6fa40500c998b96c1b8e6bc281b80eae80c9/tsconfig.json
[tsconfig.spec.json]: https://github.com/lathonez/clicker/blob/4b2a6fa40500c998b96c1b8e6bc281b80eae80c9/src/tsconfig.spec.json
[typings.d.ts]:       https://github.com/lathonez/clicker/blob/4b2a6fa40500c998b96c1b8e6bc281b80eae80c9/src/typings.d.ts
