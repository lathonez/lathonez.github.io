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

Your project lifecycle creates artifacts in your `www/build` folder that you do not want going into your `.apk`.

We needed to delete the output of our test build phase (from `www/build/test`). I came across others doing the same, whilst some had `.git` artefacts to prune.

Which builder am I using?
------------------------

You should see a build exec line when running `cordova build` or `ionic build`. Below is ours, using [gradle][gradle-home].

```
Running command: /home/lathonez/code/clicker/platforms/android/cordova/build
...
Running: /home/lathonez/code/clicker/platforms/android/gradlew cdvBuildDebug -b /home/lathonez/code/clicker/platforms/android/build.gradle -Dorg.gradle.daemon=true
```

Ant
---

If your Cordova build is using ant you should be able to override it by creating an `ant.properties` file in `platforms/android/`, as per [this SO answer][so-ant-props].

```
aapt.ignore.assets=!*.map:!thumbs.db:!.git:.*:*~
```

With this line, any file or directory that matches one of these patterns will not be included in the .apk file:

* `*.map`
* `thumbs.db`
* `.git`
* `.*`
* `*~`

The `!` before the pattern is to prevent ant from spitting a warning out for each file. See the [original article][ant-original] for more info.

Gradle
------

We experimented for some time with [build-extras.gradle][cordova-beg], before stumbling on [this SO answer][so-no-gradle], which explains why it won't work.

As the next best alterntive we went for a `before_compile` [Cordova hook][cordova-hooks]. `cordova build` is shorthand for `cordova prepare` then `cordova compile`. The `cordova prepare` is copying your assets over in prepraration for compilation.

We can therefore make use of the `before_compile` hook to remove these assets from `platform/android/assets` so they don't make their way into the `.apk`, whilst leaving the output in `www/build` intact.

The hook is a one liner into config.xml:

```xml
<hook type="before_compile" src="cordovaHooks/beforeCompile.js" />
```

Node is recommended for the hook scripts to keep your project cross-plaform. Ours looked like this:

```javascript
module.exports = function(ctx) {

    var del    = require('del');
    var config = require('../ionic.config');
    var path   = 'platforms/**/assets' + '/' + [config.paths.test.dest];

    console.log('Removing test build from assets before compilation: del(' + path + ')');
    return del(path);
};
```
When running `cordova build` or `ionic build`, Cordova will invoke your hook script just before it compiles your code, removing those pesky files.

Help!
-----

If you can't get any of this working in your own project, [raise an issue][clicker-issue-new] and I'll do my best to help out.

[ant-original]:        https://coderwall.com/p/ogxpdg/exclude-files-from-cordova-phonegap-build-android
[clicker-issue-15]:    https://github.com/lathonez/clicker/issues/15
[clicker-issue-new]:   https://github.com/lathonez/clicker/issues/new
[cordova-beg]:         https://cordova.apache.org/docs/en/dev/guide/platforms/android/#extending-build-gradle
[cordova-hooks]:       https://cordova.apache.org/docs/en/dev/guide/appdev/hooks/
[gradle-home]:         http://gradle.org/
[so-ant-props]:        http://stackoverflow.com/questions/21142848/cordova-build-ignore-files
[so-no-gradle]:        http://stackoverflow.com/a/25927034/5083721
