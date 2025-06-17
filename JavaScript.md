# JavaScript - Complete Guide for Developers from Other Languages

## JavaScript Gotchas and Confusions for Python/Other Language Developers

### Type System Differences

1. **Dynamic typing with coercion** - JavaScript automatically converts types, which can be confusing coming from Python.

   ```javascript
   // Python developers expect this to fail, but JavaScript coerces types
   console.log("5" + 3);        // "53" (string concatenation, not addition!)
   console.log("5" - 3);        // 2 (string converted to number for subtraction)
   console.log("5" * 3);        // 15 (string converted to number)
   console.log("5" / 3);        // 1.6666... (string converted to number)
   
   // Python equivalent would be:
   // "5" + 3  # TypeError in Python
   // int("5") + 3  # 8 in Python (explicit conversion needed)
   
   // More confusing examples
   console.log([] + []);        // "" (empty string, not empty array!)
   console.log([] + {});        // "[object Object]"
   console.log({} + []);        // "[object Object]" 
   console.log(true + 1);       // 2 (true becomes 1)
   console.log(false + 1);      // 1 (false becomes 0)
   ```

2. **Equality comparisons** - `==` vs `===` causes major confusion.

   ```javascript
   // == performs type coercion, === doesn't
   console.log(5 == "5");       // true (coerces string to number)
   console.log(5 === "5");      // false (strict equality, no coercion)
   
   console.log(true == 1);      // true (true becomes 1)
   console.log(true === 1);     // false
   
   console.log(null == undefined);   // true (special case)
   console.log(null === undefined);  // false
   
   console.log(0 == false);     // true (false becomes 0)
   console.log(0 === false);    // false
   
   console.log("" == false);    // true (both become 0)
   console.log("" === false);   // false
   
   // ALWAYS use === and !== unless you specifically need coercion
   // This is like Python's == which doesn't do automatic type conversion
   ```

3. **Truthy and falsy values** - Different from Python's truthiness.

   ```javascript
   // Falsy values in JavaScript (only these 8):
   console.log(Boolean(false));     // false
   console.log(Boolean(0));         // false
   console.log(Boolean(-0));        // false
   console.log(Boolean(0n));        // false (BigInt zero)
   console.log(Boolean(""));        // false (empty string)
   console.log(Boolean(null));      // false
   console.log(Boolean(undefined)); // false
   console.log(Boolean(NaN));       // false
   
   // Everything else is truthy (different from Python!)
   console.log(Boolean([]));        // true (empty array is truthy!)
   console.log(Boolean({}));        // true (empty object is truthy!)
   console.log(Boolean("0"));       // true (string "0" is truthy!)
   console.log(Boolean("false"));   // true (string "false" is truthy!)
   
   // Python comparison:
   // In Python: bool([]) is False, bool({}) is False
   // In JavaScript: Boolean([]) is true, Boolean({}) is true
   ```

4. **Variable declarations and hoisting** - Very different from Python's scope rules.

   ```javascript
   // var is hoisted and function-scoped (avoid using var!)
   console.log(x);  // undefined (not ReferenceError!)
   var x = 5;
   
   // This is equivalent to:
   // var x;  // hoisted to top, initialized as undefined
   // console.log(x);  // undefined
   // x = 5;
   
   // let and const are block-scoped (more like Python)
   console.log(y);  // ReferenceError (temporal dead zone)
   let y = 5;
   
   // Function hoisting
   sayHello();  // Works! Prints "Hello"
   
   function sayHello() {
       console.log("Hello");
   }
   
   // But function expressions don't hoist
   sayBye();  // TypeError: sayBye is not a function
   var sayBye = function() {
       console.log("Bye");
   };
   ```

### Common Pitfalls

5. **Array and object comparison** - Objects are compared by reference, not value.

   ```javascript
   // Arrays and objects are compared by reference
   console.log([1, 2, 3] === [1, 2, 3]);  // false (different objects)
   console.log({a: 1} === {a: 1});        // false (different objects)
   
   // To compare values, you need to:
   // For arrays:
   const arr1 = [1, 2, 3];
   const arr2 = [1, 2, 3];
   console.log(JSON.stringify(arr1) === JSON.stringify(arr2));  // true
   // Or use array methods
   console.log(arr1.length === arr2.length && arr1.every((val, i) => val === arr2[i]));
   
   // For objects:
   const obj1 = {a: 1, b: 2};
   const obj2 = {a: 1, b: 2};
   console.log(JSON.stringify(obj1) === JSON.stringify(obj2));  // true (careful with key order!)
   
   // Python comparison: [1, 2, 3] == [1, 2, 3] is True
   ```

6. **`this` keyword confusion** - Very different from Python's `self`.

   ```javascript
   // In Python, 'self' is explicit and predictable
   // In JavaScript, 'this' depends on HOW the function is called
   
   const obj = {
       name: "Alice",
       greet: function() {
           console.log("Hello, " + this.name);
       },
       greetArrow: () => {
           console.log("Hello, " + this.name);  // 'this' is NOT obj!
       }
   };
   
   obj.greet();        // "Hello, Alice" (this = obj)
   obj.greetArrow();   // "Hello, undefined" (this = global object/undefined)
   
   const greetFunc = obj.greet;
   greetFunc();        // "Hello, undefined" (this = global object/undefined)
   
   // Arrow functions inherit 'this' from surrounding scope
   // Regular functions get 'this' based on how they're called
   ```

7. **Numbers and precision** - JavaScript has only one number type.

   ```javascript
   // JavaScript has only one number type (64-bit floating point)
   console.log(typeof 42);      // "number"
   console.log(typeof 42.5);    // "number"
   console.log(typeof Infinity); // "number"
   console.log(typeof NaN);     // "number" (Not a Number is a number!)
   
   // Precision issues (like Python floats)
   console.log(0.1 + 0.2);      // 0.30000000000000004 (not 0.3!)
   console.log(0.1 + 0.2 === 0.3);  // false
   
   // Safe integer range
   console.log(Number.MAX_SAFE_INTEGER);  // 9007199254740991
   console.log(Number.MIN_SAFE_INTEGER);  // -9007199254740991
   
   // For larger integers, use BigInt
   console.log(BigInt(9007199254740991) + BigInt(1));  // 9007199254740992n
   ```

## Type Conversion and Casting

8. **Explicit type conversion** - How to properly convert types in JavaScript.

   ```javascript
   // String conversion
   String(123);        // "123"
   String(true);       // "true"
   String(null);       // "null"
   String(undefined);  // "undefined"
   (123).toString();   // "123"
   
   // Number conversion
   Number("123");      // 123
   Number("123.45");   // 123.45
   Number("123abc");   // NaN
   Number("");         // 0 (empty string becomes 0!)
   Number(" ");        // 0 (whitespace becomes 0!)
   Number(true);       // 1
   Number(false);      // 0
   Number(null);       // 0
   Number(undefined);  // NaN
   
   // parseInt and parseFloat for strings
   parseInt("123");      // 123
   parseInt("123.45");   // 123 (stops at decimal)
   parseInt("123abc");   // 123 (stops at non-digit)
   parseFloat("123.45"); // 123.45
   parseFloat("123.45abc"); // 123.45
   
   // Boolean conversion
   Boolean(1);         // true
   Boolean(0);         // false
   Boolean("hello");   // true
   Boolean("");        // false
   Boolean([]);        // true (!)
   Boolean({});        // true (!)
   
   // Quick conversions using operators
   +"123";             // 123 (unary + converts to number)
   !!"hello";          // true (double ! converts to boolean)
   ```

