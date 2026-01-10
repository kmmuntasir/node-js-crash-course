# Module 5: Data Structures & Algorithms

## Algorithmic Foundations

### Time & Space Complexity

**Big O Notation**

Big O notation describes algorithm efficiency as input size grows. It focuses on worst-case scenario, ignoring constants and lower-order terms. Common complexities: O(1) constant, O(log n) logarithmic, O(n) linear, O(n log n) linearithmic, O(n²) quadratic. Understanding complexity helps choose optimal algorithms and predict performance at scale.

```javascript
// O(1) - Constant time
function getFirstElement(arr) {
  return arr[0]; // Always one operation
}

// O(n) - Linear time
function findElement(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) return i; // May check all elements
  }
  return -1;
}

// O(n²) - Quadratic time
function bubbleSort(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = 0; j < arr.length - i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
      }
    }
  }
  return arr;
}
```

**Space Complexity**

Space complexity measures memory usage as input grows. O(1) means constant extra space, O(n) means space grows with input. In-place algorithms use O(1) extra space. Recursive algorithms use O(n) space due to call stack. Choosing between time and space often involves trade-offs.

```javascript
// O(1) space - In-place
function reverseArray(arr) {
  let left = 0, right = arr.length - 1;
  while (left < right) {
    [arr[left], arr[right]] = [arr[right], arr[left]];
    left++;
    right--;
  }
  return arr;
}

// O(n) space - Creates new array
function reverseArrayNew(arr) {
  return arr.slice().reverse(); // Creates copy
}

// O(n) space - Recursive call stack
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // n stack frames
}
```

---

### Algorithm Design Paradigms

**Divide and Conquer**

Divide problem into smaller subproblems, solve independently, combine solutions. Examples: merge sort, quick sort, binary search. Time complexity often O(n log n). Reduces problem size exponentially each recursion level.

```javascript
// Binary search - O(log n)
function binarySearch(arr, target) {
  let left = 0, right = arr.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  
  return -1;
}
```

**Dynamic Programming**

Solves complex problems by breaking into overlapping subproblems and storing results. Two approaches: top-down (memoization) and bottom-up (tabulation). Key indicators: optimal substructure and overlapping subproblems.

```javascript
// Fibonacci with memoization - O(n) time, O(n) space
function fibonacci(n, memo = {}) {
  if (n in memo) return memo[n];
  if (n <= 1) return n;
  
  memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo);
  return memo[n];
}

// Fibonacci with tabulation - O(n) time, O(1) space
function fibonacciTabulation(n) {
  if (n <= 1) return n;
  
  let prev2 = 0, prev1 = 1;
  for (let i = 2; i <= n; i++) {
    const current = prev1 + prev2;
    prev2 = prev1;
    prev1 = current;
  }
  
  return prev1;
}
```

**Greedy Algorithms**

Makes locally optimal choices at each step, hoping for global optimum. Doesn't always work but efficient when greedy choice property holds. Examples: activity selection, Huffman coding, Dijkstra's algorithm.

```javascript
// Activity selection - Choose max non-overlapping activities
function activitySelection(activities) {
  // Sort by finish time
  activities.sort((a, b) => a.finish - b.finish);
  
  const selected = [activities[0]];
  let lastFinish = activities[0].finish;
  
  for (let i = 1; i < activities.length; i++) {
    if (activities[i].start >= lastFinish) {
      selected.push(activities[i]);
      lastFinish = activities[i].finish;
    }
  }
  
  return selected;
}
```

---

## Arrays & Strings

### Array Operations

**Two-Pointer Technique**

Uses two pointers to traverse array from different positions. Useful for sorted arrays, palindrome checking, and finding pairs. Reduces time complexity from O(n²) to O(n).

```javascript
// Two Sum II - Input array is sorted
function twoSum(numbers, target) {
  let left = 0, right = numbers.length - 1;
  
  while (left < right) {
    const sum = numbers[left] + numbers[right];
    
    if (sum === target) return [left + 1, right + 1];
    if (sum < target) left++;
    else right--;
  }
  
  return [];
}

// Remove duplicates from sorted array
function removeDuplicates(nums) {
  let slow = 0;
  
  for (let fast = 1; fast < nums.length; fast++) {
    if (nums[fast] !== nums[slow]) {
      slow++;
      nums[slow] = nums[fast];
    }
  }
  
  return slow + 1; // New length
}
```

**Sliding Window**

Maintains a window of elements that slides through array. Useful for subarray problems, finding maximum/minimum in subarrays. Time complexity O(n), space O(1) or O(k) for window.

