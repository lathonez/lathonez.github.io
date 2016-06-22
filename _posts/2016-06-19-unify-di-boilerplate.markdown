---
title:  "Unify DI Boilerplate"
date:   2016-06-18 20:34:23
categories: [dev]
tags: [angular2, testing, di, test]
---

When setting up dependency injection for [my demo project][clicker-repo] I follwed this [excellent blog post][angular2-di-testing].

This post demonstrates how to unify boilerplate necessary for dependency injection away from your test specs into a single file.

What boilerplate?
-----------------

Below is a very simple test spec with dependency injection component we're testing. As you can see, it's 99% boilerplate, lots of imports along with the `beforeEach` and `beforeEachProvider` functions.

```javascript
import {
  beforeEach,
  beforeEachProviders,
  describe,
  expect,
  injectAsync,
  it,
}                               from '@angular/core/testing';
import {
  ComponentFixture,
  TestComponentBuilder,
}                               from '@angular/compiler/testing';
import { provide }              from '@angular/core';
import { Page2 }                from './page2';
import { Utils }                from '../../services/utils';
import { ConfigMock }           from './mocks';
import {
  Config,
  Form,
  App,
  NavController,
  NavParams,
  Platform,
}                               from 'ionic-angular';

let page2: Page2 = null;
let page2Fixture: ComponentFixture = null;

describe('Page2', () => {

  beforeEachProviders(() => [
    Form,
    provide(NavController, {useClass: MockClass}),
    provide(NavParams, {useClass: MockClass}),
    provide(Config, {useClass: MockClass}),
    provide(App, {useClass: MockClass}),
    provide(Platform, {useClass: MockClass}),
  ]);

  beforeEach(injectAsync([TestComponentBuilder], (tcb: TestComponentBuilder) => {
    return tcb
      .createAsync(Page2)
      .then((componentFixture: ComponentFixture) => {
        page2Fixture = componentFixture;
        page2 = componentFixture.componentInstance;
        page2Fixture.detectChanges();
      })
      .catch(Utils.promiseCatchHandler);
  }));

  it('initialises', () => {
    expect(page2).not.toBeNull();
    expect(page2Fixture).not.toBeNull();
  });
});
```

For a few files this is fine. In a larger project with tens of spec files, maintaining this boilerplate became a headache for us;  Angular 2 and Ionic 2 were evolving and the imports and providers kept changing.

It's also not great having to read ~50 lines of boilerplate at the top of each file before you can see what's being tested.

Unifying the boilerplate
-------------------------

We set out to remove as many of the imports as we could, as well as outsourcing the `beforeEach` functions into a single file.

Here's the what we ended up with:

```javascript
import { provide, Type }                              from '@angular/core';
import { ComponentFixture, TestComponentBuilder }     from '@angular/compiler/testing';
import { injectAsync }                                from '@angular/core/testing';
import { Control }                                    from '@angular/common';
import { App, Config, Form, NavController, Platform } from 'ionic-angular';
import { ClickersMock, ConfigMock, NavMock }          from './mocks';
import { Utils }                                      from '../app/services/utils';
import { Clickers }                                   from '../app/services/clickers';
export { TestUtils }                                  from './testUtils';

export let providers: Array<any> = [
  Form,
  provide(Config, {useClass: ConfigMock}),
  provide(Clickers, {useClass: ClickersMock}), // required by ClickerButton
  provide(App, {useClass: ConfigMock}),        // required by ClickerList
  provide(NavController, {useClass: NavMock}), // required by ClickerList
  provide(Platform, {useClass: ConfigMock}),   // -> IonicApp
];

export let injectAsyncWrapper: Function = ((callback) => injectAsync([TestComponentBuilder], callback));

export let asyncCallbackFactory: Function = ((component, testSpec, detectChanges, beforeEachFn) => {
  return ((tcb: TestComponentBuilder) => {
    return tcb.createAsync(component)
      .then((fixture: ComponentFixture<Type>) => {
        testSpec.fixture = fixture;
        testSpec.instance = fixture.componentInstance;
        testSpec.instance.control = new Control('');
        if (detectChanges) fixture.detectChanges();
        if (beforeEachFn) beforeEachFn(testSpec);
      })
      .catch(Utils.promiseCatchHandler);
  });
});
```

`injectAsyncWrapper` wraps `injectAsync`, executing the callback passed in when TestComponentBuilder has completed. The main win from this is that you don't need to import `TestComponentBuilder` in each of your specs.

The heavy lifting is done in `asyncCallbackFactory`, which takes the following arguments:

* **component:** The coponent class we're currently testing
* **testSpec:** A reference to the test spec. This is used so we can set the `component` (instance) and `fixture` variables in the test spet without needing further boilerplate.
* **detectChanges:** Should we invoke `detectChanges()` against the component fixture in `beforeEach`? This is useful in cases when you need to do initial / bespoke setup on the component instance before detecting changes. Usually this would be `true`.
* **beforeEachFn:** Sometimes you need to run an additional function before each test, nothing to do with DI, your standard beforeEach. Before this change you'd have this code in-lined inside createAsync callback.

[This diff][clicker-button-diff] is a good example of the above arguments in use after the boilerplate has been removed.

Resultant Spec
--------------

As we import from our unified `diExports` file, most of the boilerplate has been removed:

```javascript
import { beforeEach, beforeEachProviders, describe, expect, it }          from '@angular/core/testing';
import { asyncCallbackFactory, injectAsyncWrapper, providers, TestUtils } from '../../../test/diExports';
import { Page2 }                                                          from './page2';

this.fixture = null;
this.instance = null;

describe('Page2', () => {

  beforeEachProviders(() => providers);
  beforeEach(injectAsyncWrapper(asyncCallbackFactory(Page2, this, true)));

  it('initialises', () => {
    expect(this.instance).not.toBeNull();
    expect(this.fixture).not.toBeNull();
  });
});
```

You can see the full commit of the above change [here][boilerplate-ci].

Contribute
----------

[Clickers][clicker-repo] is a work in progress. If you'd like to help out or have any suggestions, check the [roadmap sticky][clicker-issue-38].

This blog is [on github][blog-repo], if you can improve it, have any suggestions or I've failed to keep it up to date, [raise an issue][blog-issue-new] or a PR.

Help!
-----

If you can't get any of this working in your own project, [raise an issue][clicker-issue-new] and I'll do my best to help out.

<div align="center"><iframe src="https://ghbtns.com/github-btn.html?user=lathonez&repo=clicker&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe></div>

[angular2-di-testing]: https://developers.livechatinc.com/blog/testing-angular-2-apps-dependency-injection-and-components
[blog-issue-new]:      https://github.com/lathonez/lathonez.github.io/issues/new
[blog-repo]:           https://github.com/lathonez/lathonez.github.io
[boilerplate-ci]:      https://github.com/lathonez/clicker/commit/4769372596756ec60e876de07e484a01ad181de6
[clicker-button-diff]: https://github.com/lathonez/clicker/commit/4769372596756ec60e876de07e484a01ad181de6#diff-81c66575523ad4932b8772dceb8eab4c
[clicker-repo]:        http://github.com/lathonez/clicker

