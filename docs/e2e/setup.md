---
title: Setting Up Detox
---

# Setting Up Detox

## Installing Detox

First, let's install the global Detox CLI tool:

```bash
$ brew tap wix/brew
$ brew install --HEAD applesimutils
$ yarn global add detox-cli
```

Next, we need to add Detox as a dependency to our project. Detox can use either Jest or Mocha as a test runner, but we'll use Mocha since we've used it for our other types of test.

```bash
$ yarn add --dev detox
```

If your app is an Expo app, a few other packages are also needed for Detox to work with it:

```bash
$ yarn add --dev detox-expo-helpers \
                 expo-detox-hook
```

Now, initialize Detox in your app to get some config files set up. We specify that we'll be using Jest as the test runner.

```bash
$ detox init -r jest
```

After this, we need to add some extra config for Detox to our `package.json`. If using Expo, use the following:

```diff
 {
   ...
   "detox": {
-    "test-runner": "jest"
+    "test-runner": "jest",
+    "configurations": {
+      "ios.sim": {
+        "binaryPath": "bin/Exponent.app",
+        "type": "ios.simulator",
+        "name": "iPhone 8"
+      }
+    }
   }
 }
```

If using React Native CLI, use the following, replacing `YourAppName` with the name of the app you entered:

```diff
 {
   ...
   "detox": {
-    "test-runner": "jest"
+    "test-runner": "jest",
+    "configurations": {
+      "ios.sim.debug": {
+        "binaryPath": "ios/build/Build/Products/Debug-iphonesimulator/YourAppName.app",
+        "build": "xcodebuild -project ios/YourAppName.xcodeproj -scheme YourAppName -configuration Debug -sdk iphonesimulator -derivedDataPath ios/build",
+        "type": "ios.simulator",
+        "name": "iPhone 8"
+      }
+    }
   }
 }
```

Finally, if you're using Expo, you need to download a built version of Expo that Detox can use to hook into. Go to [the Expo Tools page](https://expo.io/tools#client) and click the "Download IPA" link. Expand the downloaded archive, then change the name of the folder to "Exponent.app". Create a `bin` folder in your project and move "Exponent.app" into it.

## Configuring ESLint

If you're using ESLint (and you probably should be!), here are steps to set it up to recognize Detox code.

```bash
$ yarn add --dev eslint-plugin-detox
```

Then add the detox plugin and environment to your ESLint config:

```diff
   parser: 'babel-eslint',
   plugins: [
+    'detox',
     'jest',
   ],
   env: {
     'browser': true,
+    'detox/detox': true,
     'es6': true,
     'jest/globals': true,
     'node': true,
   },
```

## Smoke Test

Detox installs a sample test for you that you can tweak. If you are installing Detox into a brand-new React Native app, you can make a passing test doing the following.

First, add a `testID` prop to an element in `App.js` so Detox can find it:

```diff
   render() {
     return (
       <View style={styles.container}>
-        <Text style={styles.welcome}>
+        <Text style={styles.welcome} testID="welcome">
           Welcome to React Native!
         </Text>
```

Then open the `firstTest.spec.js` that `detox init` generated. Replace the contents with the following:

```javascript
describe('App', () => {
  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should show the welcome message', async () => {
    await expect(element(by.id('welcome'))).toBeVisible();
  });
});
```

If you're using Expo, you'll need to make a slight tweak to the file to get it to load properly:

```diff
+ const { reloadApp } = require('detox-expo-helpers');
+
 describe("App", () => {
   beforeEach(async () => {
-    await device.reloadReactNative();
+    await reloadApp();
   });
```

To run this test, start the Metro packager as usual:

```bash
$ react-native start
```

In another terminal window, run the detox tests with:

```bash
$ detox test
```

You should see the following output:

```bash
detox[87254] INFO:  [DetoxServer.js] server listening on localhost:63646...
detox[87254] INFO:  [AppleSimUtils.js] org.reactjs.native.example.CLIJestTest launched. The stdout and stderr logs were recreated, you can watch them with:
        tail -F /Users/josh/Library/Developer/CoreSimulator/Devices/638CF558-A2B2-4C27-9C8D-7DB5E348E5D8/data/tmp/detox.last_launch_app_log.{out,err}
 PASS  e2e/firstTest.spec.js (13.513s)
  App

    ✓ should show the welcome message (930ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        13.646s, estimated 55s
Ran all test suites matching /e2e/i with tests matching "^((?!:android:).)*$".
```