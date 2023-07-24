Redux is a state management tool, which is commonly used with Redux, but can be used with other frameworks like angular, ember etc for state management tasks. Anywhere we write JS, redux can be used there (backend frontend everything).

Redux has 5 major functions and 4 out of them are more or less just utility helper functions. 

First of all we will try to learn `Just redux` without any intervention of React or anything else. We will later plug it in the react application. 

You might be thinking, why learn redux now ? Isn't React getting support of things like useReducer ? There are a lot of features that useReducer lack and you can get that from Redux api. Also the react-redux api (which helps us to integrate redux to react), internally depends on context api.


### combine

Lets start with one very simple function that redux provides us i.e. `combine`. Let's say we have a string,
which needs to go through multiple transformations, how would you achieve that? Simple, go one by one on each transformation and pass the result of one as an argument to the other one. Let's see this with an example:

Let's go through 4 functions expressions: 
Function Exp 1: Here we split the given string on the basis of a space charater and then join them based on empty character. 
```
const removeSpaces = (string) => string.split(" ").join("");
```
Function Exp 2: Here we do string repitition twice.
```
const repeatString = (string) => string.repeat(2);
```
Function Exp 3: Here we convert the incoming string to all uppercase
```
const makeUpperCase = (string) => string.toUpperCase();
```
Function Exp 4: Here we wrap the string in <i></i> tags.
```
const makeItalics = (string) => string.italics();
```

Now what if we want to first split the string and join it, then make a repition, convert everything to uppercase and then wrapping in to `<i>` tags. 

So how the code will look like is 
```
console.log(makeItalics(makeUpperCase(repeatString(removeSpaces(input)))));
```

You can see this expression looks like f(g(h(x))). One more way could have been to create a map on an array of functions and then achieve the same task. 


Now what combine will do is the same thing for us in an elegant fashion.

```
console.log(compose(makeItalics, makeUpperCase, repeatString, removeSpaces));
```

So what is does is that it takes a bunch of functions and compose them into one chain of functions. 

## createStore

In order to understand the createStore function, we need to understand how `Reducers` work which is an argument to the createStore function (there are other non mandatory arguments also).

Reducers are simple function, which takes 2 arguments
 - State of the world which is a JS object. 
 - And something that just happened or some event which is a JS object.

Reducers return one output
 - State of the world which is a JS object. (there can a situation when nothing changed in the state)

Now `createStore` provides 4 more functions as a result that can be a real deal. educer

```
const initialState = {value: 0};

const reducer = (state, actions) => { // this reducer actually doesnt do anything
    return state;
}

const store = createStore(reducer, initialState);
console.log(store); // doing this you can see 4 functions inside the store
```

When you print the store, it shows 4 functions
 - replaceReducer -> helps us to replace the reducer function, helps us in code splitting. Because just one function might not help the codebase to scale, hence this can help to later split the code, and swap the current reducer with another one (which we can create by combine reducer to combine multiple small reducers)
 - getState -> helps to determine the current state
 - dispatch -> triggers some action in the reducer
 - subscribe -> subscribes to state changes in the store

### Now what is an action ?
In reducer pattern we dont just call the methods on the store object, and it changes stuff. How does the store knows that here comes an action, go to the corresponding reducer and change the state acoordingly. We use the dispatch method for this. 

Every action to be registered in the reducer needs to have one mandatory requirement i.e. `type`. You might see people passing `payload` as well to pass the values to the action.

```
const incrementAction = {type: 'INCREMENT', payload: 5}
```

We use this type for putting conditionals in the reducer function.

```
const reducer = (state, action) => {
    if(action.type == 'INCREMENT') {
        return {value: state.value + action.payload};
    }

    return state;
}
```

Sometimes passing this string can lead to spelling errors, so we can store it in a variable.


A lot of times we can create a function also referred as ActionCreater that can return the action object for us. 

```
const increment = (amount) => ({type: 'INCREMENT', payload: amount});

store.dispatch(increment());

console.log(store.getState());

store.subscribe(() => console.log("SUBSCRIBER", store.getState()));
```

Whats the benefit ? The same as of functions, if we write plain objects then we have to find them everywhere we used them in case of bugs and updates, but using a function it is encapsulated in one place. 

### Good practice for reducers
 - No mutating objects, if you touch it, you replace it.
 - These are just JS functions.
 - You have to return something and ideally, it should be the unchanged state if there is nothing u need to do. 
 - Prefer flat objects, in case you APIs return you a nested object, then write a translator on your UI layer and then update the flatter object. Also use spread operator more often for updating objects.

Note: Never make multiple store objects, its an anti pattern. If your state goes in multiple stores, can clash things. You can have one store and multiple reducer. 


## bindActionCreator
So to dispath an action we have to always call `store.dispatch(action())` again and again. What if we want to bind functions to the dispatch call so we dont have to do store.dispatch again and again. 
```
const actions = bindActionCreator({increment, anyOtherfunction}, store.dispatch);
// now we can directly do
actions.increment(100);
```
The first argument of the bindActionCreator method is an object containing methods that we want to bind and the second argument is the store function we want to bind to. 

we can make a custom bindActionCreator function as well using `compose`
```
const [dispatchIncrement, dispatchAnyOtherFunction] = [increment, anyOtherfunction]
.map(fn => compose(store.dispatch, fn));
```


## combineReducer

What if we have a complicated state object, which is difficult to update in the reducers, then we can do the following:
- Try to keep the state as flat as possible, the UI can be nested but keeping the state simple can help.
- Distribute the reducer in to small multiple reducers.

Lets take an example:
```
const initialState = {
    users: {
        {id: 1, name:"sanket"},
        {id: 2. name: "Sarthak"}
    },
    tasks: {
        {title: "todo 1"},
        {title: "todo 2"}
    }
}

const ADD_USER = "ADD_USER";
const ADD_TASK = "ADD_TASK";

const addTask = (title) => ({type: ADD_TASK, payload: {title}});
const addUser = (name) => ({type: ADD_USER, payload: {name}});

const userReducer = (users = initialState.users, action) => {
    if(action.type == ADD_USER) {
        return [...users, action.payload]
    }
    return users;
}

const taskReducer = (tasks = initialState.tasks, action) => {
    if(action.type == ADD_TASKS) {
        return [...tasks, action.payload]
    }
    return tasks;
}

const reducer = combineReducer({users: userReducer, tasks: taskReducer})
const store = createStore(reducer);
```
Note: all actions are gone through every reducer,(try this with console log)

### enhancers

Before we create the store object, we can use enhancers toapply anything on it, it can be passed as an argument to create store in the form of a function.

```
const reducer = (state) => state;
const monitorEnhancer = (createStore) => (reducer, initialState, enhance) => {
const monitoredReducer = (state, action) => {
const start = performance. now () ;
const newState = reducer (state, action);
const end = performance. now () ;
const diff = end - start;
console.log (diff);
return newState;
};
return createStore (monitoredReducer, initialState, enhancer);
}
const store = createStore (reducer, monitorEnhancer);
store.dispatch ({ type: "Hello" });
```


-- middlewares pending: https://redux.js.org/api/applymiddleware
