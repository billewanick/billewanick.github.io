# A case for ES6 curry functions

## tl;dr

Currying and partial application in JavaScript is useful to make small, simple functions you can compose together for complex (but maintainable) logic.  
eg:

```javascript
    const add = x => y => x + y
    add(5)(2) === 7
    const add2 = add(2)  
    [1,2,3,4,5].map(add2) === [3,4,5,6,7]
```

## Explanation

So with the new ES6 additions JavaScript got the fat arrow operator. This is JavaScript gaining more and more Functional Programming ideas into the language. I also find it interesting to look at this through the lens of language design and trying to find out what the language designers were going for.  
<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions>  
(Side note: there is a meaningful distinction between statement and expression. I used to think they were interchangeable, but they aren't. Expressions take their inputs and return an output. Statements don't necessarily return anything, and are more used for side effects. Expressions are declarative, statements are imperative)

Arrow functions don't have a `this` keyword in them, and are best used for stateless computation. Let's take a closer look at the code above.  
`const add = x => y => x + y`  
You could also write this as:  
`const add = (x) => { return (y) => { return x + y}}`  
Parenthesis are optional for single arguments, and curly braces and return statements are optional if there is only one expression. So the language designers are pushing us towards having single argument functions with single return expressions.

But how do we do multiple argument functions!?

The arrow operator returns a function as its return value. In the add example, add is a function that takes a value (x), and returns a function. This next function takes a value (y), and returns the sum of x and y. The first arrow closes x over the rest of the function (ie binds it), so it can be referenced in the next function.

If I have a big expression I need to evaluate, I can keep adding arguments as arrows.  
eg. `const stuff => valA => valB => exprC => exprD => valA + valB + exprC + exprD`  
This looks and acts like Haskell, every arrow returning a function that takes more arguments until it gets to the bottom of the chain.

The big difference here is how the functions are called, which takes some getting used to. You can't use commas when calling these functions, as every function technically takes only one argument. You need to invoke them like so:  
`add(3)(6)`  
It's a bit weird, and takes some getting used to. Especially for me, since I'm not a fan of Lisp like syntax (too many parenthesis).
However this allows you to supply arguments to a function at different times, slowly building up your logic as you get to it.

If you do need to use two or more arguments at once, you can as seen here:  
`const test = (x,y) => z => x + y + z`  
To invoke this, you have to pass two parameters as the first argument, otherwise it won't work properly.  
`test(5,2)(3) === 10`

Ok, so this is all well and good, but who cares? Why is this useful?

I find this helpful for writing small, atomic functions that you can compose into complex logic while still being maintainable. Because you don't need all the functions arguments up front, you can `partially apply` the arguments and build up your logic. As an example, here is some code I wrote a while ago for a personal scheduling project.

```javascript
const isDayOfWeek = dayOfWeek => date => equals(dayOfWeek)(getDay(date))
const isThursday = isDayOfWeek(Days.Thursday)
const isSunday = isDayOfWeek(Days.Sunday)

// If next week is in a different month, date is the last Sun/Mon/etc of the month
const isLastBlankDayOfMonth = date => isDiffMonth(addWeek(date), date)

const isLastSundayOfMonth = both(isSunday, isLastBlankDayOfMonth)
```

This code starts off with a function `isDayOfWeek`, which takes an enum of a day, a date, and returns whether they are equal. It is then partially applied into `isThursday` and `isSunday`. Those two functions only take a date, and will tell you if that date is a Thursday or a Sunday. So far, so good, simple expressions that can't be wrong.

I needed to send emails out on the last Sunday of the month, so I created a function that takes a date, and if 7 days later is in a different month, then that date must be the last one of the month.

Finally I can determine if a given date is the last Sunday of the month by combining the previous functions I've written.

At this point, since everything is small and atomic, I can be sure of the logic building up to it. I can also create more helper functions for other dates, and model my data very precisely in these functions.

If this is confusing or feels off, not to worry. This way of thinking is quite a paradigm change and takes a lot of immersion to be comfortable with. Thinking of computation as the result of evaluating mathematical expressions, as opposed to a list of instructions to the machine, was a big change for me when I started looking into this. The overarching idea is that our computers are better at issuing instructions than we are, so our time is better spent modelling our domain and letting the computer figure the nitty-gritty out.