```javascript
// Maximum subarray sum - Kadane's algorithm
function maxSubArray(nums) {
  let maxSum = nums[0];
  let currentSum = nums[0];
  
  for (let i = 1; i < nums.length; i++) {
    currentSum = Math.max(nums[i], currentSum + nums[i]);
    maxSum = Math.max(maxSum, currentSum);
  }
  
  return maxSum;
}

// Longest substring without repeating characters
function lengthOfLongestSubstring(s) {
  const charIndex = new Map();
  let left = 0, maxLength = 0;
  
  for (let right = 0; right < s.length; right++) {
    const char = s[right];
    
    if (charIndex.has(char) && charIndex.get(char) >= left) {
      left = charIndex.get(char) + 1;
    }
    
    charIndex.set(char, right);
    maxLength = Math.max(maxLength, right - left + 1);
  }
  
  return maxLength;
}
```

**Prefix Sums**

Precomputes cumulative sums for O(1) range sum queries. Useful for subarray sum problems. Build O(n), query O(1), update O(n).

```javascript
// Prefix sum array
function buildPrefixSum(nums) {
  const prefix = [0];
  for (let i = 0; i < nums.length; i++) {
    prefix.push(prefix[i] + nums[i]);
  }
  return prefix;
}

// Range sum query - O(1)
function rangeSum(prefix, left, right) {
  return prefix[right + 1] - prefix[left];
}

// Subarray sum equals k
function subarraySum(nums, k) {
  const prefix = [0];
  const sumCount = new Map([[0, 1]]);
  let count = 0;
  
  for (let i = 0; i < nums.length; i++) {
    prefix.push(prefix[i] + nums[i]);
    const target = prefix[i + 1] - k;
    
    if (sumCount.has(target)) {
      count += sumCount.get(target);
    }
    
    sumCount.set(prefix[i + 1], (sumCount.get(prefix[i + 1]) || 0) + 1);
  }
  
  return count;
}
```

---

### String Manipulation

**String Traversal**

Iterate through strings character by character. Use two pointers for palindrome checking, reversing. Time complexity O(n), space O(1) or O(n) depending on operations.

```javascript
// Valid palindrome
function isPalindrome(s) {
  let left = 0, right = s.length - 1;
  
  while (left < right) {
    // Skip non-alphanumeric
    while (left < right && !isAlphaNumeric(s[left])) left++;
    while (left < right && !isAlphaNumeric(s[right])) right--;
    
    if (s[left].toLowerCase() !== s[right].toLowerCase()) {
      return false;
    }
    
    left++;
    right--;
  }
  
  return true;
}

// Reverse string
function reverseString(s) {
  return s.split('').reverse().join('');
}

// Reverse words in string
function reverseWords(s) {
  return s.trim().split(/\s+/).reverse().join(' ');
}
```

**Substring Search**

Finding pattern within text. Naive approach O(n×m), KMP O(n+m), Rabin-Karp O(n+m) average. Use built-in methods for simple cases.

```javascript
// Valid anagram
function isAnagram(s, t) {
  if (s.length !== t.length) return false;
  
  const count = new Array(26).fill(0);
  
  for (let i = 0; i < s.length; i++) {
    count[s.charCodeAt(i) - 97]++;
    count[t.charCodeAt(i) - 97]--;
  }
  
  return count.every(c => c === 0);
}

// First unique character
function firstUniqChar(s) {
  const count = new Map();
  
  for (const char of s) {
    count.set(char, (count.get(char) || 0) + 1);
  }
  
  for (let i = 0; i < s.length; i++) {
    if (count.get(s[i]) === 1) return i;
  }
  
  return -1;
}
```

---

## Hash Maps & Sets

### Hash Map Fundamentals

**Hash Map Operations**

Hash maps provide O(1) average time for insert, delete, lookup. Use JavaScript `Map` or `Object`. Collision resolution handled internally. Good for counting, caching, and fast lookups.

```javascript
// Two Sum - Hash map approach
function twoSum(nums, target) {
  const seen = new Map();
  
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    
    if (seen.has(complement)) {
      return [seen.get(complement), i];
    }
    
    seen.set(nums[i], i);
  }
  
  return [];
}

// Group anagrams
function groupAnagrams(strs) {
  const groups = new Map();
  
  for (const str of strs) {
    const key = str.split('').sort().join('');
    
    if (!groups.has(key)) {
      groups.set(key, []);
    }
    
    groups.get(key).push(str);
  }
  
  return Array.from(groups.values());
}

// Top K frequent elements
function topKFrequent(nums, k) {
  const count = new Map();
  
  for (const num of nums) {
    count.set(num, (count.get(num) || 0) + 1);
  }
  
  return Array.from(count.entries())
    .sort((a, b) => b[1] - a[1])
    .slice(0, k)
    .map(entry => entry[0]);
}
```

**Collision Resolution**

When multiple keys hash to same index, handle with chaining (linked list) or open addressing (probing). Chaining is simpler, open addressing has better cache performance. JavaScript handles internally.

---

### Set Data Structure

**Set Operations**

Sets store unique elements with O(1) operations. Use `Set` in JavaScript. Useful for deduplication, membership testing, and set operations (union, intersection).