9. **Checking types safely** - How to properly check types in JavaScript.

   ```javascript
   // typeof operator
   typeof 123;           // "number"
   typeof "hello";       // "string"
   typeof true;          // "boolean"
   typeof undefined;     // "undefined"
   typeof null;          // "object" (this is a famous bug!)
   typeof [];            // "object"
   typeof {};            // "object"
   typeof function(){};  // "function"
   
   // Better type checking
   Array.isArray([]);              // true
   Array.isArray({});              // false
   
   // Check for null specifically
   value === null;                 // only way to check for null
   
   // Check for NaN
   Number.isNaN(NaN);              // true
   Number.isNaN("hello");          // false (not converted!)
   isNaN("hello");                 // true (converts then checks)
   
   // instanceof for objects
   [] instanceof Array;            // true
   {} instanceof Object;           // true
   new Date() instanceof Date;     // true
   ```

## JavaScript for Data Structures and Algorithms

### Arrays (Dynamic Lists)

10. **Array methods essential for DSA** - JavaScript arrays with useful methods.

    ```javascript
    // Array creation
    let arr = [1, 2, 3, 4, 5];
    let arr2 = new Array(5);        // [undefined, undefined, undefined, undefined, undefined]
    let arr3 = new Array(5).fill(0); // [0, 0, 0, 0, 0]
    let arr4 = Array.from({length: 5}, (_, i) => i); // [0, 1, 2, 3, 4]
    
    // Essential methods for DSA
    arr.push(6);           // Add to end: [1, 2, 3, 4, 5, 6]
    arr.pop();             // Remove from end: returns 6
    arr.unshift(0);        // Add to beginning: [0, 1, 2, 3, 4, 5]
    arr.shift();           // Remove from beginning: returns 0
    
    // Access and modify
    arr[0];                // First element
    arr[arr.length - 1];   // Last element
    arr.length;            // Size of array
    
    // Slicing and splicing
    arr.slice(1, 3);       // [2, 3] (doesn't modify original)
    arr.splice(1, 2);      // Removes 2 elements starting at index 1
    arr.splice(1, 0, 'new'); // Insert 'new' at index 1
    
    // Searching
    arr.indexOf(3);        // Index of first occurrence of 3
    arr.lastIndexOf(3);    // Index of last occurrence of 3
    arr.includes(3);       // true if 3 exists in array
    arr.find(x => x > 3);  // First element > 3
    arr.findIndex(x => x > 3); // Index of first element > 3
    
    // Iteration methods
    arr.forEach((val, index) => console.log(val, index));
    arr.map(x => x * 2);   // Transform each element
    arr.filter(x => x > 2); // Keep elements that match condition
    arr.reduce((sum, x) => sum + x, 0); // Reduce to single value
    
    // Sorting (IMPORTANT: sorts as strings by default!)
    [3, 1, 4, 1, 5].sort(); // [1, 1, 3, 4, 5] - works for single digits
    [10, 2, 1, 20].sort();  // [1, 10, 2, 20] - WRONG! (string sort)
    [10, 2, 1, 20].sort((a, b) => a - b); // [1, 2, 10, 20] - correct number sort
    [10, 2, 1, 20].sort((a, b) => b - a); // [20, 10, 2, 1] - descending
    ```

### Objects (Hash Maps/Dictionaries)

11. **Objects and Maps for DSA** - Hash tables and dictionaries in JavaScript.

    ```javascript
    // Object as hash map (like Python dict)
    let hashMap = {};
    hashMap['key1'] = 'value1';
    hashMap.key2 = 'value2';        // Same as hashMap['key2']
    
    // Check if key exists
    'key1' in hashMap;              // true
    hashMap.hasOwnProperty('key1'); // true
    hashMap.key1 !== undefined;     // true (but undefined values will fail)
    
    // Get all keys/values
    Object.keys(hashMap);           // ['key1', 'key2']
    Object.values(hashMap);         // ['value1', 'value2']
    Object.entries(hashMap);        // [['key1', 'value1'], ['key2', 'value2']]
    
    // Iterate
    for (let key in hashMap) {
        console.log(key, hashMap[key]);
    }
    
    // Map object (better for non-string keys)
    let map = new Map();
    map.set('string-key', 'value1');
    map.set(42, 'value2');          // Number key
    map.set(true, 'value3');        // Boolean key
    
    map.get('string-key');          // 'value1'
    map.has(42);                    // true
    map.delete(true);               // removes boolean key
    map.size;                       // number of entries
    
    // Map iteration
    for (let [key, value] of map) {
        console.log(key, value);
    }
    
    // Frequency counter pattern (common in DSA)
    function countFrequency(arr) {
        let freq = {};
        for (let item of arr) {
            freq[item] = (freq[item] || 0) + 1;
        }
        return freq;
    }
    
    countFrequency([1, 2, 2, 3, 3, 3]); // {1: 1, 2: 2, 3: 3}
    ```

### Sets for Unique Collections

12. **Sets for handling unique elements** - Like Python sets.

    ```javascript
    // Set creation
    let set = new Set();
    let set2 = new Set([1, 2, 3, 2, 1]); // {1, 2, 3} - duplicates removed
    
    // Set operations
    set.add(1);
    set.add(2);
    set.add(1);           // Won't add duplicate
    
    set.has(1);           // true
    set.delete(2);        // removes 2
    set.size;             // number of elements
    set.clear();          // removes all elements
    
    // Set iteration
    for (let value of set) {
        console.log(value);
    }
    
    // Convert between array and set
    let arr = [1, 2, 2, 3, 3, 3];
    let uniqueArr = [...new Set(arr)]; // [1, 2, 3] - remove duplicates
    
    // Set operations (intersection, union, difference)
    let setA = new Set([1, 2, 3]);
    let setB = new Set([3, 4, 5]);
    
    // Union
    let union = new Set([...setA, ...setB]); // {1, 2, 3, 4, 5}
    
    // Intersection
    let intersection = new Set([...setA].filter(x => setB.has(x))); // {3}
    
    // Difference
    let difference = new Set([...setA].filter(x => !setB.has(x))); // {1, 2}
    ```

### Strings for Text Processing

13. **String manipulation for DSA** - Common string operations.

    ```javascript
    let str = "Hello World";
    
    // Basic properties
    str.length;                    // 11
    str[0];                       // 'H' (first character)
    str.charAt(0);                // 'H' (same, but safer for old browsers)
    str.charCodeAt(0);            // 72 (ASCII code of 'H')
    
    // String methods
    str.toLowerCase();            // "hello world"
    str.toUpperCase();            // "HELLO WORLD"
    str.substring(0, 5);          // "Hello"
    str.slice(0, 5);              // "Hello" (similar to substring)
    str.slice(-5);                // "World" (last 5 characters)
    
    // Searching
    str.indexOf('o');             // 4 (first occurrence)
    str.lastIndexOf('o');         // 7 (last occurrence)
    str.includes('World');        // true
    str.startsWith('Hello');      // true
    str.endsWith('World');        // true
    
    // Splitting and joining
    str.split(' ');               // ["Hello", "World"]
    str.split('');                // ["H", "e", "l", "l", "o", " ", "W", "o", "r", "l", "d"]
    ['Hello', 'World'].join(' '); // "Hello World"
    
    // String to array and back (for manipulation)
    let chars = str.split('');
    chars.reverse();
    let reversed = chars.join(''); // "dlroW olleH"
    
    // String replacement
    str.replace('World', 'JavaScript'); // "Hello JavaScript"
    str.replaceAll('l', 'L');          // "HeLLo WorLd"
    
    // Regular expressions
    str.match(/[a-z]/g);          // ["e", "l", "l", "o", "o", "r", "l", "d"]
    str.replace(/\s+/g, '-');     // "Hello-World"
    
    // Template literals (for building strings)
    let name = "Alice";
    let age = 25;
    let message = `Hello, ${name}! You are ${age} years old.`;
    ```

