// TODO must happen before this one
// general "what is recursion"
// Watch out for stack overflow and not changing arguments
//When writing recursively, we have to consider how many times a function may call. In the worst case, we encounter [stack overflow](https://en.wikipedia.org/wiki/Stack_overflow) and our recursive function either errors or crashes our program. In considering //this, we know there is data being kept in memory as the recursive function continues to call itself. Our memory will be freed and our //stack will only be 'unwound' once one of our end conditions is met. What expensive operations do we have occ
(put this in other thing)

Let's write a recursive binary search function. In order to start thinking about this, let's assume the data entering the function (the arguments) have been previously checked for the right data types and sizes (no errors due to bad data!). While often no more complex than their iterative counterparts, recursive solutions can be intimidating when approached from a blank slate. Taking a moment to identify and isolate the discrete cases that can occur will help us structure our solution.

In our recursive solution, we have 4 base cases we need to address:
  1. The ```target``` is not present: *Signals an end to the recursion!*
  2. The ```target``` is present: *Signals an end to the recursion!*
  3. The ```target``` may be in the array, at a position > the current: *Recursion required!*
  4. The ```target``` may be in the array, at a position < the current: *Recursion required!*

If we encounter case 3 or 4, we know that we must recur with an altered argument (lest we place ourselves in an endless loop). As our ```target``` argument should never change, our only option is to alter the array! Let's give it a shot.

```javascript
function binarySearch(arr, target) {
  if (arr.length === 1 && target !== arr[0]) return false
  let midPoint = Math.floor(arr.length/2)
  if (arr[midPoint] === target) {
    return true
  } else if (arr[midPoint] < target) {
    return binarySearch(arr.slice(midPoint, arr.length), target)
  } else if (arr[midPoint] > target) {
    return binarySearch(arr.slice(0, midPoint), target)
  }
}
```

This solution is looking pretty good. It is short, to the point, and minimally cluttered. Two potential optimizations jump out: one readability based; one performance based. Let's address the readability first:


##### Readability Optimization

Our first discrete case on line 2 is not lumped with the rest of our logic handling. If we were to combine it with the other case tests, then we will always be assigning the temporary variable ```midPoint```, which means we potentially assign a variable that we never use (e.g. in discrete case 1)!

This boils down to a direct conflict between performance and readability. While we could simply replace ```Math.floor(arr.length/2)``` everywhere ```midPoint``` is called, we end up sacrificing our function's aesthetics. Let's consider the reality that we are solving this in Javascript, which belies how far we really want to go to squeeze performance [(see benchmarking)](https://julialang.org/benchmarks/). If the type of speed/memory gains we see from this degree of refactoring is a concern of ours, we may want to consider switching to a language ['closer to the metal'](https://www.quora.com/What-does-it-mean-for-a-programming-language-to-be-closer-to-the-metal).

Let's bite the negligible performance bullet and refactor for readability:
```javascript
function binarySearch(arr, target) {
  let midPoint = Math.floor(arr.length/2)
  switch (true) {
    case arr[midPoint] === target:
      return true
    case arr.length === 1:
      return false
    case arr[midPoint] < target:
      return binarySearch(arr.slice(midPoint, arr.length), target)
    case arr[midPoint] > target:
      return binarySearch(arr.slice(0, midPoint), target)
  }
}
```

Woah! Now there is a good looking function. The logic has been combined and we were able to shorten our discrete case 1 logic. Additionally, we are now using the switch statement, a control mechanism intended for handling discrete cases. Feels good. Let's turn our eye towards performance.


##### Performance Optimizations

Anytime we are implementing algorithms we should seek to minimize both the time and the amount of memory that is required to complete them. Memory management is particularly important when writing recursively as it is easy to carelessly pass more data as arguments than we need in each subsequent function call.

In our example above, we have both an expensive operation occurring with each recursion, as well as an excess of data being passed! Our culprit is the ```Array.prototype.slice()``` method that we are calling on ```arr```! In addition to copying half of the current array size with each iteration, we are passing that whole new array to the next recursive call. Meanwhile, as our function(s) chug along, all of these needless arrays are sitting in memory, waiting for the stack to be unwound.

In the end, we don't need brand new arrays (we aren't altering *any* elements). We only need to know the range of the array we have narrowed our search down to. Let's make an attempt to keep track of just the range (via a starting index and an ending index) instead of passing a new slice of the array:

```javascript
function binarySearch(arr, target, start=0, stop=(arr.length-1)) {
  let midPoint = Math.floor(((stop-start)/2) + start)
  switch (true) {
    case arr[midPoint] === target:
      return true
    case stop - start === 0:
      return false
    case arr[midPoint] < target:
      return binarySearch(arr, target, midPoint+1, stop)
    case arr[midPoint] > target:
      return binarySearch(arr, target, start, midPoint)
  }
}
```

We did a couple different things to get our function operating correctly, so let's unpack them:
  1. We provided default arguments so our recursive function can continue to be called simply à la ```binarySearch(myArray, val)```. These will only ever be used once if the callee does not explicitly provide a range, (allowing our function to default to searching the whole array)
  2. We ensured midPoint is updating properly, as we can no longer rely on the array resizing
  3. We made sure to pass an altered range every time we recur, narrowing ever closer towards either finding our element or confirming it does not exist. *Note: passing ```arr``` to each recursive call is relatively cheap -- it is only passing a pointer to the array's position in memory, as opposed to copying the whole array*

Whew! Here we saw how a common search algorithm can be tackled with recursion. Why were we successful? Instead of praying to the recursion Gods (which have never looked kindly on me), we took a step back, handled our 'recursion breaking conditions', and ensured we were narrowing the data with each call.