```javascript
// Contains duplicate
function containsDuplicate(nums) {
  const seen = new Set();
  
  for (const num of nums) {
    if (seen.has(num)) return true;
    seen.add(num);
  }
  
  return false;
}

// Single number - Every element appears twice except one
function singleNumber(nums) {
  let result = 0;
  
  for (const num of nums) {
    result ^= num; // XOR cancels pairs
  }
  
  return result;
}

// Intersection of two arrays
function intersection(nums1, nums2) {
  const set1 = new Set(nums1);
  const result = new Set();
  
  for (const num of nums2) {
    if (set1.has(num)) {
      result.add(num);
    }
  }
  
  return Array.from(result);
}
```

---

## Linked Lists

### Linked List Fundamentals

**Linked List Structure**

Nodes contain data and reference to next node. Singly linked: one direction. Doubly linked: both directions. O(1) insertion/deletion at known position, O(n) search. No random access.

```javascript
class ListNode {
  constructor(val = 0, next = null) {
    this.val = val;
    this.next = next;
  }
}

// Create linked list: 1 -> 2 -> 3 -> null
const head = new ListNode(1);
head.next = new ListNode(2);
head.next.next = new ListNode(3);

// Traverse linked list
function traverse(head) {
  const values = [];
  let current = head;
  
  while (current !== null) {
    values.push(current.val);
    current = current.next;
  }
  
  return values;
}
```

**Node Operations**

Insertion and deletion require updating pointers. For singly linked list, need previous node reference. For doubly linked, can go backwards. Handle edge cases (head, tail, null).

```javascript
// Insert at beginning
function insertAtHead(head, val) {
  const newNode = new ListNode(val);
  newNode.next = head;
  return newNode; // New head
}

// Delete node (given reference to node)
function deleteNode(node) {
  if (node.next === null) return; // Can't delete last node
  
  node.val = node.next.val;
  node.next = node.next.next;
}
```

---

### Linked List Algorithms

**Two-Pointer Technique**

Fast and slow pointers for cycle detection, finding middle. Fast moves 2 steps, slow moves 1 step. When they meet, there's a cycle. O(n) time, O(1) space.

```javascript
// Detect cycle - Floyd's algorithm
function hasCycle(head) {
  let slow = head, fast = head;
  
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    
    if (slow === fast) return true;
  }
  
  return false;
}

// Find middle node
function middleNode(head) {
  let slow = head, fast = head;
  
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  
  return slow;
}
```

**List Reversal**

Reverse by changing pointers iteratively or recursively. Iterative: O(n) time, O(1) space. Recursive: O(n) time, O(n) space (call stack).

```javascript
// Reverse linked list - Iterative
function reverseList(head) {
  let prev = null, current = head;
  
  while (current !== null) {
    const next = current.next;
    current.next = prev;
    prev = current;
    current = next;
  }
  
  return prev; // New head
}

// Reverse linked list - Recursive
function reverseListRecursive(head) {
  if (head === null || head.next === null) return head;
  
  const newHead = reverseListRecursive(head.next);
  head.next.next = head;
  head.next = null;
  
  return newHead;
}
```

**Palindrome Check**

Use fast/slow to find middle, reverse second half, compare. O(n) time, O(1) space. Or use stack: O(n) time, O(n) space.

```javascript
function isPalindrome(head) {
  if (!head || !head.next) return true;
  
  // Find middle
  let slow = head, fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  
  // Reverse second half
  let secondHalf = reverseList(slow);
  let firstHalf = head;
  
  // Compare
  let result = true;
  while (secondHalf) {
    if (firstHalf.val !== secondHalf.val) {
      result = false;
      break;
    }
    firstHalf = firstHalf.next;
    secondHalf = secondHalf.next;
  }
  
  return result;
}
```

---

## Stacks & Queues

### Stack Data Structure

**LIFO Principle**

Last In, First Out. Operations: push (add), pop (remove), peek (view top). Used for function calls, undo operations, expression evaluation. Array-based: push/pop O(1).

```javascript
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
}

// Valid parentheses
function isValid(s) {
  const stack = [];
  const pairs = { '(': ')', '[': ']', '{': '}' };
  
  for (const char of s) {
    if (pairs[char]) {
      stack.push(char);
    } else if (Object.values(pairs).includes(char)) {
      if (stack.length === 0 || pairs[stack.pop()] !== char) {
        return false;
      }
    }
  }
  
  return stack.length === 0;
}
```

**Min Stack**

Track minimum element with O(1) time. Use auxiliary stack to store minimums. When pushing, compare and push new minimum if smaller.

```javascript
class MinStack {
  constructor() {
    this.stack = [];
    this.minStack = [];
  }
  
  push(val) {
    this.stack.push(val);
    
    if (this.minStack.length === 0 || val <= this.minStack[this.minStack.length - 1]) {
      this.minStack.push(val);
    }
  }
  
  pop() {
    const val = this.stack.pop();
    
    if (val === this.minStack[this.minStack.length - 1]) {
      this.minStack.pop();
    }
  }
  
  top() {
    return this.stack[this.stack.length - 1];
  }
  
  getMin() {
    return this.minStack[this.minStack.length - 1];
  }
}
```

