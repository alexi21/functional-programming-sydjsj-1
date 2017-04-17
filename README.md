# Functional Programming
### 12 April 2017
### SydJS(J)

---

### Thinking Functionally

#### Extensible
Minimise the need to refactor code when adding additional functionality.

#### Modular
Make changes to the code base without producing unwanted and/or untraceable effects in other parts of the code. 

#### Re-usable
DRY code that has little duplication and re-uses patterns across the code base.

#### Testable
The code is written in such a way that makes it is to unit test the functions throughout the code base.

#### Reasonable
The code is well structured and easy to reason about.

---

### Why functional programming is important

1. Many of the major programming languages have either begun to incorporate functional programming techniques into their API, or have been designed explicitly with functional programming in mind.

2. Javascript is arguably more effective and easier to reason about, particularly as the complexity of the code base increases, when programmed using a functional style.

3. Writing Javascript in a functional style not only makes it easier to reason about, and more efficient, it also provides a better understanding of the Javascript language.

---

### What is functional programming?

Functional programming is, at its most basic, a way of thinking that places an emphasis on using functions. "It's not a matter of just applying functions to come up with a result; the goal is to abstract control flows and operations on data with functions in order to avoid side effects and reduce mutation of state in your application."

Functional programming is about decomposing your programs into tiny re-usable functions that can be composed together to provide the specific functionality needed. By decomposing a problem into pieces we create small units of functionality that are testable and re-usable throughout the code base. Composition allows us to abstract from the individual solutions for each component part, making solutions built from composition easier to reason about and readily extensible throughout the code base.

##### Extensible
##### Modular
##### Re-usable
##### Testable
##### Reasonable

---
#### Array

##### Example One

We want to iterate over a List of values and apply a specific set of instructions to mutate the values in that list.

- The values in the List are Integers
- Add one to each value
- Times each value by 10
- Divide each value by 2

1. First attempt 

#### Imperative

This is one way of using the imperative style of programming to solve the problem described above. We explicitly provide a set of specific instructions and mutate the variable `myList`

```js
// [Int]

let myList = [1,2,3,4,5];

for (let i = 0; i < 5; i = i + 1)
  myList[i] = ((myList[i] + 1) * 10)/2;
```

2. Second Attempt

#### map 

Map and Reduce - and the many other related functions such as Filter and Find, provide the necessary abstraction to start using functional programming. We also start using immutability here by declaring a new variable

```js
const myList = [1,2,3,4,5];

const myNewList = myList.map(function(x) {
 return ((x + 1) * 10)/2;
});
```

3. Third Attempt

#### ES6

Using ES6 lambda expressions we can greatly simplify the syntax. As we will see they also provide a great deal of flexibility as we begin to code in a more functional manner.

```js
const myList = [1,2,3,4,5];

const myNewList = myList.map(x =>
 ((x + 1) * 10)/2
);
```

4. Fourth Attempt

#### Wrap in a function

```js
const myList = [1,2,3,4,5];

const mapAddOneTimesTenDivideByTwo = list =>
  list.map(x =>
    ((x + 1) * 10)/2
  )
  
mapAddOneTimesTenDivideByTwo(myList)
```

5. Fifth attempt

#### Functional

```js
const addOne = x =>
  x + 1
  
const timesTen = x =>
  x * 10
  
const divideByTwo = x =>
  x / 2
  
const mapAddOneTimesTenDivideByTwo = list =>
  list.map(x =>
    divideByTwo(
      timesTen(
        addOne(x)
      )
    )
  )
  
const myList = [1,2,3,4,5];
 
mapAddOneTimesTenDivideByTwo(myList) 
```
Okay, so now we have four functions. The first three are simple and modular functions that do a basic arithmetic evaluation. The last is a little ugly, but nonetheless does the job. It maps over the list and composes each operation in turn to the inputted value and returns a new array.

What I hope is that by illustrating the transformation

But we are missing a key ingredient here: composition

---

### Composition

Composition allows us to, well compose functions together in a neat way that is easier to read.

Just taking the inner function of the map, essentially what we want to do is:
1. take a value
2. Add one
3. Then times the result by ten
4. Then divide the result of that by two

`x -> x + 1 -> y * 10 -> z / 2 -> result`

Composition is generally done from the bottom up, so it looks like this

`result <- z / 2 <- y * 10 <- x + 1 <- start`

So, using the previously declared functions

```js
  compose(
    divideByTwo,
    timesTen,
    addOne
  )
```

Which can be thought of as precisely the same as

```js
 divideByTwo(
   timesTen(
    addOne(x)
   )
 )
```

Except we have lost the `x` ???

What we have here is a function waiting for a value to be inputted.

```js
const addOneTimesTenDivideByTwo = x =>
    compose(
      divideByTwo,
      timesTen,
      addOne
    )(x)
```

