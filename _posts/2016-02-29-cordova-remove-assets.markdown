---
title:  "Excluding files from a Cordova build"
date:   2016-02-28 11:34:23
categories: [dev]
tags: [ionic2, cordova, gradle]
---

Quick post to explain how to remove files from a Cordova (also works for Ionic2) project at build time.

See the [original issue on github][clicker-issue-15].

Why would I need to do this?
-----------------------------

Your project lifecycle creates artifacts in `www/build` that you do not want going into the `.apk`.

In our case we needed to delete the output of our test build phase from `www/build/test`.

Don't touch the builder
------------------------

Originally it seemed the correct approach would be to [override the builder configuration][so-ant-props]. However that brings with it a few problems:

* The builders are platform specific (e.g. if you build Android and iOS you'll need two changes)
* There are different builders for Android - `ant` or `gradle` - depending on which version of Cordova you're running
* Currently the [override does not work][so-no-gradle] in `gradle`

Use the Cordova build hooks
------

We went for an `after_prepare` [Cordova hook][cordova-hooks]. `cordova build` is shorthand for `cordova prepare` then `cordova compile`. `cordova prepare` copies your assets over in prepraration for compilation.

We can therefore make use of the `after_prepare` hook to remove these assets from `platform/**/assets` so they don't make their way into the `.apk`, whilst leaving the output in `www/build` intact.

If you add your hook into `./hooks/after_prepare`, no changes to `config.xml` are required - [more info][cordova-hooks].

Node is recommended for the hooks to keep your project cross-plaform. Here's what we came up with, inspired by an existing hook in Ionic's [app base][ionic-ab-hook]

```javascript
#!/usr/bin/env node

var del  = require('del');
var fs   = require('fs');
var path = require('path');

var rootdir = process.argv[2];

if (rootdir) {

  // go through each of the platform directories that have been prepared
  var platforms = (process.env.CORDOVA_PLATFORMS ? process.env.CORDOVA_PLATFORMS.split(',') : []);

  for(var x=0; x<platforms.length; x++) {
    // open up the index.html file at the www root
    try {
      var platform = platforms[x].trim().toLowerCase();
      var testBuildPath;

      if(platform == 'android') {
        testBuildPath = path.join('platforms', platform, 'assets', 'www', 'build', 'test');
      } else {
        testBuildPath = path.join('platforms', platform, 'www', 'build', 'test');
      }

      if(fs.existsSync(testBuildPath)) {
        console.log('Removing test build from assets after prepare: ' + testBuildPath);
        del.sync(testBuildPath);
      } else {
        console.log('Test build @ ' + testBuildPath + ' does not exist for removal');
      }

    } catch(e) {
      process.stdout.write(e);
    }
  }
}

```
When running `cordova build` or `ionic build`, Cordova will invoke your hook after it has prepared your code for compilation, removing those pesky files.

Contribute
----------

[Clickers][clicker-repo] is a work in progress. If you'd like to help out or have any suggestions, check the [roadmap sticky][clicker-issue-38].

This blog is [on github][blog-repo], if you can improve it, have any suggestions or I've failed to keep it up to date, [raise an issue][blog-issue-new] or a PR.

Help!
-----

If you can't get any of this working in your own project, [raise an issue][clicker-issue-new] and I'll do my best to help out.

[ant-original]:        https://coderwall.com/p/ogxpdg/exclude-files-from-cordova-phonegap-build-android
[blog-issue-new]:      https://github.com/lathonez/lathonez.github.io/issues/new
[blog-repo]:           https://github.com/lathonez/lathonez.github.io
[clicker-issue-15]:    https://github.com/lathonez/clicker/issues/15
[clicker-issue-191]:    https://github.com/lathonez/clicker/issues/191
[clicker-issue-38]:    https://github.com/lathonez/clicker/issues/38
[clicker-issue-new]:   https://github.com/lathonez/clicker/issues/new
[clicker-repo]:        http://github.com/lathonez/clicker
[cordova-beg]:         https://cordova.apache.org/docs/en/dev/guide/platforms/android/#extending-build-gradle
[cordova-hooks]:       https://cordova.apache.org/docs/en/dev/guide/appdev/hooks/
[gradle-home]:         http://gradle.org/
[so-ant-props]:        http://stackoverflow.com/questions/21142848/cordova-build-ignore-files
[so-no-gradle]:        http://stackoverflow.com/a/25927034/5083721
[ionic-ab-hook]:      https://github.com/driftyco/ionic2-app-base/blob/master/hooks/after_prepare/010_add_platform_class.js
