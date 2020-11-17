<h1 align="center">
Splitting-and-Combining-Reducers
</h1>

You've been using a single reducer to manage state in your redux store.

As your app grows, it's become necessary to use multiple reducers, each managing a slice of state.

<h1 align="center">
Splitting reducers
</h1>

Imagine that your fruit stand is extremely successful and it grows so much that you need multiple farmers helping you to keep your stand stocked with fruit.

Your app's state will need to grow to store not only and array of fruit but also a farmers object that keeps track of your farmers.

here's a sample state tree of your updated applications:

```js
    {
        fruit: [
            'APPLE',
            'APPLE',
            'ORANGE',
            'GRAPEFRUIT',
            'WATERMELON',
        ],
        farmers: {
            1: {
            id: 1,
            name: 'John Smith',
            paid: false,
            },
            2: {
            id: 2,
            name: 'Sally Jones',
            paid: false,
            },
        }
    }
```

The store now needs to handle new action types like 'HIRE_FARMER' and 'PAY_FARMER' by updating the farmers slice of state.

You could add more cases to your exisitng reducer, but eventually the existing reducer would become to large and difficult to manage.

The solution is to split the reducer into seperate fruit and farmers reducers.

Splitting up the reducer into multiple reducers handling separate, independent slices of state is called reducer compostion, and it's the fundamental pattern of building redux apps.

Bacause each reducer only handles a single slice of state, its state parameter corresponds only to the part of the state that manages and it only repsonds to actions that concern that slice of state.

Split up ypur popular fruit stand app's reducer into two reducers

    * fruitReducer - a reducing function that handle actions updating the fruits slice of state
    * farmersReducer - a reducing function that handles actions updating the new farmers slice of state

/src/reducers/fruitReducer.js

```js
    import {
        ADD_FRUIT,
        ADD_FRUITS,
        SELL_FRUIT,
        SELL_OUT,
        } from '../actions/fruitActions';

        const fruitReducer = (state = [], action) => {
        switch (action.type) {
            case ADD_FRUIT:
            return [...state, action.fruit];
            case ADD_FRUITS:
            return [...state, ...action.fruits];
            case SELL_FRUIT:
            const index = state.indexOf(action.fruit);
            if (index) !== -1) {
                // remove first instance of action.fruit
                return [...state.slice(0, index), ...state.slice(index + 1)];
            }
            return state; // if action.fruit is not in state, return previous state
            case SELL_OUT:
            return [];
            default:
            return state;
        }
};

export default fruitReducer;
```

/src/reducers/farmersReducers.js

```js

import { HIRE_FARMER, PAY_FARMER } from '../actions/farmersActions';

const farmersReducer = (state = {}, action) => {
        let nextState = Object.assign({}, state);
        switch (action.type) {
            case HIRE_FARMER:
            const farmerToHire = {
                id: action.id,
                name: action.name,
                paid: false
            };
            nextState[action.id] = farmerToHire;
            return nextState;
            case PAY_FARMER:
            const farmerToPay = nextState[action.id];
            farmerToPay.paid = !farmerToPay.paid;
            return nextState;
            default:
            return state;
        }
};

export default farmersReducer;
```

You also need to define a module containing the 'HIRE_FARMER' and 'PAY_FARMER' actions:

./src/actions/farmersActions.js

```js

    export const HIRE_FARMER = 'HIRE_FARMER';
    export const PAY_FARMER = 'PAY_FARMER';

    export const hireFarmer = (name) => ({
        type: HIRE_FARMER,
        id: new Date().getTime(),
        name,
    });

    export const payFarmer = (id) => ({
        type: PAY_FARMER,
        id,
    });
```

<h1 align="center">
Combining reducers
</h1>

Your reducer setup is now much more modular.

However, createStore only takes one reducer arguement, so you must combine your reducers back into a single reducer to pass to the store.

To do this you'll use the combineReducers method from the redux package and pass it an object that maps state keys to the reducers that handle those slices of state.

Below, the combineREducers maps the fruitReducer for the fruit slice of state and the farmerReducer for the farmers slice of state.