---

### Queue Data Structure

**FIFO Principle**

First In, First Out. Operations: enqueue (add), dequeue (remove), peek (view front). Used for BFS, task scheduling, buffering. Array-based: shift O(n), use circular buffer or linked list for O(1).

```javascript
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
  
  peek() {
    return this.items[0];
  }
  
  isEmpty() {
    return this.items.length === 0;
  }
}

// Implement queue using stacks
class MyQueue {
  constructor() {
    this.inStack = [];
    this.outStack = [];
  }
  
  push(x) {
    this.inStack.push(x);
  }
  
  pop() {
    this._transfer();
    return this.outStack.pop();
  }
  
  peek() {
    this._transfer();
    return this.outStack[this.outStack.length - 1];
  }
  
  _transfer() {
    if (this.outStack.length === 0) {
      while (this.inStack.length > 0) {
        this.outStack.push(this.inStack.pop());
      }
    }
  }
}
```

**Priority Queue**

Elements have priority, highest priority dequeued first. Implementation: heap (binary heap) O(log n) operations. Use for Dijkstra's, Huffman coding, task scheduling.

```javascript
class MinHeap {
  constructor() {
    this.heap = [];
  }
  
  push(val) {
    this.heap.push(val);
    this._bubbleUp(this.heap.length - 1);
  }
  
  pop() {
    const min = this.heap[0];
    const last = this.heap.pop();
    
    if (this.heap.length > 0) {
      this.heap[0] = last;
      this._bubbleDown(0);
    }
    
    return min;
  }
  
  _bubbleUp(index) {
    while (index > 0) {
      const parent = Math.floor((index - 1) / 2);
      
      if (this.heap[parent] <= this.heap[index]) break;
      
      [this.heap[parent], this.heap[index]] = [this.heap[index], this.heap[parent]];
      index = parent;
    }
  }
  
  _bubbleDown(index) {
    while (true) {
      const left = 2 * index + 1;
      const right = 2 * index + 2;
      let smallest = index;
      
      if (left < this.heap.length && this.heap[left] < this.heap[smallest]) {
        smallest = left;
      }
      
      if (right < this.heap.length && this.heap[right] < this.heap[smallest]) {
        smallest = right;
      }
      
      if (smallest === index) break;
      
      [this.heap[smallest], this.heap[index]] = [this.heap[index], this.heap[smallest]];
      index = smallest;
    }
  }
}
```

---

## Trees

### Tree Fundamentals

**Tree Terminology**

Root: top node. Node: element with data. Leaf: node with no children. Depth: distance from root. Height: max distance to leaf. Binary tree: each node has max 2 children. BST: left < root < right.

```javascript
class TreeNode {
  constructor(val = 0, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}

// Create BST: 5 -> 3,7 -> 2,4,6,8
const root = new TreeNode(5);
root.left = new TreeNode(3);
root.right = new TreeNode(7);
root.left.left = new TreeNode(2);
root.left.right = new TreeNode(4);
root.right.left = new TreeNode(6);
root.right.right = new TreeNode(8);
```

---

### Tree Traversals

**Depth-First Search (DFS)**

Explores as deep as possible before backtracking. Pre-order: root, left, right. In-order: left, root, right (BST gives sorted). Post-order: left, right, root. Recursive or iterative with stack.

```javascript
// Pre-order traversal
function preorderTraversal(root) {
  const result = [];
  
  function traverse(node) {
    if (!node) return;
    
    result.push(node.val); // Root
    traverse(node.left);  // Left
    traverse(node.right); // Right
  }
  
  traverse(root);
  return result;
}

// In-order traversal
function inorderTraversal(root) {
  const result = [];
  
  function traverse(node) {
    if (!node) return;
    
    traverse(node.left);  // Left
    result.push(node.val); // Root
    traverse(node.right); // Right
  }
  
  traverse(root);
  return result;
}

// Post-order traversal
function postorderTraversal(root) {
  const result = [];
  
  function traverse(node) {
    if (!node) return;
    
    traverse(node.left);  // Left
    traverse(node.right); // Right
    result.push(node.val); // Root
  }
  
  traverse(root);
  return result;
}
```

**Breadth-First Search (BFS)**

Explores level by level. Uses queue. Level-order: all nodes at depth d before depth d+1. Shortest path in unweighted graphs. O(n) time, O(w) space where w is max width.

```javascript
// Level-order traversal
function levelOrder(root) {
  if (!root) return [];
  
  const result = [];
  const queue = [root];
  
  while (queue.length > 0) {
    const level = [];
    const levelSize = queue.length;
    
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      level.push(node.val);
      
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    
    result.push(level);
  }
  
  return result;
}
```

---

### Binary Search Tree Operations

**BST Insertion**

Insert maintaining BST property: left < root < right. Start at root, compare, go left or right, insert at null. O(h) time where h is height. Balanced: O(log n), unbalanced: O(n).

