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






