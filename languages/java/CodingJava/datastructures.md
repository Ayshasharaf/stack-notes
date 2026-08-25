# Java Data Structures — Complete Cheat Sheet
---

## The Big Picture — Which One Do I Use?

```
I need to...                                   Use this
─────────────────────────────────────────────────────────
Store items, fixed size, raw speed         →   int[] / String[]
Store items, size unknown upfront          →   ArrayList<T>
key → value lookup in O(1)                →   HashMap<K, V>
"Is X in here?" check in O(1)             →   HashSet<T>
Last In First Out (undo, DFS, brackets)   →   ArrayDeque<T> as Stack
First In First Out (BFS, level order)     →   LinkedList<T> as Queue
Always get the min/max element            →   PriorityQueue<T>
Count letter frequencies                  →   int[26]
```

---

## 1. Array — `int[]`

### What it is
A row of slots in memory, numbered from 0. **Fixed size** — you decide the size when creating it and can never change it.

```
Index:  [0]  [1]  [2]  [3]  [4]  [5]
        ┌────┬────┬────┬────┬────┬────┐
        │  2 │  7 │  4 │  1 │  9 │  3 │
        └────┴────┴────┴────┴────┴────┘
          ↑                        ↑
        arr[0] = 2              arr[5] = 3
```

### Syntax
```java
// Create
int[] arr = new int[6];           // 6 slots, all 0
int[] arr = {2, 7, 4, 1, 9, 3};  // create with values

// Access & update
int x = arr[2];    // x = 4
arr[0] = 99;       // change slot 0

// Size — NOTE: .length not .size()
int n = arr.length;

// Sort
Arrays.sort(arr);

// Loop
for (int i = 0; i < arr.length; i++) { }
for (int num : arr) { }           // for-each
```

### Complexity
| Operation | Time |
|-----------|------|
| Access `arr[i]` | O(1) |
| Update `arr[i]` | O(1) |
| Search (unsorted) | O(n) |
| Sort | O(n log n) |

### ⚡ The Letter Frequency Trick
```java
// Count how many times each letter appears — use int[26]
int[] freq = new int[26];
for (char c : s.toCharArray()) {
    freq[c - 'a']++;        // 'a'→0, 'b'→1, 'z'→25
}
// freq[0] = count of 'a'
// freq[1] = count of 'b'  ...etc
```
> Faster than HashMap for lowercase letter problems. Used in Valid Anagram, Group Anagrams.

### Use when
- You know the exact size upfront
- Sliding window / two pointers (most efficient)
- Counting letter frequencies (`int[26]`)
- Prefix sums
- You need raw speed with no overhead

### ❌ Don't use when
- Size is unknown — use ArrayList instead
- You need `.add()` or `.remove()` — arrays are fixed

---

## 2. ArrayList — `List<Integer>`

### What it is
A resizable array. Exactly like `int[]` but **grows automatically** when full. Internally doubles in size when needed — you never worry about capacity.

```
After add(10), add(20), add(30):

size=3         reserved capacity=6
┌────┬────┬────╔════╦════╦════╗
│ 10 │ 20 │ 30 ║ .. ║ .. ║ .. ║
└────┴────┴────╚════╩════╩════╝
  [0]  [1]  [2]    (empty slots)
```

### Syntax
```java
// Create
List<Integer> list = new ArrayList<>();

// Add
list.add(10);           // adds to end
list.add(0, 99);        // insert at index 0 — O(n)!

// Access & update
int x = list.get(0);   // NOT list[0]
list.set(0, 99);        // update index 0

// Size — NOTE: .size() not .length
int n = list.size();

// Remove
list.remove(0);         // remove by index
list.remove(Integer.valueOf(10)); // remove by value

// Sort
Collections.sort(list);

// Loop
for (int num : list) { }
```

### Complexity
| Operation | Time |
|-----------|------|
| `get(i)` | O(1) |
| `add()` at end | O(1) average |
| `add(i, x)` in middle | O(n) |
| `remove(i)` | O(n) |
| `size()` | O(1) |

