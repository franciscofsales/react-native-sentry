.. class:: platform-react-native

React Native
============

This is the documentation for our beta clients for React-Native.

.. admonition:: note

   This is an early beta release that only supports iOS.  If you require
   Android support

Installation
------------

Start with adding sentry and linking it::

    $ npm install react-native-sentry --save
    $ react-native link react-native-sentry

Note that we only support ``react-native >= 0.41`` at the moment and you
will have to make sure a recent version of :ref:`sentry-cli <sentry-cli>`
installed.

Updating Build Steps
--------------------

Open up your xcode project in the iOS folder, go to your project's target and
change the "Bundle React Native code and images" build script.  The script that
is currently there needs to be adjusted as follows::

    cp -r ${CONFIGURATION_BUILD_DIR}/Sentry.framework ${CONFIGURATION_BUILD_DIR}/${EXECUTABLE_NAME}.app/Frameworks
    cp -r ${CONFIGURATION_BUILD_DIR}/KSCrash.framework ${CONFIGURATION_BUILD_DIR}/${EXECUTABLE_NAME}.app/Frameworks
    export SENTRY_ORG=YOUR_ORG_SLUG
    export SENTRY_PROJECT=YOUR_PROJECT_SLUG
    export SENTRY_AUTH_TOKEN=YOUR_AUTH_TOKEN
    export NODE_BINARY=node
    sentry-cli react-native-xcode ../node_modules/react-native/packager/react-native-xcode.sh
    sentry-cli upload-dsym

You can find the slugs in the URL of your project (sentry.io/your-org-slug/your-project-slug)
If you don't have an auth token yet you can [create an auth token here](https://sentry.io/api/).

This also uploads debug symbols in the last line which however will not work for
bitcode enabled builds.  If you are using bitcode you need to remove that
line (``sentry-cli upload-dsym``) and consult the documentation on dsym
handling instead (see :ref:`dsym-with-bitcode`).

Client Configuration
--------------------

Add sentry to your `index.ios.js`:

.. sourcecode:: javascript

    import { Sentry } from 'react-native-sentry';

    Sentry.config('___DSN___').install();

Additionally you need to register the native crash handler in your `AppDelegate.m`:

.. sourcecode:: objc

    #import <React/RNSentry.h>

    /* ... */
    [RNSentry installWithRootView:rootView];

Additional Configuration
------------------------

These are functions you can call in your javascript code:

.. sourcecode:: javascript

    import {
      Sentry,
      SentrySeverity,
      SentryLog
    } from 'react-native-sentry';

    // disable stacktrace merging
    Sentry.config("___DSN___", {
      deactivateStacktraceMerging: true
    }).install();

    // change log level
    Sentry.setLogLevel(SentryLog.Debug);

    // export an extra context
    Sentry.setExtraContext({
      "a_thing": 3,
      "some_things": {"green": "red"},
      "foobar": ["a", "b", "c"],
      "react": true,
      "float": 2.43
    });

    // set the tag context
    Sentry.setTagsContext({
      "environment": "production",
      "react": true
    });

    // set the user context
    Sentry.setUserContext({
      email: "john@apple.com",
      userID: "12341",
      username: "username",
      extra: {
        "is_admin": false
      }
    });

    // set a custom message
    Sentry.captureMessage("TEST message", {
      level: SentrySeverity.Warning
    }); // Default SentrySeverity.Error

    // This will trigger a crash in the native sentry client
    //Sentry.nativeCrash();