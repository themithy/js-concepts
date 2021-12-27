I present a simple yet accurate Javascript interview. I call it "The Fibonacci
interview". Contrary to two-hours-long-spec-and-ecosystem-questions this interview
can be completed in 15 minutes and can be used to demonstrate the basic understanding
of algorithms and the ability to write actual code. 

The task can be substituted for any that requires working with loops and
substitutions, but in its "pure" form it goes like this:

> Write a function that computes the n-th number of Fibonacci series.

To recall, a number in the Fibonacci series is a sum of two predecessors:

```
0, 1, 1, 2, 3, 5, 8, 13, 21 ...
```

The simplest and often the first version is recursive the form, which is also
pretty straightforward:

```js
function fib(n) {
  /* Stoping criteria here: ... */

  return fib(n - 1) + fib(n - 2)
}
```

The recursive version makes the first point out of the total of five to be
earned.  Of course this version has many drawbacks including the poor
performance and quick stack overflow. So the iterative version is needed to fix
those issues:

```js
function fib(n) {
  /* Stoping criteria here: ... */

  let prev = 0
  let next = 1

  for (let i = 2; i < n; i++) {
    const temp = next + prev
    prev = next
    next = temp
  }

  return next  
}
```

At this moment there are already 2 points earned, and that means hire. Solving
such a task requires the basic understanding of algorithms necessary for
day-to-day duties like array processing, string conversion, state updates,
working with hooks in apps etc.  However there are still three "bonus points"
to be earned.

The first bonus point is for performing the mental struggle associated with
removing the temporary value and doing a swap of variables. A simple way to do
the swap is like this:

```js
A = A - B 
B = A + B
A = B - A
```

It turns out to be even simpler when incorporating the computation of the next
item in the series:

```js
next = next + prev
prev = next - prev
```

Until now we have not mentioned stopping criteria, but they are usually written
in that form:

```js
if (n == 0) {
  return 0
}
if (n == 1) {
  return 1
}
```

Here it can be observed that the value equals the argument, so it can be
simplified to this:

```js
if (n < 2) {
  return n
}
```

This is the fourth point. Nevertheless we can simplify it even more by going
back in time and computing negative numbers in the series:

```
... 2, -1, 1, 0, 1, 1, 2, 3, 5, 8, 13, 21 ...
```

So the final, all-inclusive solution looks like this:

```js
function fib(n) {
  let prev = 1
  let next = 0

  for (let i = 0; i < n; i++) {
    next = next + prev
    prev = next - prev
  }

  return next  
}
```

Of course those are all "dynamic programming" algorithms, but a closed-form solution
does exist:

```js
function fib(n) {
  return .5 ** n / SQRT_5 * ((1 + SQRT_5) ** n - (1 - SQRT_5) ** n)
}
```

