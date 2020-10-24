---
title: Animate a number with Greensock
date: 2020-10-24T12:00
---

This effect can bring a little bit of life into a display of key numbers say in an annual report or similar. We animate counting from one number to another one. This effect is further enhanced by only starting once the whole number is in the viewport for the user, thus making sure that the effect can be seen, no matter where on the page it is positioned.

## Basic markup

Make sure that the animated number is the only content of some element. This is because the animated value will be injected into the DOM using Javascript's `element.innerHTML`. We also use some data attributes that can be set up using a CMS to make customization of the animation easily accessible from a backend.

```html
<div
  class="counter"
  data-counter-start="0"
  data-counter-end="238"
  data-counter-duration="2"
>
  <span class="counter-number">0</span>
  <span class="counter-legend">Views</span>
</div>
<div
  class="counter"
  data-counter-start="10"
  data-counter-end="0"
  data-counter-duration="10"
>
  <span class="counter-number">10</span>
  <span class="counter-legend">Seconds to go</span>
</div>
```

Feel free to style this component to your liking or follow the example given below.

```css
.counter {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.counter-number {
  font-size: 3em;
  font-weight: bold;
}
```

## Animating with Greensock

Make sure you have the Greensock (gsap) library available.

```bash
npm install --save gsap
```

Import gsap and its ScrollTrigger plugin, register it and set up some sensible defaults.

```js
import { gsap } from "gsap"
import { ScrollTrigger } from "gsap/ScrollTrigger"

gsap.registerPlugin(ScrollTrigger)
ScrollTrigger.defaults({
  toggleActions: "restart none none none",
})
```

The `toggleActions` define what happens in each possible case that can occur when scrolling. They are best explained as follows: `toggleActions: "onEnter, onLeaveDown, onReenter, onLeaveUp"`. In this case we restart the count whenever it comes into view while the user is scrolling down. We always run the whole animation and only restart if the user scrolls up and then down again.

We allow for multiple counters per page as is also shown in the example. Get all of them into an array so you can loop over them with `forEach` using a handy GSAP utility.

```js
gsap.utils.toArray(".counter").forEach(counter => {})
```

For each counter we will now read in the data attributes containing the necessary animation information. We also define the element that contains the actual number and declare a counting variable where the actual counting will happen. This variable is necessary because its value will be injected into the DOM on every animation update.

```js
gsap.utils.toArray(".counter").forEach(counter => {
  const startValue = counter.getAttribute("data-counter-start")
  const endValue = counter.getAttribute("data-counter-end")
  const duration = counter.getAttribute("data-counter-duration")
  const counterElement = counter.querySelector(".count-number")
  const counting = { val: startNumber }

  // We animate the counting variable
  gsap.to(counting, {
    // Trigger the animation once the top of the counter element has passed the bottom of the viewport by 50px
    scrollTrigger: {
      trigger: counter,
      start: "top bottom-=50px",
    },
    // Set the value to which we are animating -> references the val key in the counting variable
    val: endValue,
    duration: duration,
    onUpdate(): () => {
      // This callback runs on every animation update. Use this to format the number to your wishes, then inject it into the DOM.
      const number = gsap.utils.snap(1, counting.val)
      countElement.innerHTML = number;
    }
  })
});
```

Most of what is happening here is explained inline using comments. We set up the scroll trigger which defines when to start the animation. Since we are animation an object (`counting`), we reference the object property that is to be animated and set up the targeted end value.

To format the number `onUpdate` here we are using another gsap utility function that removes all possible digits from the number.