### Array vs ArrayList — Never Confuse These Again
```
                  int[]               List<Integer>
                ─────────────────────────────────────
Size            Fixed                 Dynamic
Access          arr[i]                list.get(i)
Update          arr[i] = x            list.set(i, x)
Length/Size     arr.length            list.size()
Sort            Arrays.sort(arr)      Collections.sort(list)
Add element     ❌ can't              list.add(x)
Type            int, char, etc.       Integer, String, etc.
```

### Use when
- Building a result to return from a function
- BFS/DFS result collection
- Path tracking in backtracking
- Size is unknown

---

## 3. HashMap — `Map<K, V>`

### What it is
A **key → value** store. You give it a key, it gives back the value instantly — O(1) — no matter how many entries.

```
put("cat", 3)
put("dog", 1)
put("ant", 7)

  key       value
┌───────┐   ┌───┐
│ "cat" │ → │ 3 │
├───────┤   ├───┤
│ "dog" │ → │ 1 │
├───────┤   ├───┤
│ "ant" │ → │ 7 │
└───────┘   └───┘

get("cat") → 3  (O(1))
get("xyz") → null
```

### Syntax
```java
// Create
Map<String, Integer> map = new HashMap<>();

// Add / update
map.put("cat", 3);

// Get
map.get("cat");                      // returns 3
map.get("xyz");                      // returns null

// Safe get — returns default if key missing
map.getOrDefault("xyz", 0);          // returns 0

// Check if key exists
map.containsKey("cat");              // true
map.containsValue(3);                // true

// Remove
map.remove("cat");

// Size
map.size();

// Iterate over entries
for (Map.Entry<String, Integer> e : map.entrySet()) {
    System.out.println(e.getKey() + " → " + e.getValue());
}

// Iterate keys only
for (String key : map.keySet()) { }

// Iterate values only
for (int val : map.values()) { }
```

### ⚡ The Frequency Pattern — Use This Constantly
```java
// Count how many times each element appears
Map<Integer, Integer> freq = new HashMap<>();
for (int num : nums) {
    freq.put(num, freq.getOrDefault(num, 0) + 1);
}
// freq.get(5) = how many times 5 appears
```

### Complexity
| Operation | Time |
|-----------|------|
| `put(k, v)` | O(1) average |
| `get(k)` | O(1) average |
| `containsKey(k)` | O(1) average |
| `remove(k)` | O(1) average |

### Use when
- Counting element frequency (most common HashMap use)
- Storing index alongside value (Two Sum)
- Grouping items by a computed key (Group Anagrams)
- Memoization / caching in DP
- Any "what was the last time I saw X?" problem

---

## 4. HashSet — `Set<Integer>`

### What it is
Like HashMap but **keys only — no values**. Every element is unique. The power: `.contains()` is O(1).

```
add(5), add(12), add(3), add(9)
add(5)  ← ignored! already there

┌────┬────┬────┬────┐
│  5 │ 12 │  3 │  9 │
└────┴────┴────┴────┘

contains(3)  → true   O(1)
contains(99) → false  O(1)
```

### Syntax
```java
// Create
Set<Integer> set = new HashSet<>();

// Add
set.add(5);
set.add(5);      // silently ignored — already there

// Check membership
set.contains(5); // true — O(1)
set.contains(99);// false — O(1)

// Remove
set.remove(5);

// Size
set.size();

// Build from array quickly
for (int n : nums) set.add(n);

// Loop
for (int n : set) { }
```

### Complexity
| Operation | Time |
|-----------|------|
| `add(x)` | O(1) average |
| `contains(x)` | O(1) average |
| `remove(x)` | O(1) average |

### HashMap vs HashSet — Side by Side
```
                  HashMap<K,V>          HashSet<T>
                ─────────────────────────────────────
Stores          key + value             key only
Question it     "What is the count      "Is X in
answers         of X?"                  here?"
Example         "cat" → 3              {"cat", "dog"}
Use for         frequency, grouping     membership, visited
```

### Use when
- Longest Consecutive Sequence 
- "Contains Duplicate" — add all, check size vs length
- "Visited" set in BFS/DFS
- Remove duplicates from a list
- Two-pointer problems where you need O(1) lookup

---

## 5. Stack — `Deque<Integer>`

### What it is
**Last In, First Out (LIFO)**. You can only add/remove from the top. Like a stack of plates — last plate placed is first taken.