Invoking the combineReducers function returns a single rootReducer that you can use to create your redux store.

// ./src/reducers/rootReducer.js

```js
    import { combineReducers } from 'redux';
    import fruitReducer from './fruitReducer';
    import farmersReducer from './farmersReducer';

    const rootReducer = combineReducers({
        fruit: fruitReducer,
        farmers: farmersReducer
    });

    export default rootReducer;
```

```js
    import { createStore } from 'redux';
    import rootReducer from './reducers/rootReducer';

    const store = createStore(rootReducer);

    export default store;
```

<h1 align="center">
Delegating to reducers
</h1>

Another aspect of reducer compostion involves delegating state updates to suboridnate reducers.

Consider the farmers reducer.

You can modify is so that the farmers reducer delegates to a farmer reducer whenever a single farmers's attributes need to be modified

// ./src/reducers/farmersReducer.js

```js

import { HIRE_FARMER, PAY_FARMER } from '../actions/farmersActions';

    const farmerReducer = (state, action) => {
        // State is a farmer object.
        switch (action.type) {
            case HIRE_FARMER:
            return {
                id: action.id,
                name: action.name,
                paid: false
            };
            case PAY_FARMER:
            return Object.assign({}, state, {
                paid: !state.paid
            });
            default:
            return state;
        }
    };

    const farmersReducer = (state = {}, action) => {
        let nextState = Object.assign({}, state);
        switch (action.type) {
            case HIRE_FARMER:
                nextState[action.id] = farmerReducer(undefined, action);
            return nextState;
            case PAY_FARMER:
                nextState[action.id] = farmerReducer(nextState[action.id], action);
            return nextState;
            default:
            return state;
        }
    };

export default farmersReducer;
```

<h1 align="center">
Catching and preventing state mutation bugs
</h1>

Updating the farmersReducer to delegate from farmer state updates to the farmerReducer resolved a suble state mutation big.

Take a look at the original implementation of the farmersREducer function:

```js
    const farmersReducer = (state = {}, action) => {
        let nextState = Object.assign({}, state);
        switch (action.type) {
            case HIRE_FARMER:
            const farmerToHire = {
                id: action.id,
                name: action.name,
                paid: false
            };
            nextState[action.id] = farmerToHire;
            return nextState;
            case PAY_FARMER:
            const farmerToPay = nextState[action.id];
            farmerToPay.paid = !farmerToPay.paid;
            return nextState;
            default:
            return state;
        }
    };
```

Notice that the state parameter is duplicated to the nextState variable using the Object.assign method:

```js
    let nextState = Object.assign({}, state);
```

While this code correctly creates a duplicate of the state object, nextState is only a shallow duplicate as only the top-level object is duplicated.

Each "farmer" object that the state object refers to are still the same objects.

In the PAY_FARMER case clause, the farmer object is mutated by setting the paid property to a new value:

```js
    case PAY_FARMER:
    const farmerToPay = nextState[action.id];
    farmerToPay.paid = !farmerToPay.paid;
    return nextState;
```

Now look again at the Pay_FARMER case clause in the version of the farmersReducer that delegates farmer state updates to the farmerReducer:

```js
    case PAY_FARMER:
        nextState[action.id] = farmerReducer(nextState[action.id], action);
        return nextState;
```

This code calls the farmerReducer by passing in the farmer object for the action.id property value and the action parameter. The farmerReducer has a PAY_FARMER case clause that correctly used the Object.assign method to duplicate the farmer object with the new paid property value.

```js
    const farmerReducer = (state, action) => {
  // State is a farmer object.
        switch (action.type) {
            case HIRE_FARMER:
            return {
                id: action.id,
                name: action.name,
                paid: false
            };
            case PAY_FARMER:
            return Object.assign({}, state, {
                paid: !state.paid
            });
            default:
            return state;
        }
        };
```

Catching state mutation bugs is difficult to do.

Leveraging patterns like reducer compostion can help you from introducing these kinds of bugs int the first place.











