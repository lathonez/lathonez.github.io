---
title:  "Unit Testing an Ionic2 project"
date:   2016-02-21 11:34:23
categories: [dev]
tags: [ionic2, angular2, testing]
---

**TL;DR** - I have an Ionic 2 project on github set up with unit testing, [dive in ][clicker-repo], or read on.

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

There is a lot of discussion around where to keep unit tests. Should they live with the source code or in a separate directory? I decided to keep the two separate. My source lives in app/ and my test lives in test/.


```
app/
├── app.html
├── app.ts
├── components
│   ├── clickerButton
│   │   ├── clickerButton.html
│   │   └── clickerButton.ts
│   └── clickerForm
│       ├── clickerForm.html
│       └── clickerForm.ts
├── models
│   ├── clicker.ts
│   └── click.ts
├── pages
│   ├── clickerList
│   │   ├── clickerList.html
│   │   ├── clickerList.scss
│   │   └── clickerList.ts
│   └── page2
│       ├── page2.html
│       ├── page2.scss
│       └── page2.ts
└── services
    ├── clickers.ts
    └── utils.ts

test/
├── app.spec.ts
├── app.stub.ts
├── components
│   ├── clickerButton
│   │   └── clickerButton.spec.ts
│   └── clickerForm
│       └── clcikerForm.spec.ts
├── karma.config.js
├── models
│   ├── clicker.spec.ts
│   └── click.spec.ts
├── pages
│   └── clickerList
│       └── clickerList.spec.ts
├── services
│   ├── clickers.spec.ts
│   └── utils.spec.ts
├── test-main.js
└── testUtils.ts
```

This may not be the correct decision for your project / team. If you decide to keep the source and tests in the same folder, you should still be able to follow this post with a couple of configuration changes.

A simple unit test on app.ts
----------------------------

We probably haven't got much logic in app.ts worthy of unit testing. However, app.ts is the root source file, if we include it in our test suite, all other spec that we write will be included too.

`cp clicker/test/app.spec.ts myApp/test/app.spec.ts`

Modify the test cases in [app.spec.ts][app.spec.ts] to suit your application, or use the simple example below:

```javascript
import { TEST_BROWSER_PLATFORM_PROVIDERS, TEST_BROWSER_APPLICATION_PROVIDERS} from 'angular2/platform/testing/browser';
import { setBaseTestProviders } from 'angular2/testing';
import { IonicApp, Platform }   from 'ionic-framework/ionic';
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

Copy the following files into your project:

`cp clicker/gulpfile.js clicker/ionic.config.js clicker/tslint.json myApp/`

* [gulpfile.js][gulpfile.js]: gulp’s config file
* [ionic.config.js][ionic.config.js]: ionic config - you should have one of these in your project already. We’re just adding the test paths to it.
* [tslint.json][tslint.json]: config file for static code analysis tool [tslint][tslint-home]

This gulpfile defines several tasks which gulp will carry out for us during the test cycle:

1. **test.clean**: nuke anything that exists already in www/build/test
2. **test.lint**: perform static analysis on our code using tslint. If you don’t want to use linting, just remove the references the gulpfile and delete tslint.json.
3. **test.copyHTML**: we’ll want to have a copy of our html available to our unit tests, but our test build is wholly separate from Ionic’s webpack build. Thus we need to copy the html over into our test build directory
4. **test.compile**: compile all our Typescript (both source and test), into Javascript.
5. **test**: spin up [Karma][karma-home] and run the test (more on this later)

**Install deps:**

* Gulp and [tsd][tsd-home] are global: `npm install -g gulp tsd`
* Dev dependencies: `npm install --save-dev del gulp-typescript gulp-tslint karma tslint tsd`
* Typings for Jasmine: `tsd install jasmine --save`

You're now ready to compile the tests with `gulp test.compile`, you should see the following output

```
[23:40:08] Using gulpfile ~/code/myApp/gulpfile.js
[23:40:09] Starting 'test.clean'...
[23:40:09] Finished 'test.clean' after 13 ms
[23:40:09] Starting 'test.compile'...
[23:40:11] TypeScript: emit succeeded
[23:40:11] Finished 'test.compile' after 2.4 s
```

To verify that the compilation has succeed as planned, inspect www/build/test. You should see that app.spec.ts test has been compiled into Javascript, along with all the source code:


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

Coming soon!

[analog-clicker-img]: http://thumbs.dreamstime.com/thumblarge_304/1219960995H0ZkZw.jpg
[angular2-seed-repo]: https://github.com/mgechev/angular2-seed
[app.spec.ts]:        https://github.com/lathonez/clicker/blob/master/test/app.spec.ts
[clicker-repo]:       http://github.com/lathonez/clicker
[gulp-home]:          http://gulpjs.com/
[gulpfile.js]:        https://github.com/lathonez/clicker/blob/master/gulpfile.js
[ionic.config.js]:    https://github.com/lathonez/clicker/blob/master/ionic.config.js
[karma-home]:         https://karma-runner.github.io/0.13/index.html
[sbtp-docs]:          https://angular.io/docs/js/latest/api/testing/setBaseTestProviders-function.html
[tsd-home]:           https://www.npmjs.com/package/tsd
[tslint-home]:        https://www.npmjs.com/package/tslint
[tslint.json]:        https://github.com/lathonez/clicker/blob/master/tslint.json