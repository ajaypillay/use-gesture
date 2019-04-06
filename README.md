<p align="middle">
  <a href="https://codesandbox.io/s/n9vo1my91p"><img src="https://i.imgur.com/tg1mN1F.gif" width="655"/></a>
</p>
<p align="middle">
  <a href="https://codesandbox.io/s/j0y0vpz59"><img src="https://i.imgur.com/OxGLHeT.gif" width="515"/></a>
  <a href="https://codesandbox.io/s/r5qmj8m6lq"><img src="https://i.imgur.com/ifdCBvG.gif" width="130"/></a>
</p>
<p align="middle">
  <i>These demos are real, click them!</i>
</p>

```
npm install react-use-gesture
```

Ever thought about doing that sidebar pull-out, a view pager, some slider, any gesture on the web basically, and dropped the idea because it's too hard? In that case, this is your lib.

React-use-gesture is a React hook that lets you bind richer mouse and touch events to any component or view. With the data you receive, it becomes trivial to set up gestures, and often takes no more than a few lines of code.

You can use it stand-alone, but to make the most of it you should combine it with an animation library like [react-spring](https://github.com/react-spring/react-spring), though you can most certainly use any other.

### Api

```js
import useGesture from 'react-use-gesture'
```

The api is straight forward. You bind handlers to your view, specify the actions you want to respond to (drag, hover, move, scroll or wheel) and you will receive events when you interact with the component. These events include the source dom event, but also carry additional kinematics such as velocity, distance, delta, etc.

Hooks allow gestures to be re-used for more than one view (you can use the same `bind()` function multiple times!).

```jsx
// Rough example that makes a div respond to drag and scroll gestures
function myComponent() {
  const bind = useGesture({
    onDrag: dragState => doStuffOnDrag,
    onScroll: scrollState => doStuffOnScroll
  })
  return <div {...bind(optionalArgs)} />
}
```

### React-use-gesture only adds listeners, nothing more!

Contrary to libraries such as [react-pose](https://popmotion.io/pose/) that provide out of the box draggable components, binding a drag handler to a `div` through react-use-gesture doesn’t make it draggable. For that, you will need to pass it styling elements based on the values returned by `useGesture`.

#### Making things move

<p align="middle">
  <img src="https://i.imgur.com/ooNu3jz.gif" width="200"/>
</p>

```jsx
function myComponent() {
  const [[x, y], set] = React.useState([0, 0])
  const bind = useGesture({ onDrag: ({ local }) => set(local) })
  return <div {...bind()} style={{ transform: `translate3d(${x}px,${y}px,0)` }} />
}
```

When the user drags the `div` that receives the `bind()` prop, `useGesture` updates the state of the component and the `div` gets positioned accordingly.

In this case we fetch `local` off the gesture event, which keeps track of delta positions after release. Deltas are especially important in this lib, because they make it possible to use transitions for positioning, instead of doing complex `getBoundingClientRect()` calculations to figure out where a node went on the screen.

#### Avoid re-rendering (preferred)

In the example we’ve just seen, the component gets re-rendered every time `useGesture` drag handler fires, which can be taxing. To avoid re-rendering you may want to use libraries such as [react-spring](https://github.com/react-spring/react-spring) that allow to animate dom elements without setting state and therefore without triggering new renders.

```jsx
import { useSpring, animated } from 'react-spring'

function myComponent() {
  const [{ local }, set] = useSpring(() => ({ local: [0, 0] }))
  const bind = useGesture({ onDrag: ({ local }) => set({ local }) })

  return <animated.div {...bind()} style={{ transform: local.interpolate((x, y) => `translate3d(${x}px,${y}px,0)`) }} />
}
```

Because we’re now using `animated.div`, we’re able to make the element draggable without provoking new renders every time its position should update.

### Supported gestures

In addition to **drag**, react-use-gesture also supports **scroll** gestures, and mouse-specific gestures such as **move**, **hover** (entering and leaving an element), and **wheel**. Every gesture has a handler that you can pass to `useGesture`, and you can even pass multiple handlers for the element to respond to different gestures.

```jsx
const bind = useGesture({
  onDrag: state => {...},     // fires on drag
  onScroll: state => {...},   // fires on scroll
  onHover: state => {...},    // fires on mouse enter, mouse leave
  onMove: state => {...},     // fires on mouse move over the element
  onWheel: state => {...}     // fires on mouse wheel over the eleement
})
```

#### `on[Gesture]Start` and `on[Gesture]End`

Drag, move, scroll and wheel gestures also have two additional handlers that let you perform actions when they start or end. For example, `onScrollEnd` fires when the user finished scrolling.

**Note #1:** `on[Gesture]Start` and `on[Gesture]End` methods are provided as a commodity. `on[Gesture]` handlers receive `first` and `last` properties that indicate if the event fired is the first (i.e. gesture has started) or the last one (i.e. gesture has ended).

**Note #2:** since browsers don't have native event listeners for when scroll, move or wheel ends, react-use-gesture debounces these events to estimate when they stopped. One of the consequence of debouncing is trying to access properties from the native event when a gesture has ended will probably result in a warning: [React does event pooling](https://reactjs.org/docs/events.html#event-pooling), meaning a React event can only be queried synchronously.

### Adding gestures to dom nodes

React-use-gesture also supports to add handlers to dom nodes directly (or the `window` or `document` objects). In that case, you shouldn't spread the `bind()` object returned by `useGesture` as a prop, but use the `React.useEffect` hook as below.

```js
// this will add a scroll listener to the window
const bind = useGesture({
  onScroll: state => doStuff,
  config: { domTarget: window }
})
React.useEffect(bind, [bind])
```

_Note that using `useEffect` will also take care of removing event listeners when the component is unmounted._

### Shortcut to the drag event handler

Although React-use-gesture was initially developed to support drag events only (press, move and release), this library now supports hover, move, scroll and wheel events. To ensure retro-compatibility with **v4.x**, **v5.x** still gives you a shortcut to the `onDrag` and pass directly the handler function as the sole argument of `useGesture`

```jsx
// This:
const bind = useGesture(state => doStuff)
// is equivalent to this:
const bind = useGesture({ onDrag: state => doStuff })
```

### `useGesture` event state

Every time a handler is called, it will get passed the current event state of its corresponding gesture. An event state is an object that includes the source event and adds multiple attributes listed below:

| Name                                             | Type           | Description                                                                                                                                                                  |
| ------------------------------------------------ | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `event`                                          | `object`       | source event                                                                                                                                                                 |
| `time`                                           | `Number`       | timestamp of the current gesture                                                                                                                                             |
| `xy`                                             | `Vec2 ([x,y])` | for touch/mouse events, `xy` returns the position of the pointer on the screen. For scroll/wheel events `xy` returns how much the element has been scrolled on x and y axis. |
| `previous`                                       | `Vec2`         | previous `xy`                                                                                                                                                                |
| `initial`                                        | `Vec2`         | `xy` value when the gesture has started                                                                                                                                      |
| `delta`                                          | `Vec2`         | delta offset (`xy - initial`)                                                                                                                                                |
| `local`                                          | `Vec2`         | delta with book-keeping (remembers the `xy` value throughout gestures)                                                                                                       |
| `lastLocal`                                      | `Vec2`         | previous `local`                                                                                                                                                             |
| `vxvy`                                           | `Vec2`         | momentum / sped of the gesture (x and y axis separated)                                                                                                                      |
| `velocity`                                       | `Number`       | momentum / speed of the gesture (x and y axis combined)                                                                                                                      |
| `distance`                                       | `Number`       | delta distance                                                                                                                                                               |
| `first`                                          | `Boolean`      | marks the first event                                                                                                                                                        |
| `last`                                           | `Boolean`      | marks the last event                                                                                                                                                         |
| `active`                                         | `Boolean`      | `true` when the gesture is active, `false` otherwise                                                                                                                         |
| `temp`                                           | `Any`          | serves as a cache storing any value returned by your handler during its previous run. See below for an example.                                                              |
| `cancel`                                         | `Function`     | you can call `cancel` to interrupt the drag gesture. `cancel`is only relevant in the `onDrag` handler.                                                                       |
| `down`                                           | `Boolean`      | mouse / touch down                                                                                                                                                           |
| `touches`                                        | `Number`       | number of touches pressing the screen                                                                                                                                        |
| `shiftKey`<br>`altKey`<br>`ctrlKey`<br>`metaKey` | `Boolean`      | modifier keys are pressed                                                                                                                                                    |
| `dragging`                                       | `Boolean`      | `true` when the user is dragging                                                                                                                                             |
| `moving`                                         | `Boolean`      | `true` when the user is moving the mouse                                                                                                                                     |
| `hovering`                                       | `Boolean`      | `true` when the mouse hovers the element                                                                                                                                     |
| `scrolling`                                      | `Boolean`      | `true` when the user is scrolling                                                                                                                                            |
| `wheeling`                                       | `Boolean`      | `true` when the user is wheeling                                                                                                                                             |
| `args`                                           | `Any`          | arguments you passed to `bind`                                                                                                                                               |

### `useGesture` config

You can pass a `config` object to `useGesture` to customize its behavior.

| Name        | Default Value                     | Description                                                                                                                                                                            |
| ----------- | --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `domTarget` | `undefined`                       | lets you specify a dom node you want to attach gestures to (body, window, document...)                                                                                                 |
| `event`     | `{passive: true, capture: false}` | the event config attribute lets you configure `passive` and `capture` options passed to event listeners                                                                                |
| `transform` | `{x: x => x, y =>y }`             | Transform functions you can pass to modify `x` and `y` values.                                                                                                                         |
| `window`    | `window`                          | Lets you specify which `window` element `useGesture` should use. See this [thread](https://github.com/react-spring/react-use-gesture/pull/43#issue-262835054) for a relevant use case. |

### Examples

#### React hooks with onAction (and react-spring) (basic pull & release)

Demo: https://codesandbox.io/s/r24mzvo3q

<p align="middle">
  <img src="https://i.imgur.com/JyeQsEI.gif" width="200"/>
</p>

#### React hooks with onAction (and react-spring) (decay)

Demo: https://codesandbox.io/s/zq19y1xr9m

This demo reads out further data like velocity and direction to calculate decay. `temp` in this case is a simple storage that picks up whatever value you (optionally) return inside the event handler. It's valid as long as the gesture is active. Without this you would need to store the initial xy value somewhere else and conditionally update it when the gesture begins.

```jsx
const [{ xy }, set] = useSpring(() => ({ xy: [0, 0] }))
const bind = useGesture(({ down, delta, velocity, direction, temp = xy.getValue() }) => {
  set({
    xy: add(delta, temp),
    immediate: down,
    config: { velocity: scale(direction, velocity), decay: true }
  })
  return temp
})
return <animated.div {...bind()} style={{ transform: xy.interpolate((x, y) => `translate3d(${x}px,${y}px,0)`) }} />
```

### Frequently asked questions

**What are the differences between using `useGesture` and adding listeners manually?**
Not a lot! Essentially `useGesture` simplifies the implementation of the drag gesture, calculates kinematics values you wouldn't get out of the box from the listeners, and debounces move scroll and wheel events to let you know when they end.

**How do you pass state to `useGesture`?**
The recommended way of passing an external value to `useGesture` is by using `React.useRef`.

**Why `onMove` when `onDrag` already exists?**
`onDrag` only fires while your touch or press the element. You just need to hover your mouse above the element to trigger `onMove`.

**Why `onWheel` and `onScroll`?**
Scrolling and wheeling are structurally different events although they produce similar results most of the time. First of all, `wheel` is a mouse-only event. For `onScroll` to be fired, the element you're scrolling needs to actually scroll, therefore have content overflowing, while you just need to wheel over an element to trigger `onWheel`. If you use [react-three-fiber](https://github.com/drcmda/react-three-fiber), `onWheel` might prove useful to simulate scroll on canvas elements.
