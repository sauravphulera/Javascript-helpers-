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
Given two same DOM tree A, B, and an Element a in A, find the corresponding Element b in B.

By corresponding, we mean a and b have the same relative position to their DOM tree root.

follow up

This could be a problem on general Tree structure with only children.

Could you solve it recursively and iteratively?

Could you solve this problem with special DOM api for better performance?

What are the time cost for each solution?

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

## Detect Type of a value
For all the basic data types in JavaScript, how could you write a function to detect the type of arbitrary data?

Besides basic types, you need to also handle also commonly used complex data type including <code>Array, ArrayBuffer, Map, Set, Date and Function</code>

The goal is not to list up all the data types but to show us how to solve the problem when we need to.

The type should be lowercase

**Solution**
```javascript
function detectType(data) {
  if(data === null) {
    return 'null'
  }
  const type = typeof data;

  if (type !== 'object') {
    return type;
  }
  const constructor = data.constructor;
  switch(constructor) {
    case Number: 
    return 'number';

    case Array:
    return 'array';

    case ArrayBuffer: 
    return 'arraybuffer';

    case Map:  
    return 'map';

    case Set: 
    return 'set';

    case Date: 
    return 'date';

    case Function: 
    return 'function';

    case String: 
    return 'string';

    case Boolean: 
    return 'boolean';
  }
}
```
## Implement JSON.stringify()
In this problem, you are asked to implement your own version of JSON.stringify().  
JSON.stringify() support two more parameters which is not covered here.

```javascript
function getArrayString(data) {
  const str = "[";
  const res =  data.reduce((str,value, i, arr) => {
    if(typeof value === 'symbol') {
      str+= 'null';
      return str;
    }
      str += stringify(value);
    if(i !== arr.length -1 ) {
      str += ','
    }
    return str;
  }, str)
  return res + ']'
}
/**
 * @param {any} data
 * @return {string}
 */
function stringify(data) {
  const type = detectType(data);

  if(type === 'bigint') {
    throw('Do not know how to serialize a BigInt')
  }

  const nulls = [NaN, null, undefined, Infinity];
  if(nulls.includes(data)) {
    return 'null'
  }

  if(type === null) {
    return 'null'
  }
  if(type === 'string') {
    return '"' + data + '"'
  }
  if (type === 'boolean' || type === 'number') {
    return ""+data;
  }

  if(type === 'array') {
    return getArrayString(data);
  }

  if(type === 'date') {
    return '"'+data.toISOString()+'"';
  }                              

  if(typeof data === 'object') {
     const keys = Object.keys(data);
     let res = '{';
     for(const [index, key] of keys.entries()) {
      if(data[key] === undefined) continue;
        res += '"' + key + '":'
        res += stringify(data[key]);
        if(index !== keys.length -1) {
          res += ','
        }
     }
     res += '}'
     return res;
  } 
}
```