```javascript
function insertIntoBST(root, val) {
  if (!root) return new TreeNode(val);
  
  if (val < root.val) {
    root.left = insertIntoBST(root.left, val);
  } else {
    root.right = insertIntoBST(root.right, val);
  }
  
  return root;
}
```

**BST Search**

Find value by comparing with nodes. Go left if smaller, right if larger. O(h) time. Returns node or null if not found.

```javascript
function searchBST(root, val) {
  if (!root || root.val === val) return root;
  
  if (val < root.val) {
    return searchBST(root.left, val);
  } else {
    return searchBST(root.right, val);
  }
}
```

**BST Validation**

Check if tree satisfies BST property. All left subtree values < node < all right subtree values. Use min/max bounds. O(n) time.

```javascript
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;
  
  if (root.val <= min || root.val >= max) return false;
  
  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}
```

---

### Tree Algorithms

**Maximum Depth**

Height of tree: longest path from root to leaf. DFS: recursive depth calculation. BFS: level counting. O(n) time.

```javascript
// DFS approach
function maxDepth(root) {
  if (!root) return 0;
  
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}

// BFS approach
function maxDepthBFS(root) {
  if (!root) return 0;
  
  let depth = 0;
  const queue = [root];
  
  while (queue.length > 0) {
    depth++;
    const levelSize = queue.length;
    
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
  }
  
  return depth;
}
```

**Invert Binary Tree**

Swap left and right children recursively. Mirror image. O(n) time, O(h) space (call stack).

```javascript
function invertTree(root) {
  if (!root) return null;
  
  [root.left, root.right] = [root.right, root.left];
  invertTree(root.left);
  invertTree(root.right);
  
  return root;
}
```

**Lowest Common Ancestor**

Find deepest node that is ancestor of both nodes. For BST: first node where nodes are in different subtrees. O(n) time.

```javascript
// For BST
function lowestCommonAncestorBST(root, p, q) {
  if (p.val < root.val && q.val < root.val) {
    return lowestCommonAncestorBST(root.left, p, q);
  }
  
  if (p.val > root.val && q.val > root.val) {
    return lowestCommonAncestorBST(root.right, p, q);
  }
  
  return root;
}

// For any binary tree
function lowestCommonAncestor(root, p, q) {
  if (!root || root === p || root === q) return root;
  
  const left = lowestCommonAncestor(root.left, p, q);
  const right = lowestCommonAncestor(root.right, p, q);
  
  if (left && right) return root;
  return left || right;
}
```

---

## Graphs

### Graph Fundamentals

**Graph Terminology**

Vertex (node): element. Edge: connection between vertices. Degree: number of edges. Path: sequence of vertices. Cycle: path starting and ending at same vertex. Directed: edges have direction. Weighted: edges have values.

**Graph Representations**

Adjacency list: array of lists/sets. Space O(V+E), efficient for sparse graphs. Adjacency matrix: 2D array. Space O(V²), O(1) edge lookup.

```javascript
// Adjacency list representation
class Graph {
  constructor() {
    this.adjacencyList = new Map();
  }
  
  addVertex(vertex) {
    if (!this.adjacencyList.has(vertex)) {
      this.adjacencyList.set(vertex, []);
    }
  }
  
  addEdge(v1, v2, directed = false) {
    this.adjacencyList.get(v1).push(v2);
    if (!directed) {
      this.adjacencyList.get(v2).push(v1);
    }
  }
}

const graph = new Graph();
graph.addVertex('A');
graph.addVertex('B');
graph.addVertex('C');
graph.addEdge('A', 'B');
graph.addEdge('B', 'C');
graph.addEdge('A', 'C');
```

---

### Graph Traversals

**Depth-First Search (DFS)**

Explore as deep as possible, backtrack. Use stack or recursion. Mark visited to avoid cycles. O(V+E) time. Applications: cycle detection, pathfinding, topological sort.

```javascript
function dfs(graph, start, visited = new Set()) {
  visited.add(start);
  console.log(start); // Process node
  
  for (const neighbor of graph.adjacencyList.get(start) || []) {
    if (!visited.has(neighbor)) {
      dfs(graph, neighbor, visited);
    }
  }
}

// Detect cycle in undirected graph
function hasCycle(graph) {
  const visited = new Set();
  
  function dfs(node, parent) {
    visited.add(node);
    
    for (const neighbor of graph.adjacencyList.get(node) || []) {
      if (!visited.has(neighbor)) {
        if (dfs(neighbor, node)) return true;
      } else if (neighbor !== parent) {
        return true; // Back edge found
      }
    }
    
    return false;
  }
  
  for (const vertex of graph.adjacencyList.keys()) {
    if (!visited.has(vertex)) {
      if (dfs(vertex, null)) return true;
    }
  }
  
  return false;
}
```

**Breadth-First Search (BFS)**