### Functions and Closures

14. **Functions for DSA algorithms** - Function concepts important for algorithms.

    ```javascript
    // Function declarations vs expressions
    function factorial(n) {        // Function declaration (hoisted)
        if (n <= 1) return 1;
        return n * factorial(n - 1);
    }
    
    const fibonacci = function(n) { // Function expression (not hoisted)
        if (n <= 1) return n;
        return fibonacci(n - 1) + fibonacci(n - 2);
    };
    
    // Arrow functions (concise syntax)
    const square = x => x * x;
    const add = (a, b) => a + b;
    const isEven = n => n % 2 === 0;
    
    // Higher-order functions (functions that take/return functions)
    function memoize(fn) {
        const cache = {};
        return function(...args) {
            const key = JSON.stringify(args);
            if (key in cache) {
                return cache[key];
            }
            const result = fn.apply(this, args);
            cache[key] = result;
            return result;
        };
    }
    
    // Memoized fibonacci (much faster)
    const fastFib = memoize(function(n) {
        if (n <= 1) return n;
        return fastFib(n - 1) + fastFib(n - 2);
    });
    
    // Closures (functions that remember their scope)
    function createCounter() {
        let count = 0;
        return function() {
            return ++count;
        };
    }
    
    const counter = createCounter();
    console.log(counter()); // 1
    console.log(counter()); // 2
    ```

### Advanced Data Structures

15. **Implementing common DSA structures** - Building data structures in JavaScript.

    ```javascript
    // Stack (LIFO)
    class Stack {
        constructor() {
            this.items = [];
        }
        
        push(item) {
            this.items.push(item);
        }
        
        pop() {
            return this.items.pop();
        }
        
        peek() {
            return this.items[this.items.length - 1];
        }
        
        isEmpty() {
            return this.items.length === 0;
        }
        
        size() {
            return this.items.length;
        }
    }
    
    // Queue (FIFO)
    class Queue {
        constructor() {
            this.items = [];
        }
        
        enqueue(item) {
            this.items.push(item);
        }
        
        dequeue() {
            return this.items.shift();
        }
        
        front() {
            return this.items[0];
        }
        
        isEmpty() {
            return this.items.length === 0;
        }
        
        size() {
            return this.items.length;
        }
    }
    
    // Binary Tree Node
    class TreeNode {
        constructor(val, left = null, right = null) {
            this.val = val;
            this.left = left;
            this.right = right;
        }
    }
    
    // Linked List Node
    class ListNode {
        constructor(val, next = null) {
            this.val = val;
            this.next = next;
        }
    }
    
    // Binary Search Tree
    class BST {
        constructor() {
            this.root = null;
        }
        
        insert(val) {
            this.root = this._insertRec(this.root, val);
        }
        
        _insertRec(node, val) {
            if (!node) return new TreeNode(val);
            
            if (val < node.val) {
                node.left = this._insertRec(node.left, val);
            } else if (val > node.val) {
                node.right = this._insertRec(node.right, val);
            }
            
            return node;
        }
        
        search(val) {
            return this._searchRec(this.root, val);
        }
        
        _searchRec(node, val) {
            if (!node || node.val === val) return node;
            
            if (val < node.val) {
                return this._searchRec(node.left, val);
            }
            return this._searchRec(node.right, val);
        }
        
        inorder() {
            const result = [];
            this._inorderRec(this.root, result);
            return result;
        }
        
        _inorderRec(node, result) {
            if (node) {
                this._inorderRec(node.left, result);
                result.push(node.val);
                this._inorderRec(node.right, result);
            }
        }
    }
    ```

### Sorting Algorithms

14. **Sorting algorithms in JavaScript** - Essential sorting algorithms for DSA.

    ```javascript
    // Quick sort
    function quickSort(arr) {
        if (arr.length <= 1) return arr;
        
        const pivot = arr[Math.floor(arr.length / 2)];
        const left = arr.filter(x => x < pivot);
        const middle = arr.filter(x => x === pivot);
        const right = arr.filter(x => x > pivot);
        
        return [...quickSort(left), ...middle, ...quickSort(right)];
    }
    
    // Merge sort
    function mergeSort(arr) {
        if (arr.length <= 1) return arr;
        
        const mid = Math.floor(arr.length / 2);
        const left = mergeSort(arr.slice(0, mid));
        const right = mergeSort(arr.slice(mid));
        
        return merge(left, right);
    }
    
    function merge(left, right) {
        const result = [];
        let i = 0, j = 0;
        
        while (i < left.length && j < right.length) {
            if (left[i] <= right[j]) {
                result.push(left[i]);
                i++;
            } else {
                result.push(right[j]);
                j++;
            }
        }
        
        return result.concat(left.slice(i)).concat(right.slice(j));
    }
    
    // Bubble sort (for educational purposes)
    function bubbleSort(arr) {
        const n = arr.length;
        for (let i = 0; i < n - 1; i++) {
            for (let j = 0; j < n - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]; // Swap
                }
            }
        }
        return arr;
    }
    
    // Insertion sort
    function insertionSort(arr) {
        for (let i = 1; i < arr.length; i++) {
            let key = arr[i];
            let j = i - 1;
            
            while (j >= 0 && arr[j] > key) {
                arr[j + 1] = arr[j];
                j--;
            }
            arr[j + 1] = key;
        }
        return arr;
    }
    
    // Selection sort
    function selectionSort(arr) {
        for (let i = 0; i < arr.length - 1; i++) {
            let minIdx = i;
            for (let j = i + 1; j < arr.length; j++) {
                if (arr[j] < arr[minIdx]) {
                    minIdx = j;
                }
            }
            if (minIdx !== i) {
                [arr[i], arr[minIdx]] = [arr[minIdx], arr[i]];
            }
        }
        return arr;
    }
    ```

### String Manipulation for DSA

