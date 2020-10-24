---
title: Functional React Components
date: 2020-10-23T16:50
---

This article will explain how to migrate a class based component to a function based one. With the introduction of hooks and clear focus on functions by the React core team this move seems advisable.

## Props

In functional components props are passed as arguments directly to the function. They can also be destructured as argument. To migrate make sure to migrate `this.props` to `props` everywhere.

## State

To make use of state in functional components React provides the `useState()` hook. Make sure to import the hook and then use it as follows.

```jsx
import React, { useState } from "react"

function Example() {
  // Declare a new state variable called count
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>You have clicked the button {count} times.</p>
      <button onClick={setCount(count + 1)}>Click me</button>
    </div>
  )
}
```

When initializing state you first declare the state variable (above `count`) and the function to manipulate that variable (above `setCount`). The optional argument in the `useState()` function can be used to set up an initial state.

If you want/need to set up multiple state variables jsut call `useState()` multiple times.

## Side effects

Observe changes and react to them similar to how you do in class based components with `componentDidMount`, `componentDidUpdate` and `componentWillUnmount` using the `useEffect` hook. Here is an example:

```jsx
import React, { useState, useEffect } from "react"

function Example() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    // Update the document title using the browser API.
    document.title = `You clicked ${count} times.`
  })

  return (
    <div>
      <p>You have clicked the button {count} times.</p>
      <button onClick={setCount(count + 1)}>Click me</button>
    </div>
  )
}
```

Usual examples of side effects include data fetching, setting up a subscription or manually changing the DOM in React components. Using this Hook informs your component that it needs to do something after render. By calling it within the component it shares the component's scope and thus gives us direct access to, for example state variables. By default `useEffect` runs after both the first render and after every update. React does not run this effect until the DOM has been updated.

### Effects with cleanup

Most effects do not need any cleanup however there are some, like setting up a subscription to an external data source, that needs cleanup to prevent a memory leak. In classes this was done by using `componentDidMount` to set up a subscription and clean it up with `componentWillUnmount`. Following is an example of subscribing to a friend's online status, including the clean up all within `useEffect`.

```jsx
import React, { useState, useEffect } from "react"

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null)

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline)
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange)

    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange)
    }
  })

  if (isOnline === null) {
    return "… Loading"
  }

  return isOnline ? "Online" : "Offline"
}
```

To clean up after an effect return a function that does exactly that within `useEffect`, as is done in the example above with `cleanup()` (however this function may also just be anonoymous). By making subscribing and unsubscribing part of the same effect we are able to keep these closely related actions right next to each other.

React does not only clean up after itself before unmounting the component but before every render. This makes sure previous effects are properly cleaned up before a new render.

### More useful knowledge

It is possible to use multiple effects if you wish to separate logic. Just call everything that is related to each other within one `useEffect`.

To optimize performance the Hook can be instructed to only run if certain values actually have changed. To do this, just add the variables to observe as the second argument in an array.

```jsx
useEffect(() => {
  document.title = `You clicked ${count} times.`
}, [count])
```

With this use of the Hook, on every update the value of `count` is compared to the previous state and the effect only run if `count` has changed. If you add multiple values to the array, the effect is run if at least one of them has changed.

Pass an empty array `[]` as second argument if you want the effect to only run on mount and cleanup on unmount.