Explore level by level. Use queue. Shortest path in unweighted graphs. O(V+E) time. Applications: shortest path, connected components.

```javascript
function bfs(graph, start) {
  const visited = new Set();
  const queue = [start];
  visited.add(start);
  
  while (queue.length > 0) {
    const node = queue.shift();
    console.log(node); // Process node
    
    for (const neighbor of graph.adjacencyList.get(node) || []) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push(neighbor);
      }
    }
  }
}

// Number of islands (grid BFS)
function numIslands(grid) {
  if (!grid.length) return 0;
  
  let count = 0;
  const rows = grid.length;
  const cols = grid[0].length;
  
  function bfs(r, c) {
    const queue = [[r, c]];
    grid[r][c] = '0';
    
    while (queue.length > 0) {
      const [row, col] = queue.shift();
      const directions = [[-1,0], [1,0], [0,-1], [0,1]];
      
      for (const [dr, dc] of directions) {
        const nr = row + dr, nc = col + dc;
        
        if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] === '1') {
          grid[nr][nc] = '0';
          queue.push([nr, nc]);
        }
      }
    }
  }
  
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === '1') {
        count++;
        bfs(r, c);
      }
    }
  }
  
  return count;
}
```

---

### Graph Algorithms

**Topological Sort**

Order vertices in DAG such that every edge goes from earlier to later. Use DFS with post-order or Kahn's algorithm (BFS). O(V+E) time. Applications: task scheduling, dependency resolution.

```javascript
// Course schedule - can you finish all courses?
function canFinish(numCourses, prerequisites) {
  const adj = Array.from({ length: numCourses }, () => []);
  const inDegree = new Array(numCourses).fill(0);
  
  for (const [course, prereq] of prerequisites) {
    adj[prereq].push(course);
    inDegree[course]++;
  }
  
  const queue = [];
  for (let i = 0; i < numCourses; i++) {
    if (inDegree[i] === 0) queue.push(i);
  }
  
  let completed = 0;
  while (queue.length > 0) {
    const course = queue.shift();
    completed++;
    
    for (const next of adj[course]) {
      inDegree[next]--;
      if (inDegree[next] === 0) {
        queue.push(next);
      }
    }
  }
  
  return completed === numCourses;
}
```

**Connected Components**

Groups of vertices where each is reachable from others. Use BFS/DFS from each unvisited vertex. Count components. O(V+E) time.

```javascript
function countComponents(n, edges) {
  const adj = Array.from({ length: n }, () => []);
  
  for (const [a, b] of edges) {
    adj[a].push(b);
    adj[b].push(a);
  }
  
  const visited = new Set();
  let count = 0;
  
  function dfs(node) {
    visited.add(node);
    for (const neighbor of adj[node]) {
      if (!visited.has(neighbor)) {
        dfs(neighbor);
      }
    }
  }
  
  for (let i = 0; i < n; i++) {
    if (!visited.has(i)) {
      count++;
      dfs(i);
    }
  }
  
  return count;
}
```

---

## Sorting Algorithms

### Basic Sorting

**Bubble Sort**

Repeatedly swap adjacent elements if wrong order. O(n²) time, O(1) space. Simple but inefficient. Use only for educational purposes.

```javascript
function bubbleSort(arr) {
  const n = arr.length;
  
  for (let i = 0; i < n - 1; i++) {
    let swapped = false;
    
    for (let j = 0; j < n - i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        swapped = true;
      }
    }
    
    if (!swapped) break; // Already sorted
  }
  
  return arr;
}
```

**Selection Sort**

Find minimum in unsorted portion, swap with first. O(n²) time, O(1) space. Minimal swaps. Good when swap cost is high.

```javascript
function selectionSort(arr) {
  const n = arr.length;
  
  for (let i = 0; i < n - 1; i++) {
    let minIdx = i;
    
    for (let j = i + 1; j < n; j++) {
      if (arr[j] < arr[minIdx]) {
        minIdx = j;
      }
    }
    
    [arr[i], arr[minIdx]] = [arr[minIdx], arr[i]];
  }
  
  return arr;
}
```

**Insertion Sort**

Build sorted array one element at a time. Insert each element into correct position. O(n²) worst, O(n) best (already sorted). Good for small arrays or nearly sorted.

```javascript
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    const key = arr[i];
    let j = i - 1;
    
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    
    arr[j + 1] = key;
  }
  
  return arr;
}
```

---

### Advanced Sorting

**Merge Sort**

Divide array in half, sort each, merge. O(n log n) time, O(n) space. Stable sort. Good for linked lists and external sorting.

```javascript
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
      result.push(left[i++]);
    } else {
      result.push(right[j++]);
    }
  }
  
  return result.concat(left.slice(i)).concat(right.slice(j));
}
```

**Quick Sort**

Pick pivot, partition around it, recursively sort partitions. O(n log n) average, O(n²) worst. O(log n) space (in-place). Fast in practice, good cache performance.