15. **String manipulation essential for DSA** - Common string operations and algorithms.

    ```javascript
    let str = "Hello World";
    
    // Basic properties and methods
    str.length;                    // 11
    str[0];                       // 'H' (first character)
    str.charAt(0);                // 'H' (same, but safer for old browsers)
    str.charCodeAt(0);            // 72 (ASCII code of 'H')
    
    // String methods
    str.toLowerCase();            // "hello world"
    str.toUpperCase();            // "HELLO WORLD"
    str.substring(0, 5);          // "Hello"
    str.slice(0, 5);              // "Hello" (similar to substring)
    str.slice(-5);                // "World" (last 5 characters)
    
    // Searching
    str.indexOf('o');             // 4 (first occurrence)
    str.lastIndexOf('o');         // 7 (last occurrence)
    str.includes('World');        // true
    str.startsWith('Hello');      // true
    str.endsWith('World');        // true
    
    // Splitting and joining
    str.split(' ');               // ["Hello", "World"]
    str.split('');                // ["H", "e", "l", "l", "o", " ", "W", "o", "r", "l", "d"]
    ['Hello', 'World'].join(' '); // "Hello World"
    
    // String to array and back (for manipulation)
    let chars = str.split('');
    chars.reverse();
    let reversed = chars.join(''); // "dlroW olleH"
    
    // String replacement
    str.replace('World', 'JavaScript'); // "Hello JavaScript"
    str.replaceAll('l', 'L');          // "HeLLo WorLd"
    
    // Regular expressions
    str.match(/[a-z]/g);          // ["e", "l", "l", "o", "o", "r", "l", "d"]
    str.replace(/\s+/g, '-');     // "Hello-World"
    
    // Template literals (for building strings)
    let name = "Alice";
    let age = 25;
    let message = `Hello, ${name}! You are ${age} years old.`;
    
    // Common string algorithms
    
    // Check if palindrome
    function isPalindrome(s) {
        s = s.toLowerCase().replace(/[^a-z0-9]/g, '');
        return s === s.split('').reverse().join('');
    }
    
    // Check if anagram
    function isAnagram(s1, s2) {
        const sort = str => str.toLowerCase().split('').sort().join('');
        return sort(s1) === sort(s2);
    }
    
    // Longest common prefix
    function longestCommonPrefix(strs) {
        if (!strs.length) return '';
        
        for (let i = 0; i < strs[0].length; i++) {
            for (let j = 1; j < strs.length; j++) {
                if (strs[j][i] !== strs[0][i]) {
                    return strs[0].slice(0, i);
                }
            }
        }
        return strs[0];
    }
    ```

### Advanced Data Structures

16. **Implementing common DSA structures** - Building data structures in JavaScript.

    ```javascript
    // Stack (LIFO)
    class Stack {
        constructor() {
            this.items = [];
        }
        
        push(item) {
            this.items.push(item);
        }
        
        pop() {
            return this.items.pop();
        }
        
        peek() {
            return this.items[this.items.length - 1];
        }
        
        isEmpty() {
            return this.items.length === 0;
        }
        
        size() {
            return this.items.length;
        }
    }
    
    // Queue (FIFO)
    class Queue {
        constructor() {
            this.items = [];
        }
        
        enqueue(item) {
            this.items.push(item);
        }
        
        dequeue() {
            return this.items.shift();
        }
        
        front() {
            return this.items[0];
        }
        
        isEmpty() {
            return this.items.length === 0;
        }
        
        size() {
            return this.items.length;
        }
    }
    
    // Binary Tree Node
    class TreeNode {
        constructor(val, left = null, right = null) {
            this.val = val;
            this.left = left;
            this.right = right;
        }
    }
    
    // Linked List Node
    class ListNode {
        constructor(val, next = null) {
            this.val = val;
            this.next = next;
        }
    }
    
    // Binary Search Tree
    class BST {
        constructor() {
            this.root = null;
        }
        
        insert(val) {
            this.root = this._insertRec(this.root, val);
        }
        
        _insertRec(node, val) {
            if (!node) return new TreeNode(val);
            
            if (val < node.val) {
                node.left = this._insertRec(node.left, val);
            } else if (val > node.val) {
                node.right = this._insertRec(node.right, val);
            }
            
            return node;
        }
        
        search(val) {
            return this._searchRec(this.root, val);
        }
        
        _searchRec(node, val) {
            if (!node || node.val === val) return node;
            
            if (val < node.val) {
                return this._searchRec(node.left, val);
            }
            return this._searchRec(node.right, val);
        }
        
        inorder() {
            const result = [];
            this._inorderRec(this.root, result);
            return result;
        }
        
        _inorderRec(node, result) {
            if (node) {
                this._inorderRec(node.left, result);
                result.push(node.val);
                this._inorderRec(node.right, result);
            }
        }
    }
    
    // Min Heap
    class MinHeap {
        constructor() {
            this.heap = [];
        }
        
        getMin() {
            return this.heap[0];
        }
        
        insert(val) {
            this.heap.push(val);
            this._heapifyUp();
        }
        
        extractMin() {
            if (this.heap.length === 0) return null;
            if (this.heap.length === 1) return this.heap.pop();
            
            const min = this.heap[0];
            this.heap[0] = this.heap.pop();
            this._heapifyDown();
            return min;
        }
        
        _heapifyUp() {
            let index = this.heap.length - 1;
            while (index > 0) {
                const parentIndex = Math.floor((index - 1) / 2);
                if (this.heap[parentIndex] <= this.heap[index]) break;
                
                [this.heap[parentIndex], this.heap[index]] = [this.heap[index], this.heap[parentIndex]];
                index = parentIndex;
            }
        }
        
        _heapifyDown() {
            let index = 0;
            while (this._hasLeftChild(index)) {
                let smallerChildIndex = this._getLeftChildIndex(index);
                if (this._hasRightChild(index) && this._getRightChild(index) < this._getLeftChild(index)) {
                    smallerChildIndex = this._getRightChildIndex(index);
                }
                
                if (this.heap[index] < this.heap[smallerChildIndex]) break;
                
                [this.heap[index], this.heap[smallerChildIndex]] = [this.heap[smallerChildIndex], this.heap[index]];
                index = smallerChildIndex;
            }
        }
        
        _getLeftChildIndex(parentIndex) { return 2 * parentIndex + 1; }
        _getRightChildIndex(parentIndex) { return 2 * parentIndex + 2; }
        _getLeftChild(index) { return this.heap[this._getLeftChildIndex(index)]; }
        _getRightChild(index) { return this.heap[this._getRightChildIndex(index)]; }
        _hasLeftChild(index) { return this._getLeftChildIndex(index) < this.heap.length; }
        _hasRightChild(index) { return this._getRightChildIndex(index) < this.heap.length; }
    }
    ```

### Functions and Closures

17. **Functions for DSA algorithms** - Function concepts important for algorithms.

    ```javascript
    // Function declarations vs expressions
    function factorial(n) {        // Function declaration (hoisted)
        if (n <= 1) return 1;
        return n * factorial(n - 1);
    }
    
    const fibonacci = function(n) { // Function expression (not hoisted)
        if (n <= 1) return n;
        return fibonacci(n - 1) + fibonacci(n - 2);
    };
    
    // Arrow functions (concise syntax)
    const square = x => x * x;
    const add = (a, b) => a + b;
    const isEven = n => n % 2 === 0;
    
    // Higher-order functions (functions that take/return functions)
    function memoize(fn) {
        const cache = {};
        return function(...args) {
            const key = JSON.stringify(args);
            if (key in cache) {
                return cache[key];
            }
            const result = fn.apply(this, args);
            cache[key] = result;
            return result;
        };
    }
    
    // Memoized fibonacci (much faster)
    const fastFib = memoize(function(n) {
        if (n <= 1) return n;
        return fastFib(n - 1) + fastFib(n - 2);
    });
    
    // Closures (functions that remember their scope)
    function createCounter() {
        let count = 0;
        return function() {
            return ++count;
        };
    }
    
    const counter = createCounter();
    console.log(counter()); // 1
    console.log(counter()); // 2
    
    // Currying (partial application)
    function multiply(a) {
        return function(b) {
            return a * b;
        };
    }
    
    const double = multiply(2);
    console.log(double(5)); // 10
    
    // Or with arrow functions
    const multiplyArrow = a => b => a * b;
    const triple = multiplyArrow(3);
    console.log(triple(4)); // 12
    ```

