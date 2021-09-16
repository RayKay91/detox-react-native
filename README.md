## Quick guide to setting up Detox end 2 end testing with React Native

#### (iOS & Jest as test runner)

I'm using yarn but you can switch it out for `npm` too.

1. Run `yarn global add detox-cli` -- install the Detox CLI globally
2. Run `brew tap wix/brew && brew install applesimutils` -- install `applesimutils` using Homebrew
3. Run `yarn add -D detox` -- install Detox as a dev dependency
4. Run `yarn add -D jest` -- install Jest as a dev dependency
5. Sometimes you may need to run `yarn` / `npm i` here again just to make sure everything is up to date.
6. Run `detox init -r jest` -- tell detox to use Jest as the test **r**unner, and scaffold some files
7. Add `npx react-native run-ios` in `/.detoxrc.json` `apps.ios.build` . (You might need to add the property if it doesn't exist)
8. In `/ios` run `pod install` -- install cocoa pods for ios
9. Run `detox build -c ios` -- this will run the command specified in step 7. `-c` is short for `--configuration`
10. Find your `APPNAME.app` binary and paste the path in `/.detoxrc.json` `apps.ios.binaryPath`*
11. Run `detox test -c ios` -- this runs the tests in the `/e2e/` . The file names for the test should be `TESTNAME.e2e.js`
12. You should see the test suite start to run.

*If you can't find your app binary or want to set it in a logical place follow the following steps:

1. Launch Xcode workspace for the relevant app.
   1. File > Workspace settings...
   2. Click Derived Data and set it to 'Workspace-relative location'
   3. Click Advanced
   4. Click Custom, set the drop down input to 'Relative to Workspace'
   5. Done.


---

---

The Device object is globally available in the test files and has several methods. It facilitates control of the attached device.

```js
device.launchApp()
device.reloadApp()
device.reloadReactNative()
device.openUrl()
// and more
```

For a full list of methods visit: https://github.com/wix/Detox/blob/master/docs/APIRef.DeviceObjectAPI.md

The tests are comprised of 3 parts - the **matcher** + the **action** + the **expectation**.

1. The ***matcher*** 'gets' the element on the screen (similar to `document.getElementbyId()` or `useRef()`),

Examples: 
- `by.id()`
- `by.text()`
- `.by.label()`
- `.withAncestor()`

- The list of matchers can be found: https://github.com/wix/Detox/blob/master/docs/APIRef.Matchers.md

2. The ***action*** performs an 'action' on the matched element

Examples:
-  `.tap()`
-  `.replaceText()`
-   `.scroll()`
-   `.longPress()` 
   
- The list of actions can be found here: https://github.com/wix/Detox/blob/master/docs/APIRef.ActionsOnElement.md

3. The ***expectation*** is your expectation of what should be happening (the criteria on which the test should pass/fail)
Examples:
- `.toBeVisible()`
- `.toHaveText()`
- `.toExist()`
- `.toBeFocused()`

- The list of expectations can be found here: https://github.com/wix/Detox/blob/master/docs/APIRef.Expect.md


---
## Detox recorder
This will greatly simplify the *match* and *action* part of the test writing. Any expectations should be written by yourself. I recommend checking out the webpage as it contains a few gifs of the recorder working, and explains how the colour system works when tapping buttons: 
 [wix/DetoxRecorder](https://github.com/wix/DetoxRecorder) .
 
 For typing I'd recommend you write your own test as the way React works causes the test to become very lengthy (give it a shot and see what happens if you're curious).
 
 Occasionally Detox recorder will get things wrong, so you may need to double check the matchers if your test is failing. To help the recorder as much as possible you should try and use `testId` prop in your react components. When using the recorder and tapping elements, it will try to match those elements as best it can. You should be aiming for the component to flash green when interacted with as this means the Detox recorder was able to match it well. Yellow/Red flashes should try to be avoided and the matchers may need tweaking manually.

To install:

- run `yarn add -D detox-recorder`
- `detox recorder --bundleId com.YOUR.BUNDLE --simulatorId booted --outputTestFile "~/Desktop/NAMEOFRECORDEDTEST.js" --testName 'NAMEOFTEST' --record`
- this tells Detox to record the app with the specified bundle, use the booted simulator (the one open and running already), create the file with recorded code on the Desktop with the specified name, name the test with the specified name, and to start recording (you can use `-r` to help make the command a little shorter. It's not much but it helps :) )
- You should see the recorder button(s) at the top of the screen. As you record tap things and move/scroll the screen it'll record it and output the code to the file on your desktop.
- use `detox recorder --help` to view the help
- Once recorded and added to your `*.e2e.js` test you can run `detox test` (plus `--configuration ios` or `android`) to start the test! Happy testing :D.


--

Example test:

```js
describe('Login flow', () => {
    
  it('should login successfully', async () => {
    await device.reloadReactNative();
    
    await element(by.id('email')).typeText('john@example.com');
    await element(by.id('password')).typeText('123456');
    await element(by.text('Login')).tap();
      
    await expect(element(by.text('Welcome'))).toBeVisible();
    await expect(element(by.id('email'))).toNotExist();
  });
  
});
```

Setting up Android is a lot more involved. Follow the guide here: https://github.com/wix/Detox/blob/master/docs/Introduction.Android.md