```javascript
function quickSort(arr) {
  if (arr.length <= 1) return arr;
  
  const pivot = arr[Math.floor(arr.length / 2)];
  const left = [], middle = [], right = [];
  
  for (const val of arr) {
    if (val < pivot) left.push(val);
    else if (val === pivot) middle.push(val);
    else right.push(val);
  }
  
  return [...quickSort(left), ...middle, ...quickSort(right)];
}

// In-place quicksort
function quickSortInPlace(arr, low = 0, high = arr.length - 1) {
  if (low < high) {
    const pivotIdx = partition(arr, low, high);
    quickSortInPlace(arr, low, pivotIdx - 1);
    quickSortInPlace(arr, pivotIdx + 1, high);
  }
  return arr;
}

function partition(arr, low, high) {
  const pivot = arr[high];
  let i = low - 1;
  
  for (let j = low; j < high; j++) {
    if (arr[j] < pivot) {
      i++;
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
  }
  
  [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
  return i + 1;
}
```

**Heap Sort**

Build max heap, repeatedly extract max. O(n log n) time, O(1) space. Not stable. Good when memory is constrained.

```javascript
function heapSort(arr) {
  const n = arr.length;
  
  // Build max heap
  for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
    heapify(arr, n, i);
  }
  
  // Extract elements one by one
  for (let i = n - 1; i > 0; i--) {
    [arr[0], arr[i]] = [arr[i], arr[0]];
    heapify(arr, i, 0);
  }
  
  return arr;
}

function heapify(arr, n, i) {
  let largest = i;
  const left = 2 * i + 1;
  const right = 2 * i + 2;
  
  if (left < n && arr[left] > arr[largest]) {
    largest = left;
  }
  
  if (right < n && arr[right] > arr[largest]) {
    largest = right;
  }
  
  if (largest !== i) {
    [arr[i], arr[largest]] = [arr[largest], arr[i]];
    heapify(arr, n, largest);
  }
}
```

---

## Recursion

### Recursion Fundamentals

**Recursive Thinking**

Break problem into smaller instances of same problem. Must have base case to terminate. Each call adds stack frame. Risk: stack overflow for deep recursion.

```javascript
// Factorial
function factorial(n) {
  if (n <= 1) return 1; // Base case
  return n * factorial(n - 1); // Recursive case
}

// Fibonacci (inefficient - O(2^n))
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// Fibonacci with memoization - O(n)
function fibonacciMemo(n, memo = {}) {
  if (n in memo) return memo[n];
  if (n <= 1) return n;
  
  memo[n] = fibonacciMemo(n - 1, memo) + fibonacciMemo(n - 2, memo);
  return memo[n];
}
```

**Backtracking**

Systematically explore all possibilities by trying candidates and abandoning if invalid. Used for permutations, combinations, Sudoku, N-Queens. Time: exponential.

```javascript
// Generate all subsets
function subsets(nums) {
  const result = [];
  
  function backtrack(start, current) {
    result.push([...current]);
    
    for (let i = start; i < nums.length; i++) {
      current.push(nums[i]);
      backtrack(i + 1, current);
      current.pop();
    }
  }
  
  backtrack(0, []);
  return result;
}

// Generate all permutations
function permute(nums) {
  const result = [];
  const used = new Array(nums.length).fill(false);
  
  function backtrack(current) {
    if (current.length === nums.length) {
      result.push([...current]);
      return;
    }
    
    for (let i = 0; i < nums.length; i++) {
      if (used[i]) continue;
      
      used[i] = true;
      current.push(nums[i]);
      backtrack(current);
      current.pop();
      used[i] = false;
    }
  }
  
  backtrack([]);
  return result;
}
```

---

## Advanced Topics

### Dynamic Programming

**Top-Down (Memoization)**

Recursive with caching. Store results of subproblems. Check cache before computing. Natural for problems with optimal substructure.

```javascript
// Climbing stairs - can take 1 or 2 steps
function climbStairs(n, memo = {}) {
  if (n in memo) return memo[n];
  if (n <= 2) return n;
  
  memo[n] = climbStairs(n - 1, memo) + climbStairs(n - 2, memo);
  return memo[n];
}

// Coin change - minimum coins to make amount
function coinChange(coins, amount) {
  const memo = new Map();
  
  function dfs(remaining) {
    if (remaining === 0) return 0;
    if (remaining < 0) return Infinity;
    if (memo.has(remaining)) return memo.get(remaining);
    
    let minCoins = Infinity;
    for (const coin of coins) {
      minCoins = Math.min(minCoins, 1 + dfs(remaining - coin));
    }
    
    memo.set(remaining, minCoins);
    return minCoins;
  }
  
  const result = dfs(amount);
  return result === Infinity ? -1 : result;
}
```

**Bottom-Up (Tabulation)**

Iterative, build table from small to large. No recursion overhead. Often more space-efficient.