```
        push(3) → top
        ┌─────┐
        │  3  │  ← push/pop here ONLY
        ├─────┤
        │  2  │
        ├─────┤
        │  1  │  ← bottom (first in, last out)
        └─────┘

pop() → removes 3, top becomes 2
peek() → sees 2, doesn't remove
```

### Syntax
```java
// Use ArrayDeque, NOT the old Stack class
Deque<Integer> stack = new ArrayDeque<>();

// Push to top
stack.push(1);
stack.push(2);
stack.push(3);

// Pop from top (removes)
stack.pop();     // returns 3, top is now 2

// Peek at top (doesn't remove)
stack.peek();    // returns 2

// Check empty
stack.isEmpty(); // false

// Size
stack.size();
```


### Complexity
| Operation | Time |
|-----------|------|
| `push(x)` | O(1) |
| `pop()` | O(1) |
| `peek()` | O(1) |

### Use when
- Valid Parentheses — classic: push open brackets, pop on close
- DFS iterative (instead of recursion)
- Undo/redo operations
- Monotonic stack problems (Next Greater Element)
- Evaluating math expressions

---

## 6. Queue — `Queue<Integer>`

### What it is
**First In, First Out (FIFO)**. Like a real queue/line — first person in is first served. Add to the back, remove from the front.

```
add(A), add(B), add(C), add(D)

front                        back
  ↓                            ↓
┌────┬────┬────┬────┐
│  A │  B │  C │  D │
└────┴────┴────┴────┘
  ↑ poll() takes from here
  ↑ offer() adds to the other end

poll() → returns A, front is now B
peek() → sees B, doesn't remove
```

### Syntax
```java
// Create
Queue<Integer> queue = new LinkedList<>();

// Add to back
queue.offer(1);
queue.offer(2);
queue.offer(3);

// Remove from front
queue.poll();    // returns 1

// Peek at front (doesn't remove)
queue.peek();    // returns 2

// Check empty
queue.isEmpty();

// Size
queue.size();
```

### Complexity
| Operation | Time |
|-----------|------|
| `offer(x)` | O(1) |
| `poll()` | O(1) |
| `peek()` | O(1) |

### Use when
- **BFS — always use a Queue for BFS**
- Level-order tree traversal
- Processing nodes layer by layer
- Any "process in order of arrival" problem

---

## 7. PriorityQueue — `PriorityQueue<Integer>`

### What it is
A queue that **always gives you the smallest (or largest) element first** — regardless of the order you added items. Backed by a heap.

```
add(5), add(1), add(3), add(9), add(2)

      Min-Heap tree:
          ┌───┐
          │ 1 │  ← poll() always returns this
         ┌┘   └┐
       ┌─┴─┐ ┌─┴─┐
       │ 2 │ │ 3 │
      ┌┘   └┐
    ┌─┴─┐ ┌─┴─┐
    │ 9 │ │ 5 │

poll() → 1 (minimum)
poll() → 2 (next minimum)
```

### Syntax
```java
// Min-heap — smallest element comes out first (DEFAULT)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Max-heap — largest element comes out first
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

// Add
minHeap.add(5);
minHeap.add(1);
minHeap.add(3);

// Get min (removes it)
minHeap.poll();   // returns 1

// Peek at min (doesn't remove)
minHeap.peek();   // returns 3 (new min after 1 removed)

// Size
minHeap.size();
```

### Complexity
| Operation | Time |
|-----------|------|
| `add(x)` | O(log n) |
| `poll()` — remove min | O(log n) |
| `peek()` — see min | O(1) |

### Use when
- Top K Frequent Elements
- Kth Largest / Smallest in array
- Merge K sorted lists
- Dijkstra's shortest path
- Any "always grab the best remaining" problem

---

## 8. LinkedList (for Queue/Deque only)

### What it is
Nodes connected by pointers. In LeetCode, you'll use it as a **Queue** (most common), rarely as a standalone structure.

```
head                        tail
 ↓                            ↓
┌───┐    ┌───┐    ┌───┐    ┌───┐
│ 1 │ →  │ 2 │ →  │ 3 │ →  │ 4 │ → null
└───┘    └───┘    └───┘    └───┘
```

> In Java, use `LinkedList<>` to back a `Queue<>`. Don't overthink it.

