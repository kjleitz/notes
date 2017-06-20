# CS notes

These notes are not intended to be comprehensive, just a review of material that would mainly be good to know for a technical interiew. I can't wait to study _real_ computer science, but that's not the priority right now.

## Sorting

mergeSort

## Data structures

### arrays

- cost of retrieval: O(1)
- cost of shift: O(n)

### linked lists

- cost of retrieval: O(n)
- cost of shift: O(1)

address

trees

just realized linked lists are the same as trees, but just read from the opposite direction (which allows them to branch). next = parent. If you "delete" a node in a linked list by assigning the previous node's `next` to the node's `next` ("skipping" it), it's effectively deleted. But if you read it in reverse, it's just the branch of a tree. Shit's whack.

### binary search tree

A BST is structured such that the nodes adhere to two rules:

- no parent has more than two child nodes
- the left child is _less than_ the parent node and the right child is _greater than_ than the parent node

You try each node and check to see whether it will be more or less than that number, then continue until you find its spot. For example, making a BST out of [4, 1, 2, 5, 6, 3]:

```
   T1       T2     T3       T4             T5            T6
 
   4        4      4         4             4              4
           /      /        /   \         /   \          /   \
          1      1        1     5       1     5        1     5
                   \       \             \     \        \     \
                   2        2            2     6        2     6
                                                         \
                                                          3
```

