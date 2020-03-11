---
layout: single
title:  "Pass data to any component using Event Bus - Vuejs"
categories:
  - Web
tags:
  - vue-js
---
Passing data between components sometimes can be tough, we can't always rely on `$emit` events, especially if we are communicating between child components.

Although there's `Vuex` for managing states. But for simplier and not so complex architecture;implementing `EventBus` is our perfect solution.

### 1. Create EventBus.js file
Add `EventBus.js` file somewhere on your src folder or desire path. make sure you can import it easily.
```javascript
// EventBus.js
import Vue from 'vue'
export default new Vue()
```

### 2. Import `EventBus.js` to component-a
Let's say our component-a was the source of Object we are passing to other components. the method below should trigger via `@click` event in order to emit the payload.
```javascript
// component-a.js
import EventBus from '../EventBus'

export default {
    methods() {
        let payload = ['apple','banana','orange]; // example object
        EventBus.$emit('EVENT_NAME', payload);
    }
}
```

### 3. Transport the event to component-b
After we registered the `EVENT_NAME` to EventBus payload. it's now time to transport the data to our desired component. for this example, see below.
```javascript
// component-b.js
import EventBus from '../EventBus'

export default {
    data() {
        fruits: []
    },
    mounted() {
        EventBus.$on('EVENT_NAME', (payload) => {
            this.fruits = payload
        })
    }
}
```
As simple as that. what we did was to load the passed Object through `mounted()` hook into the `fruits` data array.

So that's it. happy coding!!