```javascript
// Climbing stairs - tabulation
function climbStairsTabulation(n) {
  if (n <= 2) return n;
  
  const dp = new Array(n + 1);
  dp[1] = 1;
  dp[2] = 2;
  
  for (let i = 3; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2];
  }
  
  return dp[n];
}

// Space-optimized climbing stairs
function climbStairsOptimized(n) {
  if (n <= 2) return n;
  
  let prev2 = 1, prev1 = 2;
  
  for (let i = 3; i <= n; i++) {
    const current = prev1 + prev2;
    prev2 = prev1;
    prev1 = current;
  }
  
  return prev1;
}
```

---

### Binary Search

**Binary Search Algorithm**

Search in sorted array by repeatedly dividing search space. O(log n) time. Requires sorted input. Can be adapted for various problems.

```javascript
function binarySearch(nums, target) {
  let left = 0, right = nums.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (nums[mid] === target) return mid;
    if (nums[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  
  return -1;
}

// Search in rotated sorted array
function searchRotated(nums, target) {
  let left = 0, right = nums.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (nums[mid] === target) return mid;
    
    // Left half is sorted
    if (nums[left] <= nums[mid]) {
      if (target >= nums[left] && target < nums[mid]) {
        right = mid - 1;
      } else {
        left = mid + 1;
      }
    }
    // Right half is sorted
    else {
      if (target > nums[mid] && target <= nums[right]) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
  }
  
  return -1;
}
```

**Binary Search on Answer**

Search for optimal value in range when problem can be verified. Examples: find minimum capacity, find smallest divisor. O(log range) × O(verification).

```javascript
// Koko eating bananas - find minimum eating speed
function minEatingSpeed(piles, h) {
  let left = 1, right = Math.max(...piles);
  
  function canFinish(speed) {
    let hours = 0;
    for (const pile of piles) {
      hours += Math.ceil(pile / speed);
    }
    return hours <= h;
  }
  
  while (left < right) {
    const mid = Math.floor((left + right) / 2);
    
    if (canFinish(mid)) {
      right = mid;
    } else {
      left = mid + 1;
    }
  }
  
  return left;
}
```

---

## Quick Reference

**Complexity Cheat Sheet**

| Data Structure | Access | Search | Insert | Delete | Space |
|--------------|---------|---------|---------|---------|--------|
| Array | O(1) | O(n) | O(n) | O(n) | O(n) |
| Linked List | O(n) | O(n) | O(1) | O(1) | O(n) |
| Hash Map | O(1)* | O(1)* | O(1)* | O(1)* | O(n) |
| Stack | O(1) | O(n) | O(1) | O(1) | O(n) |
| Queue | O(1) | O(n) | O(1) | O(1) | O(n) |
| BST (avg) | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |

*Average case, worst case O(n)

**Sorting Algorithm Comparison**

| Algorithm | Time (Best) | Time (Avg) | Time (Worst) | Space | Stable |
|-----------|--------------|-------------|---------------|--------|--------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |

**Pattern Recognition Guide**

| Pattern | When to Use | Key Technique |
|---------|--------------|---------------|
| Two Pointers | Sorted arrays, pairs, palindrome | Left/right or slow/fast pointers |
| Sliding Window | Subarray/substring problems | Maintain window, expand/contract |
| Hash Map | Fast lookups, counting, deduplication | Map/Set for O(1) operations |
| DFS | Tree traversal, path finding, backtracking | Stack or recursion |
| BFS | Shortest path, level-order | Queue |
| Divide & Conquer | Can split into independent subproblems | Merge sort, quick sort, binary search |
| DP | Overlapping subproblems, optimal substructure | Memoization or tabulation |
| Greedy | Local optimal → global optimal | Activity selection, Huffman coding |

**Algorithm Selection Decision Tree**

```
Need to search?
    ↓
Is data sorted?
    ↓ YES → Binary Search (O(log n))
    ↓ NO → Linear Search (O(n))
    ↓
Need to sort?
    ↓
Small array (< 50)?
    ↓ YES → Insertion Sort (simple, fast for small)
    ↓ NO → Merge Sort (stable) or Quick Sort (fast)
    ↓
Memory constrained?
    ↓ YES → Heap Sort (O(1) space)
    ↓ NO → Merge Sort or Quick Sort
```

**Data Structure Selection Decision Tree**

```
Need to store data?
    ↓
Need fast lookups?
    ↓ YES → Hash Map/Set (O(1))
    ↓ NO → Array/Linked List
    ↓
Need ordered data?
    ↓ YES → BST or Sorted Array
    ↓ NO → Unordered Collection
    ↓
Need frequent insertions/deletions at ends?
    ↓ YES → Linked List
    ↓ NO → Array
```

**Daily Practice Checklist**

- [ ] Solve 2 medium LeetCode problems daily
- [ ] Focus on one pattern per week
- [ ] Time yourself (30-45 min per problem)
- [ ] Review and optimize solutions
- [ ] Implement from memory after solving
- [ ] Practice whiteboard explanations
- [ ] Track patterns in problem notebook
