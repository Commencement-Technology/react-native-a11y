
# React Native A11y

This is a React Native A11y Library with following main features

🤖 Reader features: Focus, Order, Reader
⌨️ Keyboard features: Focus
🙌 Others features soon

A11y is important, there are a lot of reasons to support and be compliant with it. First of all, it helps people with disabilities work and use your application easily and live a better life. Banks, medication, shops, and delivery is a small list of what people are usually interested in, and it can be more important for people with limitations.

There are can be other reasons, customer requirements, laws and requirements for specific groups of apps, 
remote control, etc. Based on this you can find a lot of advantages and benefits to supporting A11y.

**_NOTE:_**   
0.67.* - checked only, old architecture.
Unfortunately, the new architecture is not supported yet, we plan to do it asap. Not all applications can switch architecture to the new one, there are a lot of problems with migration, usually with external libs, this library was developed on 0.67.* and it was decided to start with it. We plan to update the library to support 0.68.* - 0.70.* and also check the old versions for working.
---
## Installation
This library is not finished yer and currently on alfa stage. We will be glad to issues, questions, and help.

1. Download package with npm or yarn

```
npm i react-native-a11y
```
```
yarn add react-native-a11y
```

2. Install pods
cd ios && pod install

3. Android only

Add to the  `MainActivity.java` lines:

```
  //android/app/src/main/java/com/project-name/MainActivity.java

  ...
  import android.content.Intent;
  import android.content.res.Configuration;
  ...
  
  @Override
  public void onConfigurationChanged(Configuration newConfig) {
    super.onConfigurationChanged(newConfig);
    Intent intent = new Intent("onConfigurationChanged");
    intent.putExtra("newConfig", newConfig);
    this.sendBroadcast(intent);
  }

```

4. Add provider to root of your app:

```
watch: examples/A11ySample/App.tsx

export const App = () => {
  return (
    <A11yProvider>
        // content here
    </A11yProvider>
  );
};
```

## Usage

A11y library consists of different components and hooks, to start work with `react-native-a11y` you can get familiar with an example app in `examples/A11ySample`.

### A11yModule

The core of the library is `A11yModule`, `A11yModule` provides additional functions to work with a11y such as order, reader focus, keyboard focus, announcements, etc


| Function      | Description   | Interface |
| ------------- | ------------- | --------- |
| `isA11yReaderEnabled` | return promise with status of a11y reader (TalkBack or VoiceOver)  true(enabled)/false(disabled) |  `() => Promise<boolean>` |
| `isKeyboardConnected` | return promise with status of keyboard connection, true(connected)/false(disconnected) | `() => Promise<boolean>` |
| `a11yStatusListener` | listener for a11y reader status | `((e: { status: boolean }) => void) => void;` |
| `keyboardStatusListener` | listener for keyboard connection status | `((e: { status: boolean }) => void) => void;` |
| `announceForAccessibility` | Post a string to be announced by the screen reader. Android default, ios specific. | `(announcement: string) => void;` |
| `announceScreenChange` | Announces new screen name. | `(announcement: string) => void;` |
| `setA11yFocus` | Set a11y reader focus to the component | `(ref: React.RefObject<React.Component>) => void;` |
| `setKeyboardFocus` | Set keyboard focus to the component | `(ref: React.RefObject<React.Component>) => void;` |
| `setPreferredKeyboardFocus` | `iOS` only, set redirection of keyboard from one component to another one | `(nativeTag: number, nextTag: number) => void;` |
| `focusFirstInteractiveElement` | Focus first interactive element on a screen | `(ref: React.RefObject<React.Component>) => void;` |
| `setA11yElementsOrder` | Set a11y reader focus order | `setA11yElementsOrder: <T>(info: { tag?: RefObject<T>; views: T[]; }) => void;` |

`setA11yFocus` and `setKeyboardFocus` works similar to ` AccessibilityInfo.setAccessibilityFocus`, difference is they request refs instead of tags and you don't need use `findNodeHandle`.


### Examples
#### setA11yFocus
```
watch: examples/A11ySample/src/screens/ReaderFocusScreen

import { A11yModule } from "react-native-a11y";
...

const App = () => {
  const ref1 = useRef(null);
  const ref3 = useRef(null);
...

return (
 ...
 <Button
    ref={ref1}
    onPress={() => A11yModule.setA11yFocus(ref3)}
    title="1. Set focus to third"
 />
 ...
  <Button
    ref={ref3}
    onPress={() => A11yModule.setA11yFocus(ref2)}
    title="3. Set focus to second"
  />
}
```

