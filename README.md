# State Management

## Table of Contents
- [State Management](#state-management)
  - [Table of Contents](#table-of-contents)
  - [**01. Introduction**](#01-introduction)
    - [Types of State](#types-of-state)
  - [**02. Class-Based State**](#02-class-based-state)
    - [setState & Class](#setstate--class)
    - [setState & Asynchronicity](#setstate--asynchronicity)
    - [setState & Function](#setstate--function)
    - [setState & Callback](#setstate--callback)
    - [setState & Helper Function](#setstate--helper-function)
    - [document.title Exercise](#documenttitle-exercise)
    - [document.title Solution](#documenttitle-solution)
    - [setState Patterns](#setstate-patterns)
      - [When we’re working with props, we have PropTypes. That’s not the case with state.](#when-were-working-with-props-we-have-proptypes-thats-not-the-case-with-state)
      - [Don’t use this.state for derivations of props.](#dont-use-thisstate-for-derivations-of-props)
      - [Don’t do this. Instead, derive computed properties directly from the props themselves.](#dont-do-this-instead-derive-computed-properties-directly-from-the-props-themselves)
      - [You don’t need to shove everything into your render method.](#you-dont-need-to-shove-everything-into-your-render-method)
      - [You can break things out into helper methods.](#you-can-break-things-out-into-helper-methods)
      - [Don’t use state for things you’re not going to render.](#dont-use-state-for-things-youre-not-going-to-render)
      - [Use sensible defaults.](#use-sensible-defaults)
  - [**03. Hooks State**](#03-hooks-state)
    - [Refactoring & Hooks](#refactoring--hooks)
    - [useEffect & Dependencies](#useeffect--dependencies)
    - [useEffect Exercise](#useeffect-exercise)
    - [useEffect Solution](#useeffect-solution)
    - [Refactoring & Custom Hook](#refactoring--custom-hook)
    - [Persisting State & useRef](#persisting-state--useref)
      - [How do lifecycle methods and hooks differ?](#how-do-lifecycle-methods-and-hooks-differ)
    - [useEffect & Cleanup](#useeffect--cleanup)
  - [**04. Reducers**](#04-reducers)
    - [useReducer Introduction](#usereducer-introduction)
    - [Reducer Action & State](#reducer-action--state)
    - [Reducer Action Keys & dispatch](#reducer-action-keys--dispatch)
    - [Action & State Modification Exercise](#action--state-modification-exercise)
    - [Action & State Modification Solution](#action--state-modification-solution)
    - [React.memo & [useCallback](https://reactjs.org/docs/hooks-reference.html#usecallback)](#reactmemo--usecallback)
  - [**05. Context**](#05-context)
    - [Prop Drilling & Context API](#prop-drilling--context-api)
    - [Creating a Context Provider](#creating-a-context-provider)
    - [Context & useContext Hook](#context--usecontext-hook)
    - [Context Practice](#context-practice)
  - [**06. Data Fetching**](#06-data-fetching)
  - [**07. Thunks**](#07-thunks)

## **01. Introduction**

The main job of React is to take your application state and turn it into DOM nodes.

[pure-react-state-management](https://github.com/FrontendMasters/pure-react-state-management)

### Types of State

There are many kinds of state.
- Model data: The nouns in your application.
  - For example: products, orders
- View/UI state: Are those nouns sorted in ascending or descending order?
  - For example: list of sorted products
- Session state: Is the user even logged in?
- Communication: Are we in the process of fetching the nouns from the server?
  - For example: loading, loaded or error
- Location: Where are we in the application? Which nouns are we looking at?
  - For example: routing, shopping cart, view products

Or, it might make sense to think about state relative to time.

- Model state: This is likely the data in your application. This could be the items in a given list.
- Ephemeral state: Stuff like the value of an input field that will be wiped away when you hit “enter.” This could be the order in which a given list is sorted.

**[⬆ back to top](#table-of-contents)**

## **02. Class-Based State**

### setState & Class

```javascript
import React, { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };

    this.increment = this.increment.bind(this);
    this.decrement = this.decrement.bind(this);
    this.reset = this.reset.bind(this);
  }

  increment() {
    this.setState({ count: this.state.count + 1 })
  }

  decrement() {
    this.setState({ count: this.state.count - 1 })
  }

  reset() {
    this.setState({ count: 0 })
  }

  render() {
    const { count } = this.state;

    return (
      <div className="Counter">
        <p className="count">{count}</p>
        <section className="controls">
          <button onClick={this.increment}>Increment</button>
          <button onClick={this.decrement}>Decrement</button>
          <button onClick={this.reset}>Reset</button>
        </section>
      </div>
    );
  }
}

export default Counter;
```

**[⬆ back to top](#table-of-contents)**

### setState & Asynchronicity

```javascript
// this.setState() is asynchronous.
// React is trying to avoid unnecessary re-renders.
this.setState({ count: this.state.count + 1 });
this.setState({ count: this.state.count + 1 });
this.setState({ count: this.state.count + 1 });
console.log(this.state.count);  // 0
```

```javascript
// What will the count be after the user’s clicks the “Increment” button? 1
// Effectively, you’re queuing up state changes.
// React will batch them up, figure out the result and then efficiently make that change.
export default class Counter extends Component {
  constructor() { ... }
  increment() {
    this.setState({ count: this.state.count + 1 });
    this.setState({ count: this.state.count + 1 });
    this.setState({ count: this.state.count + 1 });
  }
  render() { ... }
}

Object.assign(
  {},
  yourFirstCallToSetState,
  yourSecondCallToSetState,
  yourThirdCallToSetState,
);

const newState = {
  ...yourFirstCallToSetState,
  ...yourSecondCallToSetState,
  ...yourThirdCallToSetState,
};
```

**[⬆ back to top](#table-of-contents)**

### setState & Function

- There is actually a bit more to this.setState().
- Did you know that you can also pass a function in as an argument?

```javascript
// 3
// can merge object, cannot merge function
// When you pass functions to this.setState(), it plays through each of them.
import React, { Component } from 'react';
export default class Counter extends Component {
  constructor() { ... }

  increment() {
    this.setState(state => { return { count: state.count + 1 } });
    this.setState(state => { return { count: state.count + 1 } });
    this.setState(({ count }) => { return { count: state.count + 1 } });
  }

  render() { ... }
}

import React, { Component } from 'react';
export default class Counter extends Component {
  constructor() { ... }
  increment() {
    this.setState(state => {
      if (state.count >= 5) return;
      return { count: state.count + 1 };
    })
  }

  render() { ... }
}
```

**[⬆ back to top](#table-of-contents)**

### setState & Callback

```javascript
import React, { Component } from 'react';
export default class Counter extends Component {
  constructor() { ... }
  increment() {
    this.setState(
      { count: this.state.count + 1 },
      () => { console.log(this.state); }
    )
  }
  render() { ... }
}
```

**[⬆ back to top](#table-of-contents)**

### setState & Helper Function

```javascript
const getStateFromLocalStorage = () => {
  const storage = localStorage.getItem('counterState');
  // storage = '{"count":0}'
  if (storage) return JSON.parse(storage);
  return { count: 0 };
};

const storeStateInLocalStorage = state => {
  localStorage.setItem('counterState', JSON.stringify(state));
  // localStorage = Storage { counterState: '{"count":0}' }
  console.log(localStorage);
};

const increment = (state, props) => {
  const { max, step } = props;
  if (state.count >= max) return;
  return { count: state.count + step };
}

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = getStateFromLocalStorage();

    this.increment = this.increment.bind(this);
    this.decrement = this.decrement.bind(this);
    this.reset = this.reset.bind(this);
  }

  increment() {
    this.setState(increment, () => storeStateInLocalStorage(this.state));
    console.log('Before!', this.state);
  }

  render() { ... }
}
```

**[⬆ back to top](#table-of-contents)**

### document.title Exercise
### document.title Solution

```javascript
class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = getStateFromLocalStorage();

    this.increment = this.increment.bind(this);
    this.decrement = this.decrement.bind(this);
    this.reset = this.reset.bind(this);
    this.updateDocumentTitle = this.updateDocumentTitle.bind(this);
  }

  updateDocumentTitle() {
    document.title = this.state.count;
  }

  increment() {
    this.setState(increment, this.updateDocumentTitle);
    console.log('Before!', this.state);
  }

  decrement() {
    this.setState({ count: this.state.count - 1 }, this.updateDocumentTitle);
  }

  reset() {
    this.setState({ count: 0 }, this.updateDocumentTitle);
  }

  render() { ... }
}
```

**[⬆ back to top](#table-of-contents)**

### setState Patterns

#### When we’re working with props, we have PropTypes. That’s not the case with state.

```javascript
function shouldIKeepSomethingInReactState(){
  if(canICalculateItFromProps()){
    // Don't duplicate data from props in state.
    // Calculate what you can in render() method.
    return false;
  }
  if(!amIUsingItInRenderMethod()){
    // Don't keep something in the state
    // if you don't use it for rendering.
    // For example, API subscriptions are
    // better off as custom private fields
    // or variables in external modules.
    return false;
  }
  // You can use React state for this!
  return true;
}
```

**[⬆ back to top](#table-of-contents)**

#### Don’t use this.state for derivations of props.

```javascript
class User extends Component {
  constructor(props) {
    super(props);
    this.state = {
      fullName: props.firstName + ' ' + props.lastName
    };
  }
}
```

**[⬆ back to top](#table-of-contents)**

#### Don’t do this. Instead, derive computed properties directly from the props themselves.

```javascript
class User extends Component {
  render() {
    const { firstName, lastName } = this.props;
    const fullName = firstName + ' ' + lastName;
    return (
      <h1>{fullName}</h1>
    );
  }
}

// Alternatively...
class User extends Component {
  get fullName() {
    const { firstName, lastName } = this.props;
    return firstName + ' ' + lastName;
  }
  render() {
    return (
      <h1>{this.fullName}</h1>
    );
  }
}
```

**[⬆ back to top](#table-of-contents)**

#### You don’t need to shove everything into your render method.
#### You can break things out into helper methods.

```javascript
class UserList extends Component {
  render() {
    const { users } = this.props;
    return (
      <section>
        <VeryImportantUserControls />
        { users.map(user => (
          <UserProfile
            key={user.id}
            photograph={user.mugshot}
            onLayoff={handleLayoff}
          /> 
        )) }
        <SomeSpecialFooter />
      </section>
    );
  }
}
```

```javascript
class UserList extends Component {
  renderUserProfile(user) {
    return (
      <UserProfile
        key={user.id}
        photograph={user.mugshot}
        onLayoff={handleLayoff}
      />
    )
  }

  render() {
    const { users } = this.props;
    return (
      <section>
        <VeryImportantUserControls />
          { users.map(this.renderUserProfile) }
        <SomeSpecialFooter />
      </section>
    );
  }
}
```

```javascript
const renderUserProfile = user => {
  return (
    <UserProfile
      key={user.id}
      photograph={user.mugshot}
      onLayoff={handleLayoff}
    />
  );
};

const UserList = ({ users }) => {
  return (
    <section>
      <VeryImportantUserControls />
      {users.map(renderUserProfile)}
      <SomeSpecialFooter />
    </section>
  );
};
```

**[⬆ back to top](#table-of-contents)**

#### Don’t use state for things you’re not going to render.

```javascript
class TweetStream extends Component {
  constructor() {
    super();
    this.state = {
      tweets: [],
      tweetChecker: setInterval(() => {
        Api.getAll('/api/tweets').then(newTweets => {
          const { tweets } = this.state;
          this.setState({ tweets: [ ...tweets, newTweets ] });
        });
      }, 10000)
    }
  }

  componentWillUnmount() {
    clearInterval(this.state.tweetChecker);
  }

  render() { // Do stuff with tweets }
}
```

```javascript
class TweetStream extends Component {
  constructor() {
    super();
    this.state = {
      tweets: []
    }
  }

  componentWillMount() {
    this.tweetChecker = setInterval( ... );
  }

  componentWillUnmount() {
    clearInterval(this.state.tweetChecker);
  }

  render() { // Do stuff with tweets }
}
```

**[⬆ back to top](#table-of-contents)**

#### Use sensible defaults.

```javascript
class Items extends Component {
  constructor() {
    super();
  }

  componentDidMount() {
    Api.getAll('/api/items').then(items => {
      this.setState({ items });
    });
  }

  render() { // Do stuff with items }
}
```

```javascript
class Items extends Component {
  constructor() {
    super();
    this.state = {
      items: []
    }
  }

  componentDidMount() {
    Api.getAll('/api/items').then(items => {
      this.setState({ items });
    });
  }

  render() { // Do stuff with items }
}
```

**[⬆ back to top](#table-of-contents)**

## **03. Hooks State**

### Refactoring & [Hooks](https://reactjs.org/docs/hooks-state.html)

```javascript
const [count, setCount] = React.useState(0);
const increment = () => setCount(count + 1);
const decrement = () => setCount(count - 1);
const reset = () => setCount(0);
```

```javascript
const increment = () => {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
};
```

```javascript
const increment = () => {
  setCount(c => c + 1);
  setCount(c => c + 1);
  setCount(c => c + 1);
};
```

```javascript
setCount(c => {
  if (c >= max) return c;
  return c + 1;
});
```

**[⬆ back to top](#table-of-contents)**

### [useEffect](https://reactjs.org/docs/hooks-effect.html) & Dependencies

Side effects

- Anything that is not return value is side effects
- Ajax request
- console log

```javascript
useEffect(() => {
  document.title = `Counter: ${count}`
}, [count]);
```

**[⬆ back to top](#table-of-contents)**

### useEffect Exercise

Can you add a second effect that updates the document's title whenever the count changes?

**[⬆ back to top](#table-of-contents)**

### useEffect Solution

```javascript
const getStateFromLocalStorage = () => {
  const storage = localStorage.getItem('counterState');
  // storage = '{"count":0}'
  console.log(storage);
  if (storage) return JSON.parse(storage).count;
  return 0;
};

const storeStateInLocalStorage = count => {
  localStorage.setItem('counterState', JSON.stringify({ count }));
  // localStorage = Storage { counterState: '{"count":0}' }
  console.log(localStorage);
};

const Counter = ({ max, step }) => {
  const [count, setCount] = useState(getStateFromLocalStorage());

  useEffect(() => {
    storeStateInLocalStorage(count);
  }, [count]);

  return ( ... );
};
```

**[⬆ back to top](#table-of-contents)**

### Refactoring & [Custom Hook](https://reactjs.org/docs/hooks-custom.html)

```javascript
const useLocalStorage = (initialState, key) => {
  const get = () => {
    const storage = localStorage.getItem(key);
    // localStorage = Storage { count: '{"value":0}' }
    // storage = '{"value":0}'
    console.log(localStorage);
    console.log(storage);
    if (storage) return JSON.parse(storage).value;
    return initialState;
  };

  const [value, setValue] = useState(get());

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify({ value }));
    // eslint-disable-next-line
  }, [value]);

  return [value, setValue];
};

const Counter = ({ max, step }) => {
  const [count, setCount] = useLocalStorage(0, 'count');

  return ( ... );
};
```

**[⬆ back to top](#table-of-contents)**

### Persisting State & [useRef](https://reactjs.org/docs/hooks-reference.html#useref)

#### How do lifecycle methods and hooks differ?

```javascript
componentDidUpdate() {
  setTimeout(() => {
    console.log(`Count: ${this.state.count}`);
  }, 3000);
}
```

```javascript
React.useEffect(() => {
  setTimeout(() => {
    console.log(`Count: ${count}`);
  }, 3000);
}, [count]);
```

- access the DOM
- keep any mutable value around 

```javascript
const Counter = ({ max, step }) => {
  const [count, setCount] = useState(0);
  const countRef = useRef();
  // countRef = { current: null }

  let message = '';
  if (countRef.current < count) message = 'Higher';
  if (countRef.current > count) message = 'Lower';

  countRef.current = count;
  // countRef = { current: 0 }

  const increment = () => {
    setCount(c => c + 1);
  };

  useEffect(() => {
    setTimeout(() => {
      console.log(`Count: ${count}`);
    }, 3000);
  }, [count]);

  return ( ... );
};
```

**[⬆ back to top](#table-of-contents)**

### useEffect & Cleanup

```javascript
useEffect(() => {
  const id = setInterval(() => {
    console.log(`Count: ${count}`);
  }, 3000);
  return () => clearInterval(id);
}, [count])
```

**[⬆ back to top](#table-of-contents)**

## **04. Reducers**

[Grudges](https://github.com/stevekinney/grudges-react-state)

### [useReducer](https://reactjs.org/docs/hooks-reference.html#usereducer) Introduction

What’s the deal with useReducer()?

- So, it turns out that it has nothing to do with Redux. 
- But, it does allow you to use reducers—just like Redux.
- The cool part is that it allows you to create interfaces where you (or a friend) can pass in the mechanics about how to update state.

- [React's useReducer Hook vs Redux](https://www.robinwieruch.de/redux-vs-usereducer)
- [Immer](https://github.com/immerjs/immer)
- [Immutable](https://immutable-js.github.io/immutable-js/)

| Redux                                                                             | useReducer                                                                  |
| --------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **one** **global** state container                                                | independent component co-located state container                            |
| combines all reducers to one ultimate reducer                                     | Not Available (Not one)                                                     |
| one dispatch function that consumes any action dedicated for any reducer function | dispatch only deals with actions that are specified by the reducer function (Not global) |
| comes with a rich middleware ecosystem                                            | no middleware for useReducer                                                |

Redux Middleware
- [redux-logger](https://github.com/LogRocket/redux-logger)
- [redux-thunk](https://github.com/reduxjs/redux-thunk)
- [redux-saga](https://github.com/redux-saga/redux-saga)

| When?                             | What?                              |
| --------------------------------- | ---------------------------------- |
| simple/small size applications    | useState                           |
| advanced/medium size applications | useState + useReducer + useContext |
| complex/large size applications   | useState + Redux                   |

```javascript
  const [grudges, setGrudges] = useState(initialState);

  const addGrudge = grudge => {
    grudge.id = id();
    grudge.forgiven = false;
    setGrudges([grudge, ...grudges]);
  };

  const toggleForgiveness = id => {
    setGrudges(
      grudges.map(grudge => {
        if (grudge.id !== id) return grudge;
        return { ...grudge, forgiven: !grudge.forgiven };
      })
    );
  };
```

**[⬆ back to top](#table-of-contents)**

### Reducer Action & State

```javascript
const reducer = (state, action) => {
  console.log({ action });
  return state;
};

const Application = () => {
  const [grudges, dispatch] = useReducer(reducer, initialState);
}
```

**[⬆ back to top](#table-of-contents)**

### Reducer Action Keys & dispatch

[Flux Standard Action](https://github.com/redux-utilities/flux-standard-action)

```javascript
const GRUDGE_ADD = 'GRUDGE_ADD';
const GRUDGE_FORGIVE = 'GRUDGE_FORGIVE';

const reducer = (state, action) => {
  if(action.type === GRUDGE_ADD) {
    return [action.payload, ...state];
  }

  return state;
};

const Application = () => {
  const [grudges, dispatch] = useReducer(reducer, initialState);

  const addGrudge = ({ person, reason}) => {
    dispatch({
      type: GRUDGE_ADD,
      payload: {
        person,
        reason,
        forgiven: false,
        id: id()
      }
    });
  };

  return ( ... );
};
```

**[⬆ back to top](#table-of-contents)**

### Action & State Modification Exercise

- Be a better person than me.
- I’ve implemented the ability to add a grudge. 
- Can you implement the ability to forgive one?

**[⬆ back to top](#table-of-contents)**

### Action & State Modification Solution

```javascript
const reducer = (state, action) => {
  if(action.type === GRUDGE_FORGIVE) {
    return state.map(grudge => {
      if (grudge.id !== action.payload.id) return grudge;
      return { ...grudge, forgiven: !grudge.forgiven };
    })
  }
  return state;
};

const Application = () => {
  const [grudges, dispatch] = useReducer(reducer, initialState);

  const toggleForgiveness = id => {
    dispatch({
      type: GRUDGE_FORGIVE,
      payload: { id }
    });
  };

  return ( ... );
};
```

**[⬆ back to top](#table-of-contents)**

### [React.memo](https://reactjs.org/docs/react-api.html#reactmemo) & [useCallback](https://reactjs.org/docs/hooks-reference.html#usecallback)

Memoization

- React.memo - render when prop changes
- useCallback - give a new function when prop changes
- useMemo - execute function when prop changes
- Wrap the action creators in useCallback
- Wrap NewGrudge and Grudge in React.memo
- Notice how we can reduce re-renders

```javascript
  const addGrudge = useCallback(
    ({ person, reason}) => {
      dispatch({
        type: GRUDGE_ADD,
        payload: {
          person,
          reason,
          forgiven: false,
          id: id()
        }
      });
    },
    [dispatch]
  );

  const toggleForgiveness = useCallback(
    id => {
      dispatch({
        type: GRUDGE_FORGIVE,
        payload: { id }
      });
    },
    [dispatch]
  );
```

```javascript
const NewGrudge = memo(({ onSubmit }) => {
  return ( ...);
});

const Grudge = memo(({ grudge, onForgive }) => {  
  return ( ... );
});
```

**[⬆ back to top](#table-of-contents)**

## **05. Context**

### Prop Drilling & Context API

What is [prop drilling](https://kentcdodds.com/blog/prop-drilling)?

- Prop drilling (also called "threading") refers to the process you have to go through to get data to parts of the React Component tree.

What problems can prop drilling cause?

- Refactor the shape of some data
- Over-forwarding props
- Under-forwarding props
- Renaming props

**[⬆ back to top](#table-of-contents)**

### Creating a Context Provider

[Context](https://reactjs.org/docs/context.html) provides a way to pass data 
through the component tree without having 
to pass props down manually at every level.

- createContext() -> Provider
- createContext() -> Consumer

```javascript
import React from 'react';
const SuperCoolContext = React.createContext();
SuperCoolContext.Provider;
SuperCoolContext.Consumer;

<CountContext.Provider value={0}>
  <CountContext.Consumer>
    { value => <p>{value}</p> }
  </CountContext.Consumer>
</CountContext.Provider>
```

We're going to rip a lot out of Application.js and move it to a new file called GrudgeContext.js and it's going to look something like this.

```javascript
import React, { useReducer, createContext, useCallback } from 'react';
import initialState from './initialState';
import id from 'uuid/v4';

export const GrudgeContext = createContext();

const GRUDGE_ADD = 'GRUDGE_ADD';
const GRUDGE_FORGIVE = 'GRUDGE_FORGIVE';

const reducer = (state = [], action) => {
  if (action.type === GRUDGE_ADD) {
    return [
      {
        id: id(),
        ...action.payload
      },
      ...state
    ];
  }

  if (action.type === GRUDGE_FORGIVE) {
    return state.map(grudge => {
      if (grudge.id === action.payload.id) {
        return { ...grudge, forgiven: !grudge.forgiven };
      }
      return grudge;
    });
  }

  return state;
};

export const GrudgeProvider = ({ children }) => {
  const [grudges, dispatch] = useReducer(reducer, initialState);

  const addGrudge = useCallback(
    ({ person, reason }) => {
      dispatch({
        type: GRUDGE_ADD,
        payload: {
          person,
          reason
        }
      });
    },
    [dispatch]
  );

  const toggleForgiveness = useCallback(
    id => {
      dispatch({
        type: GRUDGE_FORGIVE,
        payload: {
          id
        }
      });
    },
    [dispatch]
  );

  return (
    <GrudgeContext.Provider value={{ grudges, addGrudge, toggleForgiveness }}>
      {children}
    </GrudgeContext.Provider>
  );
};
```

Now, Application.js looks a lot more slimmed down.

```javascript
import React from 'react';

import Grudges from './Grudges';
import NewGrudge from './NewGrudge';

const Application = () => {
  return (
    <div className="Application">
      <NewGrudge />
      <Grudges />
    </div>
  );
};

export default Application;
```

Wrapping the Application in Your New Provider

```javascript
ReactDOM.render(
  <GrudgeProvider>
    <Application />
  </GrudgeProvider>,
  rootElement
);
```

**[⬆ back to top](#table-of-contents)**

### Context & useContext Hook

```javascript
import React, { useContext } from 'react';
import Grudge from './Grudge';
import { GrudgeContext } from './GrudgeContext';

const Grudges = () => {
  const { grudges } = useContext(GrudgeContext);

  return (
    <section className="Grudges">
      <h2>Grudges ({grudges.length})</h2>
      {grudges.map(grudge => (
        <Grudge key={grudge.id} grudge={grudge} />
      ))}
    </section>
  );
};

export default Grudges;
```

```javascript
import React, { memo, useContext } from 'react';
import { GrudgeContext } from './GrudgeContext';

const Grudge = ({ grudge }) => {
  const { toggleForgiveness } = useContext(GrudgeContext);

  return (
    <article className="Grudge">
      <h3>{grudge.person}</h3>
      <p>{grudge.reason}</p>
      <div className="Grudge-controls">
        <label className="Grudge-forgiven">
          <input 
            type="checkbox" 
            checked={grudge.forgiven} 
            onChange={() => toggleForgiveness(grudge.id)}
          />{' '}
          Forgiven
        </label>
      </div>
    </article>
  );
};

export default Grudge;
```

**[⬆ back to top](#table-of-contents)**

### Context Practice

```javascript
import React, { useState, memo, useContext } from 'react';
import { GrudgeContext } from './GrudgeContext';

const NewGrudge = () => {
  const [person, setPerson] = useState('');
  const [reason, setReason] = useState('');

  const { addGrudge } = useContext(GrudgeContext);

  const handleChange = event => {
    event.preventDefault();
    addGrudge({ person, reason });
  };

  return ( ... );
};

export default NewGrudge;
```

Some Tasting Notes

- We lost all of our performance optimizations when moving to the Context API.
- What’s the right answer? It’s a trade off.
- Grudge List might seem like a toy application, but it could also represent a smaller part of a larger system.
- Could you use the Context API to get things all of the way down to this level and then use the approach we had previously?

**[⬆ back to top](#table-of-contents)**

## **06. Data Fetching**
**[⬆ back to top](#table-of-contents)**

## **07. Thunks**
**[⬆ back to top](#table-of-contents)**