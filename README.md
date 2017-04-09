# Functional Programming
## 12 April 2017
## SydJS(J)

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

### Example One

We want to iterate over a List of values and apply a specific set of instructions to mutate the values in that list.

- The values in the List are Integers
- Add one to each value
- Times each value by 10
- Divide each value by 2

1. First attempt - imperative

```js
let myList = [1,2,3,4,5];

for (let i = 0; i < 5; i = i + 1)
  myList[i] = ((myList[i] + 1) * 10)/2;
```

2. Second Attempt - use map

```js
let myList = [1,2,3,4,5];

myList.map(function(x) {
 return ((x + 1) * 10)/2
});
```

3. Third Attempt - use ES6

```js
let myList = [1,2,3,4,5];

myList.map(x =>
 ((x + 1) * 10)/2
);
```

4. Fourth Attempt - use a function and stop mutating myList

```js
const myList = [1,2,3,4,5];

const mapAddOneTimesTenDivideByTwo = list =>
  list.map(x =>
    ((x + 1) * 10)/2
  )
  
mapAddOneTimesTenDivideByTwo(myList)
```

5. Fifth attempt - lets get functional

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

###Ramda.JS

####http://ramdajs.com/docs/#compose

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

###Pure Functions

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

We call this a "Pure Function" because:
1. It is only dependent upon the input provided, and is completely self contained.
2. It does not change anything outside of its internal scope.

It follows the principles of both "immutability" and "no side effects".

In functional programming we want to avoid both mutating state and side effects by using pure functions.

Pure functions are easily tested and extensible. By using composition we also increase their modularity and make them easier to reason about. We have decomposed the complexity by creating re-usable modular functions that are composed together to do an incrementally more complex task.