### Performance and Time Complexity

18. **Performance considerations in JavaScript** - Writing efficient code for DSA.

    ```javascript
    // Time complexity examples
    
    // O(1) - Constant time
    function getFirst(arr) {
        return arr[0];  // Always takes same time regardless of array size
    }
    
    // O(n) - Linear time
    function findMax(arr) {
        let max = arr[0];
        for (let i = 1; i < arr.length; i++) {  // Loops through entire array
            if (arr[i] > max) max = arr[i];
        }
        return max;
    }
    
    // O(n²) - Quadratic time
    function bubbleSortExample(arr) {
        for (let i = 0; i < arr.length; i++) {      // Nested loops
            for (let j = 0; j < arr.length - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]; // Swap
                }
            }
        }
        return arr;
    }
    
    // O(log n) - Logarithmic time
    function binarySearchRecursive(arr, target, left = 0, right = arr.length - 1) {
        if (left > right) return -1;
        
        const mid = Math.floor((left + right) / 2);
        if (arr[mid] === target) return mid;
        
        if (arr[mid] > target) {
            return binarySearchRecursive(arr, target, left, mid - 1);
        } else {
            return binarySearchRecursive(arr, target, mid + 1, right);
        }
    }
    
    // Performance tips
    
    // 1. Avoid creating new arrays unnecessarily
    // Bad: O(n) space for each operation
    function badReverse(str) {
        return str.split('').reverse().join(''); // Creates 3 arrays
    }
    
    // Better: Process in place when possible
    function goodReverse(str) {
        let result = '';
        for (let i = str.length - 1; i >= 0; i--) {
            result += str[i];
        }
        return result;
    }
    
    // 2. Use appropriate data structures
    // Bad: O(n) lookup time
    const badLookup = (arr, target) => arr.includes(target);
    
    // Good: O(1) average lookup time
    const goodLookup = (set, target) => set.has(target);
    
    // 3. Cache expensive computations
    const expensiveFunction = memoize((n) => {
        // Some expensive calculation
        let result = 0;
        for (let i = 0; i <= n; i++) {
            result += Math.sqrt(i);
        }
        return result;
    });
    
    // 4. Use built-in methods when appropriate
    // They're often optimized in the JavaScript engine
    const numbers = [1, 2, 3, 4, 5];
    const doubled = numbers.map(x => x * 2);           // Good
    const evens = numbers.filter(x => x % 2 === 0);    // Good
    const sum = numbers.reduce((a, b) => a + b, 0);    // Good
    ```

## Common JavaScript Patterns for DSA

19. **Essential patterns every developer should know** - Practical patterns for problem solving.

    ```javascript
    // Destructuring for swapping and multiple assignment
    let a = 1, b = 2;
    [a, b] = [b, a];  // Swap without temp variable
    
    // Array destructuring
    const [first, second, ...rest] = [1, 2, 3, 4, 5];
    // first = 1, second = 2, rest = [3, 4, 5]
    
    // Object destructuring
    const {name, age, ...others} = {name: 'Alice', age: 25, city: 'NYC'};
    
    // Spread operator for arrays
    const arr1 = [1, 2, 3];
    const arr2 = [4, 5, 6];
    const combined = [...arr1, ...arr2];  // [1, 2, 3, 4, 5, 6]
    const copied = [...arr1];             // Shallow copy
    
    // Default parameters
    function greet(name = 'World') {
        return `Hello, ${name}!`;
    }
    
    // Rest parameters
    function sum(...numbers) {
        return numbers.reduce((a, b) => a + b, 0);
    }
    sum(1, 2, 3, 4, 5); // 15
    
    // Short-circuit evaluation
    const value = someValue || 'default';  // Use 'default' if someValue is falsy
    const safe = obj && obj.property;      // Only access property if obj exists
    
    // Nullish coalescing (newer feature)
    const value2 = someValue ?? 'default'; // Use 'default' only if someValue is null/undefined
    
    // Optional chaining (newer feature)
    const street = person?.address?.street; // Safely access nested properties
    
    // Array methods chaining
    const result = [1, 2, 3, 4, 5, 6]
        .filter(x => x % 2 === 0)    // [2, 4, 6]
        .map(x => x * 2)             // [4, 8, 12]
        .reduce((a, b) => a + b);    // 24
    
    // Matrix/2D array creation
    const rows = 3, cols = 4;
    const matrix = Array(rows).fill().map(() => Array(cols).fill(0));
    // Creates 3x4 matrix filled with zeros
    
    // Deep copy of objects (simple method)
    const original = {a: 1, b: {c: 2}};
    const deepCopy = JSON.parse(JSON.stringify(original));
    
    // Frequency counting pattern
    function getCharFrequency(str) {
        return str.split('').reduce((freq, char) => {
            freq[char] = (freq[char] || 0) + 1;
            return freq;
        }, {});
    }
    
    // Range creation
    const range = (start, end) => Array.from({length: end - start}, (_, i) => start + i);
    console.log(range(1, 6)); // [1, 2, 3, 4, 5]
    
    // Find min/max in array
    const numbers = [1, 5, 3, 9, 2];
    const min = Math.min(...numbers);  // 1
    const max = Math.max(...numbers);  // 9
    
    // Remove duplicates from array
    const withDuplicates = [1, 2, 2, 3, 3, 4];
    const unique = [...new Set(withDuplicates)]; // [1, 2, 3, 4]
    
    // Flatten array
    const nested = [[1, 2], [3, 4], [5]];
    const flattened = nested.flat();  // [1, 2, 3, 4, 5]
    const deepNested = [1, [2, [3, [4]]]];
    const deepFlat = deepNested.flat(Infinity); // [1, 2, 3, 4]
    ```

### Essential Algorithm Patterns

