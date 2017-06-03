# aMPL
a Magic Persistence Layer

- [Guide](#guide)
- [Development](#development)
- [API](#api)

## Guide

This guide will demonstrate how to create a basic counter app using Electron, React, and aMPL. First, start by creating a new Electron app using Electron Forge with the React template.

```bash
$ npm install -g electron-forge
$ electron-forge init demo --template=react
```

Now, install aMPL:

```bash
$ npm install git+ssh://git@github.com/inkandswitch/ampl.git
```

In src/app.jsx, initialize the aMPL store and add the counter to the view:

```js
import React from 'react'
import aMPL from 'ampl'

export default class App extends React.Component {
  constructor() {
    super()

    // Configure our Slack signaler by setting
    // process.env.SLACK_BOT_TOKEN or ampl.config.slackBotToken
    aMPL.config.slackBotToken = "some token"

    this.store = new aMPL.Store((state, action) => {
      switch(action.type) {
        case "INCREMENT_COUNTER":
          return aMPL.Tesseract.set(state, "counter", (state.counter || 0) + 1)
        default:
          return state
      }
    })

    // Force App component to re-render when state changes
    this.store.subscribe(() => this.setState({}))
  }

  componentDidMount() {
    // Normally your app would get your document id via URL or from a file,
    // but here we will fix it to "1" so our clients join the same doc
    this.store.dispatch({ type: "OPEN_DOCUMENT", docId: "1" })
  }

  render() {
    return <div>
      <h2>Counter: { this.store.getState().counter }</h2>
      <button onClick={ () => this.store.dispatch({type: "INCREMENT_COUNTER"}) } >
        Increment
      </button>
    </div>
  }
}
```

Start two clients and try incrementing the counter, you should see the counters synchronize on both clients:

```bash
$ npm start & npm start
```

## Development

### Compiling

aMPL is configured via Babel to use ES2016 syntax. The source code is located in `src` and compiled code in `lib`. Make sure to compile and commit before creating a new release:

```bash
$ npm run compile
```

### Testing

To run tests:

```bash
$ npm run test
```

## API

### Store

`aMPL.Store` is modeled off of [Redux](http://redux.js.org/) and follows the same basic pattern.

#### new Store(reducer)

The `Store` constructor accepts a reducer function. When an action is dispatched to the store, it first checks against aMPL's provided actions and then invokes your reducer if the action did not map to any of the provided ones.

#### getState()

`getState` returns the current state object including all of your persisted data.

#### dispatch(action)

`dispatch` sends an action through your reducer. You should only modify the state through `dispatch`. Note: dispatch is a synchronous function.

#### subscribe(listener)

`subscribe` allows to register a listener callback. All listeners are invoked whenver there is a state change in the store, including inbound changes that come in through other peers over the network.