So what is this compose, and where did it come from. Essentially you can think of compose as syntactic sugar that returns the result from the functions, starting from the bottom, and inputs them into the next function up.

So in the function `addOneTimesTenDivideByTwo` we take our inputted variable `x` and run `addOne(x)` and take the result of that and pass it to `timesTen` like so `timesTen(addOne(x))`, and so on.

But where do I find such a useful function ?

---

### Ramda.JS

#### http://ramdajs.com/docs/#compose

So now we have a concrete utility function that we can use to compose our functions.

```js
import {compose} from 'ramda'

const addOneTimesTenDivideByTwo = x =>
    compose(
      divideByTwo,
      timesTen,
      addOne
    )(x)
```

But compose will automatically pass the last inputted variable to the function and then pass it on to the first function (the bottom function). So we can actually right it like this.

```js
import {compose} from 'ramda'

const addOneTimesTenDivideByTwo =
    compose(
      divideByTwo,
      timesTen,
      addOne
    )
```

So now lets return to the list.

```js
const mapAddOneTimesTenDivideByTwo = list =>
  list.map(
    compose(
      divideByTwo,
      timesTen,
      addOne
    )
  )
```

Or even better using Ramda's `map` function, which is a curried version of the vanilla Javascript `map` method. It takes two variables, the first is a function, the second is an iterable.

Essentially Ramda `map` is

```js
const map = fn => list => list.map(fn)
```

Which allows us two write this "Point Free" version

```js
import {compose, map} from 'ramda'

const mapAddOneTimesTenDivideByTwo =
  map(
    compose(
      divideByTwo,
      timesTen,
      addOne
    )
  )
```

Which produces exactly the same result in the end as

```js
const mapAddOneTimesTenDivideByTwo =
    compose(
      map(divideByTwo),
      map(timesTen),
      map(addOne)
    )
```

---

### Currying functions

Okay lets take a break from composition and talk about currying.

ES6 provides currying out of the box, so I will give you examples using ES6.

Say we want to write a function that takes any two values and adds them together.

```js
const add = (x, y) => x + y
```

But we want to create the `addOne` function we were using before out of this function. Because we are functional programmers we would write `add` like this.

```js
const add = x => y => x + y
```

Now we can write

```js
const addOne = add(1)
```

And we can do exactly the same with our other two functions

```js
const add = x => y => y + x

const multiply = x => y => y * x

const divide = x => y => y / x

const addOne = add(1)

const multiplyByTen = multiply(10)

const divideByTwo = divide(2)
```

But we want our `map` function to be even more extensible. So now we can write it like this.

```js
import {compose, map} from 'ramda'

const addTimesDivide = (x, y, z) =>
    compose(
      divide(z),
      multiply(y),
      add(x)
    )
```

Which means we could then do this

```js
const mapAddOneTimesTenDivideByTwo = 
  map(
    addTimesDivide(1, 10, 2)
  )
```

Or any number of different variations on this theme.

---

### Pure Functions

Lets return to this function

```js
import {compose, map} from 'ramda'

const mapAddOneTimesTenDivideByTwo =
  map(
    compose(
      divideByTwo,
      timesTen,
      addOne
    )
  )
```

#### What are Pure Functions?

- Only dependent upon the variables provided.
- Do not change anything.
- Follows the principles of both "immutability" and "no side effects".
- Immutability: arguments passed in are not mutated, and declared variables are not mutated.
- No side effects: function produces no effects other than the returned value.
- https://wiki.haskell.org/Referential_transparency “an expression always evaluates to the same result in any context.”

In functional programming we want to avoid both mutating state and side effects by using pure functions.


In functional programming we want to avoid both mutating state and side effects by using pure functions.

Pure functions are easily tested and extensible. By using composition we also increase their modularity and make them easier to reason about. We have decomposed the complexity by creating re-usable modular functions that are composed together to do an incrementally more complex task.

---

## Functor

There is something else that is rather special about this example. The array we have been using in Javascript is also known as a Functor in category theory. A functor is type class that can be mapped. In other words it is a container or wrapper which can be temporarily unpacked and have a function (or morphism) applied to each individual item in the container.

One of the reasons that arrays, or more generally lists, are so useful in programming is for this reason.

Now rather than embark on a lesson in category theory which is way beyond the scope of this discussion, lets have a quick look at an incredibly useful Functor that we use at EventHub.

---

### Maybe

A maybe is a functor, and a very special functor that has a number of useful properties.

Essentially, a maybe is like a container into which you place something and then apply your functions to it while inside the container, before reaching in and pulling whatever the transformed thing is out. Well almost, because you maybe can take it out of the maybe, or maybe you can't.

So a Maybe has the following type signature

(Just A | Nothing)