20. **Common algorithm patterns in JavaScript** - Essential patterns for DSA.

    ```javascript
    // Two pointers technique
    function twoSum(arr, target) {
        let left = 0, right = arr.length - 1;
        
        while (left < right) {
            const sum = arr[left] + arr[right];
            if (sum === target) {
                return [left, right];
            } else if (sum < target) {
                left++;
            } else {
                right--;
            }
        }
        return [-1, -1];
    }
    
    // Sliding window technique
    function maxSubarraySum(arr, k) {
        if (arr.length < k) return null;
        
        let windowSum = arr.slice(0, k).reduce((a, b) => a + b);
        let maxSum = windowSum;
        
        for (let i = k; i < arr.length; i++) {
            windowSum = windowSum - arr[i - k] + arr[i];
            maxSum = Math.max(maxSum, windowSum);
        }
        
        return maxSum;
    }
    
    // Binary search
    function binarySearch(arr, target) {
        let left = 0, right = arr.length - 1;
        
        while (left <= right) {
            const mid = Math.floor((left + right) / 2);
            
            if (arr[mid] === target) {
                return mid;
            } else if (arr[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return -1;
    }
    
    // Dynamic programming - memoization
    function climbStairs(n, memo = {}) {
        if (n in memo) return memo[n];
        if (n <= 2) return n;
        
        memo[n] = climbStairs(n - 1, memo) + climbStairs(n - 2, memo);
        return memo[n];
    }
    
    // BFS for graphs/trees
    function bfs(graph, start) {
        const visited = new Set();
        const queue = [start];
        const result = [];
        
        while (queue.length > 0) {
            const node = queue.shift();
            
            if (!visited.has(node)) {
                visited.add(node);
                result.push(node);
                
                for (let neighbor of graph[node] || []) {
                    if (!visited.has(neighbor)) {
                        queue.push(neighbor);
                    }
                }
            }
        }
        
        return result;
    }
    
    // DFS for graphs/trees
    function dfs(graph, start, visited = new Set(), result = []) {
        visited.add(start);
        result.push(start);
        
        for (let neighbor of graph[start] || []) {
            if (!visited.has(neighbor)) {
                dfs(graph, neighbor, visited, result);
            }
        }
        
        return result;
    }
    
    // Backtracking example - generate all permutations
    function permutations(nums) {
        const result = [];
        const current = [];
        const used = new Array(nums.length).fill(false);
        
        function backtrack() {
            if (current.length === nums.length) {
                result.push([...current]); // Make a copy
                return;
            }
            
            for (let i = 0; i < nums.length; i++) {
                if (used[i]) continue;
                
                current.push(nums[i]);
                used[i] = true;
                
                backtrack();
                
                current.pop();
                used[i] = false;
            }
        }
        
        backtrack();
        return result;
    }
    
    // Prefix sum technique
    function prefixSum(arr) {
        const prefixes = [0];
        for (let i = 0; i < arr.length; i++) {
            prefixes.push(prefixes[prefixes.length - 1] + arr[i]);
        }
        return prefixes;
    }
    
    // Range sum using prefix sums
    function rangeSum(arr, left, right) {
        const prefixes = prefixSum(arr);
        return prefixes[right + 1] - prefixes[left];
    }
    
    // Union-Find (Disjoint Set) data structure
    class UnionFind {
        constructor(n) {
            this.parent = Array.from({length: n}, (_, i) => i);
            this.rank = new Array(n).fill(0);
        }
        
        find(x) {
            if (this.parent[x] !== x) {
                this.parent[x] = this.find(this.parent[x]); // Path compression
            }
            return this.parent[x];
        }
        
        union(x, y) {
            const rootX = this.find(x);
            const rootY = this.find(y);
            
            if (rootX !== rootY) {
                // Union by rank
                if (this.rank[rootX] < this.rank[rootY]) {
                    this.parent[rootX] = rootY;
                } else if (this.rank[rootX] > this.rank[rootY]) {
                    this.parent[rootY] = rootX;
                } else {
                    this.parent[rootY] = rootX;
                    this.rank[rootX]++;
                }
                return true;
            }
            return false;
        }
        
        connected(x, y) {
            return this.find(x) === this.find(y);
        }
    }
    ```

## JavaScript Event Loop and Asynchronous Behavior

### Understanding JavaScript's Single-Threaded Nature

21. **JavaScript is single-threaded but non-blocking** - How the event loop makes this possible.

    ```javascript
    // MISCONCEPTION: Many think this means JavaScript can't handle multiple things at once
    // REALITY: JavaScript has ONE main thread, but uses an event loop for concurrency
    
    // This is the main thread - it can only do one thing at a time
    console.log("1. Start");
    console.log("2. Middle");
    console.log("3. End");
    // Output: 1. Start, 2. Middle, 3. End (in order, blocking)
    
    // But with async operations, things get interesting...
    console.log("1. Start");
    setTimeout(() => console.log("2. Timeout"), 0);  // Even with 0ms delay!
    console.log("3. End");
    // Output: 1. Start, 3. End, 2. Timeout (non-blocking!)
    
    // WHY? Because setTimeout is handled by the browser/Node.js runtime,
    // not the JavaScript engine itself
    ```

### The Event Loop Architecture

22. **Event loop components** - Understanding the machinery behind asynchronous JavaScript.

    ```javascript
    // THE EVENT LOOP HAS SEVERAL COMPONENTS:
    
    // 1. CALL STACK - Where your code executes (LIFO - Last In, First Out)
    function first() {
        console.log("First function");
        second();
    }
    
    function second() {
        console.log("Second function");
        third();
    }
    
    function third() {
        console.log("Third function");
    }
    
    first();
    // Call stack execution:
    // [third] <- executed first (top of stack)
    // [second, third] 
    // [first, second, third] <- first() was called first (bottom of stack)
    
    // 2. WEB APIs / NODE.js APIs - Where async operations happen
    // - setTimeout/setInterval
    // - DOM events
    // - HTTP requests (fetch)
    // - File system operations (Node.js)
    // - Timers, I/O operations
    
    // 3. CALLBACK QUEUE (Task Queue) - Where completed async operations wait
    console.log("A");
    setTimeout(() => console.log("B"), 1000);
    setTimeout(() => console.log("C"), 500);
    console.log("D");
    // Output: A, D, C, B
    // B and C go to callback queue when their timers complete
    
    // 4. MICROTASK QUEUE - Higher priority than callback queue
    console.log("1");
    setTimeout(() => console.log("2"), 0);           // Callback queue
    Promise.resolve().then(() => console.log("3"));  // Microtask queue
    console.log("4");
    // Output: 1, 4, 3, 2
    // Microtasks (Promise) execute before callbacks (setTimeout)
    ```

### How the Event Loop Works

23. **Event loop execution order** - The exact mechanism of non-blocking behavior.

    ```javascript
    // THE EVENT LOOP ALGORITHM (simplified):
    // 1. Execute all synchronous code (populate call stack)
    // 2. When call stack is empty, check microtask queue
    // 3. Execute ALL microtasks
    // 4. Check callback queue, execute ONE callback
    // 5. Repeat from step 2
    
    // DETAILED EXAMPLE:
    console.log("=== Event Loop Demo ===");
    
    // Synchronous code - executes immediately
    console.log("1. Sync start");
    
    // setTimeout - goes to Web API, callback goes to callback queue when timer expires
    setTimeout(() => {
        console.log("2. setTimeout 1");
    }, 0);
    
    // Promise - .then() goes to microtask queue
    Promise.resolve().then(() => {
        console.log("3. Promise 1");
    });
    
    // Another setTimeout
    setTimeout(() => {
        console.log("4. setTimeout 2");
    }, 0);
    
    // Another Promise
    Promise.resolve().then(() => {
        console.log("5. Promise 2");
    });
    
    // More synchronous code
    console.log("6. Sync end");
    
    /* OUTPUT ORDER:
    === Event Loop Demo ===
    1. Sync start
    6. Sync end
    3. Promise 1
    5. Promise 2
    2. setTimeout 1
    4. setTimeout 2
    
    EXPLANATION:
    1. All sync code runs first (1, 6)
    2. Call stack empty, check microtask queue
    3. All Promises run (3, 5)
    4. Microtask queue empty, check callback queue
    5. setTimeout callbacks run one by one (2, 4)
    */
    ```

### Asynchronous Operations in Detail

