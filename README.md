# This Containes all the helper function implementations we may need in our day to day coding practices

## Immutability Helper
If you use React, you would meet the scenario to copy the state for a slight change.

For example, for following state

```javascript
const state = {
  a: {
    b: {
      c: 1
    }
  },
  d: 2
}
```
if we are to modify d to a new state, we could use _.cloneDeep, but it is not efficient because state.a is cloned while we don't need to change that.

A better way is to do shallow copy like this

```javascript
const newState = {
  ...state,
  d: 3
}
```
now is the problem, if we want to modify c, we would have to do something like
```javascript
const newState = {
  ...state,
  a: {
    ...state.a,
    b: {
       ...state.b,
       c: 2
    }
  }
}
```
We can see that for simple data structure it would be enough to use spread operator, but for complex data structures, it is verbose.

Here comes the Immutability Helper. See this Doc [React Immutability Helper](https://github.com/kolodny/immutability-helper)
I have Implemented (Set , apply, merge, push operations in my code)
```javascript
function update(data, command) {
  if( '$push' in command) {
    if(Array.isArray(data)) {
      data = [...data, ...command['$push']]
    } else {
      throw('Error data is not an array')
    }
    return data;
  }
  if( '$merge' in command) {
    if(typeof data !== 'object') {
      throw('not an object to merge')
      // return;
    }
    data = {...data, ...command['$merge']}
    return data;
  }
  if( '$set' in command) {
      return command['$set'];
  }
  if( '$apply' in command) {
    return command['$apply'](data);
  }


  let newData;
  if(Array.isArray(data)) {
    newData = [...data]
  } else {
    newData = {...data}
  }

  for(let key in command) {
    newData[key] = update(data[key], command[key])
  }
  return newData;
}
```

## Memoization
Have you all used <code>React.memo()</code> lets see how this function would have been implemented.
We will implement a general <code>memo()</code> function, which caches the result once called, so when same arguments are passed in, the result will be returned right away.
The arguments are arbitrary, so memo should accept an extra resolver parameter, which is used to generate the cache key, like what <code> _.memoize()</code>  does.

```
```javascript
function memo(func, resolver) {
  const cache = {};
  return function (...args) {
      const context = this;
    let key;
    if(resolver) {
      key = resolver(...args)
    } else {
      key = JSON.stringify(args)
    }
    
    if(cache[key]) {
      return cache[key]
    }
    cache[key] = func.apply(context, args);
    return cache[key];
  }
}
```
## Find corresponding node in two identical DOM tree

**Recursion**
```javascript
function findCorrespondingNodeRecursively(a, bA, bB) {
  if (a === bA) {
    return bB;
  }
  
  for (let i = 0; i < bA.children.length; i++) {
    const childB = bB.children[i];
    const result = findCorrespondingNodeRecursively(a, bA.children[i], childB);
    if (result) {
      return result;
    }
  }
  
  return null;
}
```
**Iterative**
```javascript
function findCorrespondingNodeIteratively(a, bA, bB) {
  const stackA = [bA];
  const stackB = [bB];
  
  while (stackA.length) {
    const nodeA = stackA.pop();
    const nodeB = stackB.pop();
    
    if (nodeA === a) {
      return nodeB;
    }
    
    for (let i = nodeA.children.length - 1; i >= 0; i--) {
      stackA.push(nodeA.children[i]);
      stackB.push(nodeB.children[i]);
    }
  }
  
  return null;
}
```
**Tree Walker DOM API**
```javascript
const findCorrespondingNode = (rootA, rootB, target) => {
  const rootAWalker = document.createTreeWalker(rootA, NodeFilter.SHOW_ELEMENT);
  const rootBWalker = document.createTreeWalker(rootB, NodeFilter.SHOW_ELEMENT);
  let currentNodes = [rootAWalker.currentNode, rootBWalker.currentNode];
  while (currentNodes[0] !== target) {
    currentNodes = [rootAWalker.nextNode(), rootBWalker.nextNode()];
  }
  return currentNodes[1];
}
```
