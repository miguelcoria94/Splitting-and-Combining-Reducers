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