24. **setTimeout and setInterval** - Timer-based asynchronous operations.

    ```javascript
    // setTimeout - runs once after delay
    console.log("Before timeout");
    setTimeout(() => {
        console.log("Timeout executed");
    }, 1000); // 1000ms = 1 second
    console.log("After timeout setup");
    
    // The delay is MINIMUM delay, not exact
    console.log("Start:", Date.now());
    setTimeout(() => {
        console.log("End:", Date.now());
    }, 100);
    
    // If main thread is busy, timeout will be delayed
    console.log("Blocking start");
    setTimeout(() => console.log("This will be delayed"), 0);
    
    // Block main thread for 2 seconds
    const start = Date.now();
    while (Date.now() - start < 2000) {
        // Busy wait - blocks everything
    }
    console.log("Blocking end");
    // The setTimeout callback will run AFTER the blocking code finishes
    
    // setInterval - runs repeatedly
    let count = 0;
    const intervalId = setInterval(() => {
        count++;
        console.log(`Interval execution ${count}`);
        
        if (count >= 3) {
            clearInterval(intervalId); // Stop the interval
        }
    }, 500);
    
    // IMPORTANT: setInterval doesn't wait for previous execution to finish
    // If the callback takes longer than the interval, they can overlap in the queue
    ```

### Promises and the Microtask Queue

25. **Promises and microtasks** - Higher priority asynchronous operations.

    ```javascript
    // Promises create microtasks, which have higher priority than callbacks
    
    console.log("=== Promise vs setTimeout ===");
    
    setTimeout(() => console.log("setTimeout"), 0);
    
    Promise.resolve()
        .then(() => console.log("Promise 1"))
        .then(() => console.log("Promise 2"));
        
    setTimeout(() => console.log("setTimeout 2"), 0);
    
    Promise.resolve().then(() => console.log("Promise 3"));
    
    /* OUTPUT:
    === Promise vs setTimeout ===
    Promise 1
    Promise 2
    Promise 3
    setTimeout
    setTimeout 2
    
    ALL Promises run before ANY setTimeout callbacks!
    */
    
    // Chained promises create new microtasks
    Promise.resolve()
        .then(() => {
            console.log("First then");
            return Promise.resolve();
        })
        .then(() => {
            console.log("Second then");
        });
    
    Promise.resolve().then(() => {
        console.log("Another promise");
    });
    
    // Promise constructor executes immediately (synchronous)
    console.log("Before Promise constructor");
    new Promise((resolve) => {
        console.log("Inside Promise constructor"); // Runs immediately!
        resolve();
    }).then(() => {
        console.log("Promise resolved"); // Goes to microtask queue
    });
    console.log("After Promise constructor");
    
    /* OUTPUT:
    Before Promise constructor
    Inside Promise constructor
    After Promise constructor
    Promise resolved
    */
    ```

### Async/Await and the Event Loop

26. **Async/await behavior** - Syntactic sugar over Promises with event loop implications.

    ```javascript
    // async/await is syntactic sugar over Promises
    // But it has specific event loop behavior
    
    async function asyncExample() {
        console.log("1. Start of async function");
        
        // await creates a microtask
        await Promise.resolve();
        console.log("2. After await");
        
        // This is equivalent to:
        // Promise.resolve().then(() => {
        //     console.log("2. After await");
        // });
    }
    
    console.log("0. Before calling async function");
    asyncExample();
    console.log("3. After calling async function");
    
    /* OUTPUT:
    0. Before calling async function
    1. Start of async function
    3. After calling async function
    2. After await
    */
    
    // Multiple awaits create multiple microtasks
    async function multipleAwaits() {
        console.log("A");
        await Promise.resolve();
        console.log("B");
        await Promise.resolve();
        console.log("C");
    }
    
    console.log("Before");
    multipleAwaits();
    console.log("After");
    
    /* OUTPUT:
    Before
    A
    After
    B
    C
    */
    
    // async/await with setTimeout
    async function mixedAsync() {
        console.log("1. Start");
        
        // This goes to callback queue
        setTimeout(() => console.log("2. setTimeout"), 0);
        
        // This goes to microtask queue
        await Promise.resolve();
        console.log("3. After await");
        
        // Another microtask
        await Promise.resolve();
        console.log("4. After second await");
    }
    
    mixedAsync();
    console.log("5. Sync end");
    
    /* OUTPUT:
    1. Start
    5. Sync end
    3. After await
    4. After second await
    2. setTimeout
    */
    ```

### Real-World Async Examples

27. **Practical async patterns** - How async operations work in real applications.

    ```javascript
    // HTTP Requests (non-blocking I/O)
    console.log("Starting HTTP request");
    
    fetch('https://api.example.com/data')
        .then(response => response.json())
        .then(data => {
            console.log("Data received:", data);
            // This runs when the HTTP request completes
            // Could be 100ms or 5 seconds later - doesn't block other code
        })
        .catch(error => {
            console.error("Request failed:", error);
        });
    
    console.log("HTTP request initiated, continuing execution");
    
    // File operations (Node.js example)
    const fs = require('fs'); // Only in Node.js
    
    console.log("Reading file...");
    fs.readFile('largefile.txt', 'utf8', (err, data) => {
        if (err) {
            console.error("Error reading file:", err);
        } else {
            console.log("File content length:", data.length);
        }
    });
    console.log("File read initiated, continuing...");
    
    // Multiple concurrent operations
    async function concurrentOperations() {
        console.log("Starting concurrent operations");
        
        // These ALL start at the same time (concurrent, not sequential)
        const promise1 = new Promise(resolve => 
            setTimeout(() => resolve("Operation 1"), 1000)
        );
        const promise2 = new Promise(resolve => 
            setTimeout(() => resolve("Operation 2"), 500)
        );
        const promise3 = new Promise(resolve => 
            setTimeout(() => resolve("Operation 3"), 750)
        );
        
        // Wait for all to complete
        const results = await Promise.all([promise1, promise2, promise3]);
        console.log("All operations completed:", results);
        // Takes ~1000ms total (not 1000+500+750 = 2250ms)
    }
    
    // Sequential vs Concurrent
    async function sequentialVsConcurrent() {
        console.log("=== Sequential (slow) ===");
        const start1 = Date.now();
        
        await new Promise(resolve => setTimeout(resolve, 1000));
        await new Promise(resolve => setTimeout(resolve, 1000));
        await new Promise(resolve => setTimeout(resolve, 1000));
        
        console.log("Sequential time:", Date.now() - start1); // ~3000ms
        
        console.log("=== Concurrent (fast) ===");
        const start2 = Date.now();
        
        await Promise.all([
            new Promise(resolve => setTimeout(resolve, 1000)),
            new Promise(resolve => setTimeout(resolve, 1000)),
            new Promise(resolve => setTimeout(resolve, 1000))
        ]);
        
        console.log("Concurrent time:", Date.now() - start2); // ~1000ms
    }
    ```

### Common Event Loop Gotchas