#### setKeyboardFocus
```
watch: examples/A11ySample/src/screens/KeyboardFocusScreen

import { A11yModule, KeyboardProvider } from "react-native-a11y";
...

const App = () => {
  const ref1 = useRef(null);
  const ref3 = useRef(null);
...

return (
 ...
 <Button
    ref={ref1}
    onPress={() => A11yModule.setKeyboardFocus(ref3)}
    title="1. Set focus to third"
 />
 ...
  <Button
    ref={ref3}
    onPress={() => A11yModule.setKeyboardFocus(ref2)}
    title="3. Set focus to second"
  />
}
```

You can but not really need to use `A11yModule.setA11yElementsOrder` directly, we have specific useful hooks to work with order. 

### useFocusOrder and useDynamicFocusOrder
To set an order for components we have `useFocusOrder`, `useDynamicFocusOrder`

`A11yModule.setA11yElementsOrder` is a more direct one we just pass refs to components and set order, but there are a lot of questions about when to call and set order. To make a better experience two similar hooks were created `useFocusOrder` and `useDynamicFocusOrder`

#### useDynamicFocusOrder
`useDynamicFocusOrder` returns target ref, trigger function, and function to register your components.

```
export type UseDynamicFocusOrder = () => {
  a11yOrder: { 
    ref: RefObject<View>; /// target ref, we need a target ref to a container View
    onLayout: () => void; // trigger, we use onLayout to realize when components are appear on a screen, useEffect and useLayoutEffect don't work properly
  };
  registerOrder: (order: number) => (ref: View) => void; // function to set order
  reset: () => void; // clear function
};
```

```
soon
```

#### useFocusOrder
`useFocusOrder` is based on `useDynamicFocusOrder` but more static and predictable.

```
(size: number) // count of refs
    => FocusOrderInfo<T> = {
  a11yOrder: { 
    ref: RefObject<T>; // target ref, we need a target ref to a container View
    onLayout: () => void; // trigger, we use onLayout to realize when components are appear on a screen, useEffect and useLayoutEffect don't work properly
  };
  refs: ((ref: T | null) => void)[]; // array of callback refs to use for order 
  reset: () => void; // clear function
};

```

```
watch: examples/A11ySample/src/screens/A11yOrderScreen
 
import { A11yOrder, useFocusOrder } from "react-native-a11y";

const App = () => {
    const { a11yOrder, refs } = useFocusOrder(3); // 3 number of wanted refs
    ...
    return (
        <A11yOrder onLayout={onLayoutHandler} a11yOrder=  {a11yOrder}>
            <Text style={styles.font} ref={refs[0]}>
                First
            </Text>
            <Text style={styles.font} ref={refs[2]}>
                Third
            </Text>
            <Text style={styles.font} ref={refs[1]}>
                Second
            </Text>
        </A11yOrder>
    )
```
The code in example set a new order for components, instead of a direct one it will follow 1 -> 3 -> 2

You also can find a new `A11yOrder` component it's just shorts for `<View {...a11yOrder} />` 

## Roadmap 
- Add more examples, update Readme
- Add tests
- Migrate to the new architecture
- Check React Navigation for A11y and make examples 

## Problems

#### iOS
- remove halo effect for ios keyboard focus, focusEffect = nil doesn't work, overriding to. Note: tested on iphone 8 - 15.5, try to a new one

#### Android
- a11y listener for talk back doesn't work perfect, we can not listen TalkBack only, it listen the whole a11y functional in android and can return wrong results. 

## Contributing

Soon

## Acknowledgements
I really appreciate the work and solutions provided by [Andrii Koval](https://github.com/ZioVio), [Michail Chavkin](https://github.com/mchavkin), [Dzmitry Khamitsevich](https://github.com/bulletxenus). I think there was not this library without them, I also want to thank [Aliaksei Kisel](https://github.com/ziginsider) and [Herman Tseranevich](https://github.com/lollegend) for help with publishing and reviewing.

## License

MIT

---

Made with [create-react-native-library](https://github.com/callstack/react-native-builder-bob)
