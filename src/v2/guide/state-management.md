---
title: State Management
type: guide
order: 502
---

## Official Flux-Like Implementation

Large applications can often <span class='definition'>grow in complexity</span>, due to <span class='important'>multiple pieces of state scattered across many components and the interactions between them</span>. To solve this problem, Vue offers [vuex](https://github.com/vuejs/vuex): our own Elm-inspired state management library. It even integrates into [vue-devtools](https://github.com/vuejs/vue-devtools), providing zero-setup access to [time travel debugging](https://raw.githubusercontent.com/vuejs/vue-devtools/master/media/demo.gif).

<div class="vue-mastery"><a href="https://www.vuemastery.com/courses/mastering-vuex/intro-to-vuex/" target="_blank" rel="sponsored noopener" title="Vuex Tutorial">Watch a video explanation on Vue Mastery</a></div>

### Information for React Developers

If you're <span class='definition'>coming from React</span>, you may be wondering how vuex compares to [redux](https://github.com/reactjs/redux), the most popular Flux implementation in that ecosystem. <span class='definition'>Redux</span> is actually view-layer agnostic, so it can easily be used with Vue via [simple bindings](https://classic.yarnpkg.com/en/packages?q=redux%20vue&p=1). <span class='important'>Vuex is different in that it _knows_ it's in a Vue app</span>. This allows it to better integrate with Vue, offering a more intuitive API and improved development experience.

## Simple State Management from Scratch

It is often overlooked that the <span class='definition'>source of truth</span> in Vue applications is the raw `data` object - a <span class='important'>Vue instance only proxies access to it</span>. Therefore, if you have a piece of state that should be shared by multiple instances, you can share it by identity:

``` js
var sourceOfTruth = {}

var vmA = new Vue({
  data: sourceOfTruth
})

var vmB = new Vue({
  data: sourceOfTruth
})
```

Now whenever `sourceOfTruth` is <span class='definition'>mutated</span>, both `vmA` and `vmB` will <span class='definition'>update</span> their views automatically. <span class='definition'>Subcomponents</span> within each of these instances would also have access via `this.$root.$data`. We have a <span class='definition'>single source of truth</span> now, but <span class='definition'>debugging would be a nightmare</span>. Any piece of data could be changed by any part of our app at any time, without leaving a trace.

To help solve this problem, we can adopt a <span class='definition'>**store pattern**</span>:

``` js
var store = {
  debug: true,
  state: {
    message: 'Hello!'
  },
  setMessageAction (newValue) {
    if (this.debug) console.log('setMessageAction triggered with', newValue)
    this.state.message = newValue
  },
  clearMessageAction () {
    if (this.debug) console.log('clearMessageAction triggered')
    this.state.message = ''
  }
}
```

Notice all <span class='definition'>actions</span> that mutate the store's state are put inside the store itself. This type of <span class='definition'>centralized state management</span> makes it easier to understand what type of mutations could happen and how they are triggered. Now when something goes wrong, we'll also have a log of what happened leading up to the bug.

In addition, each instance/component can still own and manage its own <span class='definition'>private state</span>:

``` js
var vmA = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
})

var vmB = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
})
```

![State Management](/images/state.png)

<p class="tip">It's important to note that <span class='important'>you should never replace the original state object</span> in your actions - <span class='important'>the components and the store need to share reference to the same object in order for mutations to be observed</span>.</p>

As we continue developing the convention where components are never allowed to directly mutate state that belongs to a store, but should instead <span class='definition'>dispatch events that notify the store to perform actions</span>, we eventually arrive at the [Flux](https://facebook.github.io/flux/) architecture. The benefit of this convention is we can <span class='important'>record all state mutations happening to the store and implement advanced debugging helpers such as mutation logs, snapshots, and history re-rolls / time travel</span>.

This brings us full circle back to [vuex](https://github.com/vuejs/vuex), so if you've read this far it's probably time to try it out!