28. **Event loop pitfalls and misconceptions** - Common mistakes and how to avoid them.

    ```javascript
    // GOTCHA 1: setTimeout(0) doesn't mean immediate execution
    console.log("1");
    setTimeout(() => console.log("2"), 0); // Still goes to callback queue!
    console.log("3");
    // Output: 1, 3, 2
    
    // GOTCHA 2: Promises in loops
    console.log("=== Promise Loop Gotcha ===");
    
    // This creates all promises immediately
    for (let i = 0; i < 3; i++) {
        Promise.resolve().then(() => console.log("Promise", i));
    }
    // Output: Promise 3, Promise 3, Promise 3 (var) or Promise 0, Promise 1, Promise 2 (let)
    
    // GOTCHA 3: Mixed microtasks and callbacks
    setTimeout(() => console.log("setTimeout 1"), 0);
    Promise.resolve().then(() => {
        console.log("Promise 1");
        setTimeout(() => console.log("setTimeout 2"), 0);
    });
    Promise.resolve().then(() => console.log("Promise 2"));
    setTimeout(() => console.log("setTimeout 3"), 0);
    
    /* OUTPUT:
    Promise 1
    Promise 2
    setTimeout 1
    setTimeout 3
    setTimeout 2
    */
    
    // GOTCHA 4: Blocking the event loop
    // BAD: This blocks everything
    function badBlockingCode() {
        const start = Date.now();
        while (Date.now() - start < 5000) {
            // Blocks for 5 seconds - nothing else can run!
        }
    }
    
    // GOOD: Break up work into chunks
    function goodNonBlockingCode(iterations, callback) {
        let i = 0;
        
        function doChunk() {
            let count = 0;
            while (count < 1000 && i < iterations) {
                // Do some work
                i++;
                count++;
            }
            
            if (i < iterations) {
                setTimeout(doChunk, 0); // Let other code run
            } else {
                callback();
            }
        }
        
        doChunk();
    }
    
    // GOTCHA 5: Error handling in async code
    // Errors in callbacks don't bubble up
    try {
        setTimeout(() => {
            throw new Error("This won't be caught!");
        }, 0);
    } catch (e) {
        console.log("This won't run");
    }
    
    // Use Promises for proper error handling
    async function properErrorHandling() {
        try {
            await new Promise((resolve, reject) => {
                setTimeout(() => {
                    reject(new Error("This WILL be caught!"));
                }, 0);
            });
        } catch (e) {
            console.log("Caught error:", e.message);
        }
    }
    ```

### Event Loop Visualization

29. **Mental model for the event loop** - How to think about JavaScript's concurrency.

    ```javascript
    // VISUAL REPRESENTATION OF EVENT LOOP EXECUTION:
    
    /*
    
    ┌─────────────────────────┐
    │        Call Stack       │ ← JavaScript executes here (single-threaded)
    │  [currently executing]  │
    │  [function calls...]    │
    └─────────────────────────┘
                 │
                 ▼
    ┌─────────────────────────┐    ┌─────────────────────────┐
    │     Microtask Queue     │    │      Callback Queue     │
    │  [Promise.then]         │    │  [setTimeout]           │
    │  [queueMicrotask]       │ ◄──│  [setInterval]          │
    │  [async/await]          │    │  [DOM events]           │
    └─────────────────────────┘    │  [HTTP callbacks]       │
                 │                 └─────────────────────────┘
                 │                              │
                 ▼                              │
    ┌─────────────────────────┐                 │
    │      Event Loop         │                 │
    │                         │                 │
    │  1. Execute sync code   │                 │
    │  2. Process ALL         │ ◄───────────────│
    │     microtasks          │                 │
    │  3. Process ONE         │                 │
    │     callback            │ ◄───────────────┘
    │  4. Repeat              │
    └─────────────────────────┘
                 │
                 ▼
         ┌─────────────┐
         │  Web APIs   │ ← Browser/Node.js handles these
         │             │
         │ setTimeout  │
         │ fetch       │
         │ DOM events  │
         │ File I/O    │
         └─────────────┘
    
    */
    
    // STEP-BY-STEP EXECUTION EXAMPLE:
    function eventLoopDemo() {
        console.log("1. Sync"); // Call stack
        
        setTimeout(() => console.log("2. Callback"), 0); // → Web API → Callback Queue
        
        Promise.resolve().then(() => console.log("3. Microtask")); // → Microtask Queue
        
        console.log("4. Sync"); // Call stack
        
        // Event loop checks:
        // 1. Call stack empty? ✓
        // 2. Microtasks? ✓ Execute "3. Microtask"
        // 3. Microtasks empty? ✓
        // 4. Callbacks? ✓ Execute "2. Callback"
    }
    
    // WHY THIS MATTERS FOR PERFORMANCE:
    
    // Don't do this - blocks everything
    function blockingOperation() {
        for (let i = 0; i < 1000000000; i++) {
            // Synchronous work that blocks the thread
        }
    }
    
    // Do this instead - allows other code to run
    async function nonBlockingOperation() {
        for (let i = 0; i < 1000000000; i++) {
            if (i % 100000 === 0) {
                await new Promise(resolve => setTimeout(resolve, 0));
                // Yield control back to event loop every 100k iterations
            }
        }
    }
    
    // The key insight: JavaScript can handle thousands of async operations
    // because they don't block the main thread - they're handled by the runtime
    const promises = [];
    for (let i = 0; i < 10000; i++) {
        promises.push(
            new Promise(resolve => setTimeout(() => resolve(i), Math.random() * 1000))
        );
    }
    
    Promise.all(promises).then(results => {
        console.log(`Handled ${results.length} async operations!`);
        // This is possible because setTimeout is handled by Web APIs,
        // not the JavaScript engine itself
    });
    ```

This section explains exactly how JavaScript achieves non-blocking behavior despite being single-threaded. The event loop is the "magic" that makes it possible to handle thousands of concurrent operations without blocking the main execution thread. Understanding this is crucial for writing efficient JavaScript code and avoiding common pitfalls.

This comprehensive JavaScript guide covers all the major gotchas that trip up developers from Python and other languages, plus provides a solid foundation for data structures and algorithms in JavaScript. The examples show both common mistakes and correct approaches, making it practical for real-world use. 


### TypeScript in NPM Package Documentation

25. **Why Many NPM Packages Don't Use TypeScript in Docs**

    ```javascript
    // Traditional JSDoc comments are widely used instead of TypeScript
    /**
     * Greets a user with a custom message
     * @param {string} name - The name of the user
     * @param {number} [age] - Optional age of the user
     * @returns {string} The greeting message
     */
    function greet(name, age) {
        return age ? `Hello ${name}, you are ${age}!` : `Hello ${name}!`;
    }
    
    // JSDoc can even specify complex types
    /**
     * @typedef {Object} UserConfig
     * @property {string} username - The user's name
     * @property {number} [age] - Optional age
     * @property {'admin' | 'user'} role - User's role
     */
    
    /**
     * @param {UserConfig} config - User configuration object
     */
    function createUser(config) {
        // Function implementation
    }
    ```

**Key reasons packages often skip TypeScript in docs:**

1. **Backwards Compatibility:**
   - Many packages support older JavaScript environments
   - TypeScript is not available in all environments
   - JSDoc works with plain JavaScript and provides type hints

2. **Simplicity:**
   - JavaScript is the common denominator
   - No need for users to understand TypeScript
   - Simpler onboarding for new developers

3. **No Build Step Required:**
   - Documentation can be read directly from source
   - No need to compile TypeScript to see types
   - Works in any JavaScript environment

4. **IDE Support:**
   - Modern IDEs understand JSDoc comments
   - Provides autocomplete and type checking
   - Works even in JavaScript-only projects

**Best Practices for Non-TypeScript Documentation:**



