---
title: Enzyme Setup
---

To get Enzyme installed, add the following three packages:

```
# yarn add -D enzyme enzyme-adapter-react-16 @jonny/react-native-mock
```

Here's what these do:

- [`enzyme`][enzyme] is the core Enzyme library.
- [`enzyme-adapter-react-16`](http://airbnb.io/enzyme/docs/installation/react-16.html) sets Enzyme up to work with React 16's API. There's a separate adapter for React 15.
- [`@jonny/react-native-mock`](https://github.com/JonnyBurger/react-native-mock/) mocks out React Native to make it easier to test React Native components.

Now that these packages are installed, we need to configure Enzyme to use the adapter we installed. Create a `tests/setup.js` script and add the following:

```javascript
import Enzyme from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';

Enzyme.configure({ adapter: new Adapter() });
```

Then instruct Jest to load this file upon startup by adding the following to `package.json`:

```diff
   "jest": {
-    "preset": "react-native"
+    "preset": "react-native",
+    "setupFiles": [
+      "./tests/setup.js"
+    ]
   },
```

## Smoke Test

To confirm it's working, let's write a simple test of React Native's built-in `Text` component. Create a `tests/components` folder, then add a `smoke.spec.js` file in it with the following contents:

```javascript
import React from 'react';
import { Text } from 'react-native';
import { shallow } from 'enzyme';

describe('Text', () => {
  it('renders text', () => {
    const wrapper = shallow(<Text>Hello, world!</Text>);
    expect(wrapper.text()).toEqual('Hello, world!');
  });
});
```

Run your tests with `yarn test`. You should see the following output:

```
# yarn test
yarn run v1.7.0
$ jest tests/**/*.spec.js
 PASS  tests/components/smoke.spec.js
  Text
    ✓ renders text (167ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        1.367s, estimated 4s
Ran all test suites matching /tests\/components\/smoke.spec.js/i.
✨  Done in 1.98s.
```

[enzyme]: http://airbnb.io/enzyme/