```java
Queue<Integer> bfsQueue = new LinkedList<>();  // standard BFS setup
```

---

## Full Syntax Comparison — All at Once

```java
// ─── ARRAYS ───────────────────────────────────────────
int[] arr = {1, 2, 3};
arr[0];          // access
arr.length;      // size
Arrays.sort(arr);

// ─── ARRAYLIST ────────────────────────────────────────
List<Integer> list = new ArrayList<>();
list.add(x);
list.get(0);     // access
list.size();     // size
Collections.sort(list);

// ─── HASHMAP ──────────────────────────────────────────
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.get("a");                      // 1
map.getOrDefault("x", 0);         // 0 if missing
map.containsKey("a");              // true

// ─── HASHSET ──────────────────────────────────────────
Set<Integer> set = new HashSet<>();
set.add(5);
set.contains(5);  // true

// ─── STACK ────────────────────────────────────────────
Deque<Integer> stack = new ArrayDeque<>();
stack.push(x);
stack.pop();
stack.peek();

// ─── QUEUE ────────────────────────────────────────────
Queue<Integer> queue = new LinkedList<>();
queue.offer(x);
queue.poll();
queue.peek();

// ─── PRIORITY QUEUE ───────────────────────────────────
PriorityQueue<Integer> minH = new PriorityQueue<>();
PriorityQueue<Integer> maxH = new PriorityQueue<>(Collections.reverseOrder());
minH.add(x);
minH.poll();      // removes min
minH.peek();      // sees min
```

---

## The Most Common LeetCode Patterns

### Pattern 1 — Frequency counting
```java
// "How many times does each element appear?"
Map<Integer, Integer> freq = new HashMap<>();
for (int num : nums)
    freq.put(num, freq.getOrDefault(num, 0) + 1);
```

### Pattern 2 — Seen before (complement lookup)
```java
// Two Sum — "Have I seen the complement?"
Map<Integer, Integer> seen = new HashMap<>();  // value → index
for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i];
    if (seen.containsKey(complement))
        return new int[]{seen.get(complement), i};
    seen.put(nums[i], i);
}
```

### Pattern 3 — Sequence detection
```java
// Longest Consecutive Sequence
Set<Integer> set = new HashSet<>();
for (int n : nums) set.add(n);
int longest = 0;
for (int n : nums) {
    if (!set.contains(n - 1)) {      // only start from sequence begins
        int len = 1;
        while (set.contains(n + len)) len++;
        longest = Math.max(longest, len);
    }
}
```

### Pattern 4 — Grouping by key
```java
// Group Anagrams — sort each word to get its key
Map<String, List<String>> groups = new HashMap<>();
for (String word : words) {
    char[] chars = word.toCharArray();
    Arrays.sort(chars);
    String key = new String(chars);               // "eat","tea","ate" → "aet"
    groups.computeIfAbsent(key, k -> new ArrayList<>()).add(word);
}
```

### Pattern 5 — BFS with Queue
```java
// Level-order traversal / shortest path
Queue<Integer> q = new LinkedList<>();
Set<Integer> visited = new HashSet<>();
q.offer(start);
visited.add(start);
while (!q.isEmpty()) {
    int node = q.poll();
    for (int neighbor : graph.get(node)) {
        if (!visited.contains(neighbor)) {
            visited.add(neighbor);
            q.offer(neighbor);
        }
    }
}
```

---

## Common Mistakes — Never Make These Again

```
❌ arr.size()         ✅ arr.length
❌ list.length        ✅ list.size()
❌ list[0]            ✅ list.get(0)
❌ new Stack<>()      ✅ new ArrayDeque<>()
❌ map.get(k) == 0    ✅ map.getOrDefault(k, 0) == 0
❌ Arrays.sort(list)  ✅ Collections.sort(list)
❌ "a" == "b"         ✅ "a".equals("b")   ← ALWAYS .equals() for Strings
```

---

## Interview Communication Template

For every problem, say this out loud before coding:
```
1. Brute force: [state it — always]
2. Bottleneck: [what's repeated / slow?]
3. Data structure fix: [what removes that bottleneck?]
4. Time: O(?) | Space: O(?)
```

Always add a comment at the bottom of your solution:
```java
// Time: O(n)
// Space: O(n)
```

---

