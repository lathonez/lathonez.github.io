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
<code>npm install --save-dev @types/jasmine @types/node angular-cli codelyzer jasmine-core karma karma-chrome-launcher karma-cli karma-jasmine karma-mocha-reporter karma-remap-istanbul</code>
</pre>
</div>

Install config files and boilerplate
------------------------------------

Into your project's root:

* [angular-cli.json][angular-cli.json]: Angular Cli's config file
* [karma.conf.js][karma.conf.js]: Karma's config file

Into your project's `./src` folder

* [mocks.ts][mocks.ts]: Mocks for Ionic classes we'll need to stub out when testing
* [polyfills.ts][polyfills.ts]: Pollyfills used by Angular Cli
* [test.ts][test.ts]: Main entry point for our unit tests. **Remove references to ClickerServices as they won't be applicable to you**
* [tsconfig.test.json][tsconfig.test.json]: Angular Cli's compiler config
* [typings.d.ts][typings.d.ts]: Angular Cli's typings file (simply declaring System)

For the lazy:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>for file in angular-cli.json karma.conf.js
do
  wget https://raw.githubusercontent.com/lathonez/clicker/master/${file}
done

cd src

for file in mocks.ts polyfills.ts test.ts tsconfig.test.json typings.d.ts
do
  wget https://raw.githubusercontent.com/lathonez/clicker/master/src/${file}
done</code>
</pre>
</div>

Modify existing Ionic config files:
-----------------------------------

Add jasmine typings to `compilerOptions` Ionic's [tsconfig.json][ion.tsconfig.json]:

```yaml
  "types": [
    "jasmine"
  ]
```

Add the following line to the `scripts` object in your [package.json][package.json]:

```javascript
  "test": "ng test --code-coverage"
```

test.ts
-------

This file is worth exploring a little futher. We've created a couple of functions to remove a lot of the boilerplate around an Ionic testbed setup, we'll be using these in any of our unit tests that create a Angular 2 components.

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

If you have a more general question about unit testing concepts (e.g. how can I write a unit test for `some-module`), you'll have a much quicker (and better quality) response posting on [Stack Overflow][so-ask].

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
[ion.tsconfig.json]:  https://github.com/lathonez/clicker/blob/master/tsconfig.json
[karma-console-ss]:   /images/ionic2_unit_testing/karma-console-screenshot.png
[karma-debug-ss]:     /images/ionic2_unit_testing/karma-debug-screenshot.png
[karma.conf.js]:      https://github.com/lathonez/clicker/blob/master/karma.conf.js
[lcov-app-ss]:        /images/ionic2_unit_testing/lcov-app-screenshot.png
[lcov-home]:          http://ltp.sourceforge.net/coverage/lcov.php
[lcov-index-ss]:      /images/ionic2_unit_testing/lcov-index-screenshot.png
[mocks.ts]:           https://github.com/lathonez/clicker/blob/master/src/mocks.ts
[package.json]:       https://github.com/lathonez/clicker/blob/master/package.json
[polyfills.ts]:       https://github.com/lathonez/clicker/blob/master/src/polyfills.ts
[so-ask]:             http://stackoverflow.com/questions/ask
[test.ts]:            https://github.com/lathonez/clicker/blob/master/src/test.ts
[tsconfig.json]:      https://github.com/lathonez/clicker/blob/master/tsconfig.json
[tsconfig.test.json]: https://github.com/lathonez/clicker/blob/master/src/tsconfig.test.json
[typings.d.ts]:       https://github.com/lathonez/clicker/blob/master/src/typings.d.ts