(from [this Learn lesson](https://github.com/learn-co-students/binary-search-trees-v-000))

Each node will look like this:

```js
let node = {data: 4, rightChild: null, leftChild: null}
```

So, for example, a tree of [6, 1, 8, 4] will look like this:

```js
let bst = {
  data: 6,
  rightChild: {
    data: 8,
    leftChild: null,
    rightChild: null
  },
  leftChild: {
    data: 1,
    leftChild: null,
    rightChild: {
      data: 4,
      rightChild: null,
      leftChild: null 
    }
  }
};
```

#### Printing a BST in order

Check it:

```js
function inOrder(currentNode) {
  if (currentNode.left) inOrder(currentNode.left);
  console.log(currentNode.data);
  if (currentNode.right) inOrder(currentNode.right);
}
```

at each node, recursively call `inOrder`, doing `inOrder(left)`, then printing `currentNode.data`, then `inOrder(right)`. Makes sense.

#### Find or add to tree

```js
function findOrAdd(rootNode, someNode) {
  if (someNode.data < rootNode.data) {
    if (rootNode.left) {
      findOrAdd(rootNode.left, someNode);
    } else {
      rootNode.left = someNode;
      return rootNode.left;
    }
  } 
  
  if (someNode.data > rootNode.data) {
    if (rootNode.right) {
      findOrAdd(rootNode.right, someNode);
    } else {
      rootNode.right = someNode;
      return rootNode.right;
    }
  }
  
  return rootNode;
}
```

## old stuff

### Breadth-First Search

Maybe this shouldn't be here, but I wrote a recursive algorithm from scratch, not having seen a solution before, to perform a [Breadth-First Search](https://en.wikipedia.org/wiki/Breadth-first_search) (extra good job patting yourself on the back for this one, Keegs. _Keep in mind it's *not* actually technically a Breadth-First Search pattern, I don't think..._). I'm proud of it! I did it before reading the one in the Learn readme for this lesson and... and... I wanna keep it and I think it's good for posterity (it's probably bad code... but it was hard to reason about! And mine uses no mutation! And it's kinda _partially_ tail-recursive? call stack should only increase when encountering a new array... I should optimize this):

```js
const ary = [11, 12, [21, 22, [31, 32, 33], 23], [[34, 35], 24, [36]], 13];

function findWith(array, criteriaFn) {
  const firstMember = array[0];
  
  if (criteriaFn(firstMember)) return firstMember;
  if (typeof firstMember === "undefined") return null;
  
  if (Array.isArray(firstMember)) {
    const result = findWith(firstMember, criteriaFn);
    if (result) return result;
  }

  return findWith(array.slice(1), criteriaFn);
}

// works for all members of ary
findWith(ary, (n) => n === 34); //=> 34
findWith(ary, (n) => n === 12); //=> 12
findWith(ary, (n) => n === 24); //=> 24
findWith(ary, (n) => n === 50); //=> null
```

Here's the one from Learn (minus comments):

```js
function find(array, criteriaFn) {
  let current = array
  let next = []

  while (current) {
    if (criteriaFn(current)) {
      return current
    }
 
    if (Array.isArray(current)) {
      for (let i = 0, l = current.length; i < l; i++) {
        next.push(current[i])
      }
    }

    current = next.shift()
  }

  return null
}
```

...which I believe could be improved by:

```js
function find(array, criteriaFn) {
  let current = array
  const next = []

  while (current) {
    if (criteriaFn(current)) return current;
    if (Array.isArray(current)) next.push(...current);
    current = next.shift()
  }

  return null
}
```

But, those require mutation, which would be nice to avoid :)

For posterity, an explanation I gave Alex:

the function, in a broad view, is given an array of arrays of arrays (of arbitrary length and complexity—it’s a tree. every time there’s an array within the array, you can think of that as a branch, and branches can have branches can have branches). Within the function, it goes takes the first member of the array, sees if it matches. If it doesn’t match, it sees if it’s out of members. Every time it fails to find the result, it cuts off the head of the array, and runs again (thereby checking each member of the array by continuously cutting off the first member and checking the first member of the “new” array).

1. Check to see if the first member of the array it’s given matches the criteria… in that case, return that member, because that’s the one we’re looking for!

2. If there _is no first member_ of the array it’s been given (a.k.a. the array is empty, because it’s had its head cut off so many times), return `null` to signify that the current array being searched has been reduced to nothing and no matching member was found

3. If it gets past those without returning, we know that there _is_ a member present, and it _hasn’t_ matched the criteria, so we check to see if the member is an array, itself (a branch with leaves). If it is, we run the function on that branch the same way we ran it on the root array. Same deal. And from the context where it’s called again, right here, it doesn’t matter what’s going on inside the function. All we need to know is that if it finds the member somewhere in that nested array (even if it has branches, itself), it will return that member. If it exhausts all the members of the array, it will return `null`. So if it finds it, then return it, otherwise go on to the final step

4. Run the function again, but without the first member of the array (essentially doing the same thing but with the second member of the array, until the array is exhausted and returns `null`, or the result if it finds it)

So it will search _every member_ of the tree until it finds (and returns) it or returns `null`.

```      
      [11, 12, v, v, 13]
               |  |
              /    \
             ^      ^
 [21, 22, v, 23]  [v, 24, v]
          |        |       \
         /         |        \
        ^          ^         ^
[31, 32, 33]    [34, 35]    [36]
```

…like that. So when you call:

```
findWith(ary, (n) => n === 34);
```

…it goes:

```              
              [no, no, fw, fw: return 34, 13]
                       /   ^ \       ^
   ,------------------`   /   `---,   \
   v                     /        v    \
 [no, no, fw, no: return null]   [fw: return 34, 24, v]
          /    ^                   |  ^               \
         |      `------,           |   \               |
         v              \          v    \              v
       [no, no, no: return null]  [yes: return 34, 35] [36]
```

(`no` means it checked it and no match, `fw` means it called `findWith` on that branch, `return` means it returned something other than continuing with a shortened array)

### Recursion in general

Before we get into recursion in general... here's a factorial function I just wrote! :D This is getting fun! But also, I'm getting really fucking arrogant by saving these.

```js
function factorial(number) {
  return number <= 1 ? 1 : (number * factorial(number - 1));
}
```

So, I looked up some basic stuff about recursion, and found some helpful tips:

0. Put the algorithm in terms of calling the function with a next-simpler case! This is CRAZY helpful, trust me:

```js
sum([1, 2, 3, 4]) === 1 + sum([2, 3, 4]);
sum([1, 2, 3, 4]) === 1 + 2 + sum([3, 4]);
sum([1, 2, 3, 4]) === 1 + 2 + 3 + sum([4]);
sum([1, 2, 3, 4]) === 1 + 2 + 3 + 4;
```

(now you know: base case is array length equal to 1 - return array first member. else, return first member plus func(remainder of array). simple!)

1. Start by writing `if`. Why?
    - there is always a (base) case where the function does not call itself
    - there is always a (recursive) case where the function calls itself
2. Handle the simplest case(s)... a.k.a. the base case. Why?
    - the base case requires no looping; it may be complex, but there's nothing wonky going on, and you can define the "final procedure"
    - you're going to return a value
3. Write the recursive case(s)
    1. Write the recursive call. What do you use as the argument?
        - use the "next simplest value/input/state"
    2. Assume the recursive call works. Ask yourself:
        - what is it supposed to do/return? Maybe store that value in a `result` variable
        - how does that help/what can you do (repeatedly) with it to get your result?

### Return deepest nested element

Gah this took forever:

```js
function deepestChild() {
  const element = document.querySelector("#grand-node");
  let children = element.children;

  if (children.length === 0) return element;

  let nextChildren;

  for (;;) {
    nextChildren = [...children].filter(child => child.children.length > 0);

    if (nextChildren.length === 0) {
      return children[0];
    } else {
      children = nextChildren.reduce((acc, child) => [...acc, ...child.children], []);
    }
  }
}
```

From [this lab](https://github.com/learn-co-students/javascript-hide-and-seek-v-000).