Like the array, which wraps a collection. In general terms a maybe is the same thing, you wrap the value in a maybe, and then using a special type of map operation called an fmap you reach in and apply the provided function to the value inside the Maybe wrapper.

You then resolve the Maybe by getting the value out, if it is still there, or you get Nothing.

Lets look at a concrete example that will explain it far better than I can here.

---

### Maybe

#### Example 2.

Say you want to fetch something from your database and then convert it to a JSON, grab a value from it, and then serve it up to the frontend for further manipulation. Using a maybe makes this potentially hazardous journey into the unknown far safer.

So we are going to take the following steps

1. Fetch our user from the database using a given `uuid`
2. Convert the model to a JSON
3. Grab the `username` prop from the newly created object

---

#### Maybe to the rescue

The Maybe data type provides an elegant and simple solution to this problem. We simply wrap our database inside a Maybe and the get the returned value or `null`

We are going to use the Maybe monad (don't worry it is still a Functor and I am not going to mention monads again tonight) provided by folktale on github.

https://github.com/folktale/data.maybe

So lets say you have your database model `User` and your `UUID` and want to find your user.

In SQL you might do something like this:
```sql
SELECT * FROM users WHERE id = '289df31a-96c1-4418-b315-7d2e5c2dfc6a'
```

In Javascript you are probably using an ORM or Query Builder (such as BookshelfJS and KnexJS)

So you could wrap your query in a function (reusable and modular) and it might look something like this:

```js
import User from 'my-user-model'

const userFetchById = id =>
  User.where({id}).fetch()
  
userFetchById('289df31a-96c1-4418-b315-7d2e5c2dfc6a')
```

Now if all goes well the fetched user is converted to a JSON and we grab the username.

But if something goes wrong (such as the user not being there) - BOOM!

So lets wrap our function in a `Maybe`

---

Complete example

```js

// Using some syntactic sugar below, feel free to ignore

const then = f => p => p.then(f)

const getOrElse = maybe => orElseValue => maybe.getOrElse(null)

// ------------------------------------------------------------

const getUserNameFromId = id =>
    compose( 
      then(compose(
          getOrElse('no user found'),
          map(myMaybe => 
            toJSON(myMaybe).user_name
          ),
          Maybe.fromNullable
        )),
      userFetchById
    )(id)
```

Lets explain this and then add a little more syntactic sugar

First we fetch the user from the database...

BUT

It might be null in which case if we call `toJSON` we will get an error.

So we wrap it in a maybe which protects the outer code from any inner explosions.

We use a constructor function.

`Maybe.fromNullable(valueWhichMightBeNull)`

Then ( literally because we have a promise here ) we open up the Maybe and act as if it is not null fetched but rather a user model.

Convert the model to a JSON and grab the prop.

Then we get the value from the maybe or we get the string `'no user found'`

Lovely!

---

Now if we use some more Ramda and syntactic sugar we can actually write this function like this.

```js
import R from 'ramda'

// Using some syntactic sugar below, feel free to ignore

const then = f => p => p.then(f)

const getOrElse = maybe => orElseValue => maybe.getOrElse(null)

const toJSON = R.invoker(0, 'toJSON')

// ------------------------------------------------------------

const getUserNameFromId =
    R.compose( 
      then(R.compose(
          getOrElse('no user found'),
          R.map(compose(
            R.prop('user_name'),
            toJSON
          )),
          Maybe.fromNullable
        )),
      userFetchById
    )
```

`R.prop` simply gets the value at the prop passed in as a string

`R.invoker` calls the provided method, in this case toJSON, on the value which is passed in.

The other three functions enable us to go completely point free, which is functional nirvana :)

As you can see we are using currying and point free liberally already even in this example. But this is not something to be intimidated by, once you begin to THINK functionally the actually data itself becomes less of a concern, rather you begin to think in terms of the data structure or Type signature. And Javascript becomes a powerful tool, capable of running across the full stack.

The point of this example is not to teach you functional programming, or really anything about `Maybe`. It is to demonstrate the power of functional programming. Which gets even more interesting when you consider that we may want to do this operation again and again but with different models and with different data manipulations.

In which case we can break this function up into another utility function.

```js

const getFromFunctionOrElseValue = fn => value =>
    then(R.compose(
      getOrElse(value),
      map(fn),
      Maybe.fromNullable
    ))

```

So we just declare our function to manipulate the returned model and pass that in with the value we want if the Maybe takes a null when the user is fetched from the database.

```js
const getUserName =
  R.compose(
    R.prop('user_name'),
    toJSON
  )
  
const getUserNameFromId =
    R.compose( 
      getFromFunctionOrElseValue(getUserName)('no user found'),
      userFetchById
    )
```

The motto is compose and DRY the code as much as possible, and never mutate.


