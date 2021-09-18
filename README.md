## Quick guide to setting up Detox end 2 end testing with React Native




https://user-images.githubusercontent.com/61206305/133856975-dbbc58a8-9db9-4d4b-b76a-f2ad408a3b3a.mp4



### (iOS & Jest as test runner)

I'm using yarn but you can switch it out for `npm` too.

1. Run `yarn global add detox-cli` -- install the Detox CLI globally
2. Run `brew tap wix/brew && brew install applesimutils` -- install `applesimutils` using Homebrew
3. Run `yarn add -D detox jest` -- install Detox and Jest as dev dependencies
4. Sometimes you may need to run `yarn` / `npm i` here again just to make sure everything is up to date.
5. Run `detox init -r jest` -- tell detox to use Jest as the test **r**unner, and scaffold some files
6. Add `npx react-native run-ios` in `/.detoxrc.json` `apps.ios.build` . (You might need to add the `build` property if it doesn't exist)
7. In `/ios` run `pod install` -- install cocoa pods for ios
8. Run `detox build -c ios` -- this will run the command specified in step 7. `-c` is short for `--configuration`
9. Find your `APPNAME.app` binary and paste the path in `/.detoxrc.json` `apps.ios.binaryPath` (see note).
10. Run `detox test -c ios` -- this runs the tests in the `/e2e/` . The file names for the test should be `TESTNAME.e2e.js`
11. You should see the test suite start to run.

Note: If you can't find your app binary or want to set it in a logical place follow these steps:

1. Launch Xcode workspace for the relevant app.
   1. File > Workspace settings...
   2. Click Derived Data and set it to 'Workspace-relative location'
   3. Click Advanced
   4. Click Custom, set the drop down input to 'Relative to Workspace'
   5. Run `npx react-native run-ios` or `detox build -c ios` and it should now output the binary to the `ios` folder in your project. Mine was `ios/Build/Products/Debug-iphonesimulator/APPNAME.app`. Use the path to your app binary in step 10.

---

### Example test:

```js
describe('Example', () => {
  beforeAll(async () => {
    await device.launchApp()
  })

  it('should have hello world text', async () => {
    await expect(element(by.id('text'))).toBeVisible()
  })

  it('should allow typing in text input', async () => {
    await element(by.id('textInput')).typeText('Hello world! This is so cool.')
  })

  it('should show button was tapped after button is pressed', async () => {
    await element(by.id('button')).tap()
    await expect(element(by.text('Button was tapped!'))).toBeVisible()
  })
})
  
});
```

For the following React Native code

```jsx
export default function App() {
  const [text, setText] = useState('')

  return (
    <View style={styles.container}>
    
      <Text testID="text">Hello world!</Text>
      
      <View style={{ height: 30 }} />
      
      <TextInput
        value={text}
        onChangeText={userInputtedText => setText(userInputtedText)}
        testID="textInput"
        style={{ borderWidth: 1, borderColor: 'gray', width: 250, height: 40 }}
      />
      
      <View style={{ height: 30 }} />

      <Button
        title="press me!"
        onPress={() => Alert.alert('Button was tapped!')}
        testID={'button'}
      />
      
    </View>
  )
}
```



---

### The Device object is globally available in the test files and has several methods. It facilitates control of the attached device.

```js
device.launchApp()
device.reloadApp()
device.reloadReactNative()
device.openUrl()
// and more
```

#### For a full list of methods visit: https://github.com/wix/Detox/blob/master/docs/APIRef.DeviceObjectAPI.md

The tests are comprised of 3 parts - the **matcher** + the **action** + the **expectation**.

1. The ***matcher*** 'gets' the element on the screen (similar to `document.getElementbyId()` or `useRef()`),

Examples: 
- `by.id()`
- `by.text()`
- `.by.label()`
- `.withAncestor()`

#### The list of matchers can be found: https://github.com/wix/Detox/blob/master/docs/APIRef.Matchers.md

2. The ***action*** performs an 'action' on the matched element

Examples:
-  `.tap()`
-  `.typeText()`
-  `.replaceText()`
-   `.scroll()`
-   `.longPress()` 
   
#### The list of actions can be found here: https://github.com/wix/Detox/blob/master/docs/APIRef.ActionsOnElement.md

3. The ***expectation*** is your expectation of the matched component's state (the criteria on which the test should pass/fail)
Examples:
- `.toBeVisible()`
- `.toHaveText()`
- `.toExist()`
- `.toBeFocused()`

#### The list of expectations can be found here: https://github.com/wix/Detox/blob/master/docs/APIRef.Expect.md


---
## Detox recorder


<img src="https://github.com/wix/DetoxRecorder/blob/master/Documentation/Resources/Presentation.gif"/>


#### there's an issue open on the repo for the recorder not working with yarn/npm scripts and on Android for now. Hopefully they release a fix soon. For now you'll just have to paste in the full command and use the iOS simulator.

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

### Detox + React Native + Android

Setting up Android is a lot more involved. Follow the guide here: https://github.com/wix/Detox/blob/master/docs/Introduction.Android.md
