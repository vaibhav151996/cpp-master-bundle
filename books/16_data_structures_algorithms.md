# Book 16: Data Structures and Algorithms in C++

## The Complete Guide to DSA from Foundational to Competition-Level

---

### **Target Level:** Intermediate → Expert
### **Prerequisites:** Books 1–15
### **Learning Outcomes:**
By the end of this book, you will:
- Implement all fundamental data structures from scratch in C++
- Analyze time and space complexity with Big-O
- Master sorting, searching, graph, tree, and DP algorithms
- Solve classic algorithm problems using C++ STL
- Understand competitive programming techniques

---

## Chapter 1: Complexity Analysis

### 1.1 Big-O Notation

**Big-O** describes how an algorithm's runtime grows as the input size $n$ increases. It captures the **worst-case upper bound**, ignoring constants and lower-order terms.

| Notation | Name | Example | 1K items | 1M items |
|----------|------|---------|----------|----------|
| O(1) | Constant | Array access, hash table lookup | 1 op | 1 op |
| O(log n) | Logarithmic | Binary search | 10 ops | 20 ops |
| O(n) | Linear | Linear search, single loop | 1K ops | 1M ops |
| O(n log n) | Linearithmic | Merge sort, std::sort | 10K ops | 20M ops |
| O(n²) | Quadratic | Bubble sort, nested loops | 1M ops | 1T ops ✘ |
| O(2ⁿ) | Exponential | Recursive Fibonacci, power set | 10³⁰⁰ ✘ | ∞ ✘ |
| O(n!) | Factorial | Permutations, brute force TSP | ∞ ✘ | ∞ ✘ |

```
 Growth Rate Visualization (log scale):

 time
  │                                                    /  O(2^n)
  │                                                  /
  │                                              /
  │                                        . ··       O(n²)
  │                                   . ·
  │                              . ·
  │                        .·               _____------  O(n log n)
  │                    .·          ___----´
  │                .·     ___---´´                 _------  O(n)
  │            .·  ___--´´               ___---´´
  │        .·-´´´               ___---´´
  │   .·´´          ___---´´             ___....··········  O(log n)
  │.´    ___---´´        ___....······
  │--´´   ____....·····
  │───────────────────────────────────────────── O(1)
  └─────────────────────────────────────────────► n
```

**Practical guideline:** At ~$10^8$ operations per second on modern hardware:
- $O(n^2)$ is acceptable for $n \leq 10^4$
- $O(n \log n)$ is acceptable for $n \leq 10^6$
- $O(n)$ is acceptable for $n \leq 10^8$

### 1.2 Analysis Rules

```
1. Drop constants: O(2n) → O(n)
2. Drop lower terms: O(n² + n) → O(n²)
3. Nested loops multiply: for(for) → O(n × m)
4. Sequential blocks add: for + for → O(n + m)
5. Logarithms: halving → O(log n)
```

### 1.3 Amortized Analysis

Some operations are *usually* fast but *occasionally* expensive. **Amortized analysis** averages the cost over a sequence of operations.

**Example: `std::vector::push_back`**

```
Capacity: 4       push_back(5th element)
┌───┬───┬───┬───┐      Step 1: Allocate new buffer (capacity 8)
│ 1 │ 2 │ 3 │ 4 │      Step 2: Copy all 4 elements        O(n)
└───┴───┴───┴───┘      Step 3: Insert 5th element         O(1)
        │                Step 4: Free old buffer
        ▼
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 5 │   │   │   │  New capacity: 8
└───┴───┴───┴───┴───┴───┴───┴───┘

Next 3 push_backs: O(1) each (no resize needed)
Total cost for 8 push_backs: ~8 + 4 + 2 + 1 = ~15 copies
Amortized per push_back: 15/8 ≈ O(1)
```

The doubling strategy means resizes happen at sizes 1, 2, 4, 8, 16... The total copy cost across $n$ insertions is $1 + 2 + 4 + ... + n = 2n - 1 = O(n)$, so each individual insertion costs $O(1)$ amortized.

---

## Chapter 2: Arrays and Hashing

### 2.1 Two-Pointer Technique

The two-pointer technique uses two indices that move toward each other (or in the same direction) to solve problems in O(n) instead of O(n²).

```
Two Sum on sorted array [1, 3, 5, 7, 9], target = 12:

Step 1:  [1, 3, 5, 7, 9]    1+9=10 < 12  → move left →
          L              R

Step 2:  [1, 3, 5, 7, 9]    3+9=12 = 12  → FOUND!
             L           R

Key insight: if sum < target, we need a bigger number (move left pointer right)
             if sum > target, we need a smaller number (move right pointer left)
```

```cpp
// Two Sum (sorted array): O(n)
std::pair<int,int> twoSum(std::vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) return {left, right};
        else if (sum < target) ++left;
        else --right;
    }
    return {-1, -1};
}
```

### 2.2 Sliding Window

The sliding window technique maintains a "window" of k elements and slides it across the array, updating the result incrementally instead of recalculating from scratch.

```
Max sum subarray of size k=3 in [2, 1, 5, 1, 3, 2]:

Window 1:  [2, 1, 5] 1, 3, 2     sum = 8
            └───────┘
Window 2:   2 [1, 5, 1] 3, 2     sum = 8 - 2 + 1 = 7
               └───────┘
Window 3:   2, 1 [5, 1, 3] 2     sum = 7 - 1 + 3 = 9  ← MAX
                  └───────┘
Window 4:   2, 1, 5 [1, 3, 2]    sum = 9 - 5 + 2 = 6
                     └───────┘

Instead of recalculating the entire sum each time (O(k)),
we add the new element and subtract the old one (O(1)).
Total: O(n) instead of O(n×k).
```

```cpp
// Maximum sum subarray of size k: O(n)
int maxSumSubarray(const std::vector<int>& nums, int k) {
    int windowSum = 0;
    for (int i = 0; i < k; ++i) windowSum += nums[i];
    int maxSum = windowSum;
    for (int i = k; i < (int)nums.size(); ++i) {
        windowSum += nums[i] - nums[i - k];
        maxSum = std::max(maxSum, windowSum);
    }
    return maxSum;
}
```

### 2.3 Hash Map Patterns

```cpp
// Two Sum (unsorted): O(n)
std::vector<int> twoSum(const std::vector<int>& nums, int target) {
    std::unordered_map<int, int> seen;
    for (int i = 0; i < (int)nums.size(); ++i) {
        int complement = target - nums[i];
        if (seen.count(complement)) {
            return {seen[complement], i};
        }
        seen[nums[i]] = i;
    }
    return {};
}

// Frequency count:
std::unordered_map<char, int> freq;
for (char c : str) freq[c]++;
```

### 2.4 Prefix Sum

```cpp
// Range sum query: O(1) per query after O(n) preprocessing
std::vector<int> prefix(nums.size() + 1, 0);
for (int i = 0; i < (int)nums.size(); ++i)
    prefix[i + 1] = prefix[i] + nums[i];

// Sum of [l, r]:
int rangeSum = prefix[r + 1] - prefix[l];
```

---

## Chapter 3: Linked Lists

### 3.1 Implementation

A **linked list** stores elements as separate nodes connected by pointers. Unlike arrays, elements are NOT contiguous in memory — each node can be anywhere in the heap.

```
Array:    ┌───┬───┬───┬───┬───┐   Contiguous memory, O(1) access
          │ 1 │ 2 │ 3 │ 4 │ 5 │   O(n) insert/delete in middle
          └───┴───┴───┴───┴───┘

Linked    ┌───┬─┐  ┌───┬─┐  ┌───┬─┐
List:     │ 1 │─├─►│ 2 │─├─►│ 3 │─├─► NULL
          └───┴─┘  └───┴─┘  └───┴─┘
          Scattered memory, O(n) access
          O(1) insert/delete (given iterator)
```

**Reversing a linked list step-by-step:**

```
Original:   1 ─► 2 ─► 3 ─► 4 ─► NULL
            ^curr

Step 1:     1 ─► NULL    2 ─► 3 ─► 4
            ^prev       ^curr

Step 2:     2 ─► 1 ─► NULL    3 ─► 4
            ^prev            ^curr

Step 3:     3 ─► 2 ─► 1 ─► NULL    4
            ^prev               ^curr

Step 4:     4 ─► 3 ─► 2 ─► 1 ─► NULL
            ^prev               ^curr=NULL → done!
```

**Floyd’s Cycle Detection (Tortoise and Hare):**

```
       ┌────────────────┐     slow moves 1 step
       │                │     fast moves 2 steps
1 ─► 2 ─► 3 ─► 4 ─► 5 ─┘     If there’s a cycle,
                               fast will eventually
            S     F            "lap" slow and they’ll
            │     │            meet inside the cycle.
          slow   fast
```

```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

// Reverse linked list (iterative): O(n)
ListNode* reverse(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* curr = head;
    while (curr) {
        ListNode* next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}

// Detect cycle (Floyd's Tortoise and Hare): O(n)
bool hasCycle(ListNode* head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}

// Find middle: O(n)
ListNode* findMiddle(ListNode* head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}

// Merge two sorted lists: O(n + m)
ListNode* mergeSorted(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
    ListNode* tail = &dummy;
    while (l1 && l2) {
        if (l1->val <= l2->val) {
            tail->next = l1;
            l1 = l1->next;
        } else {
            tail->next = l2;
            l2 = l2->next;
        }
        tail = tail->next;
    }
    tail->next = l1 ? l1 : l2;
    return dummy.next;
}
```

---

## Chapter 4: Stacks and Queues

### 4.1 Stack Applications

```cpp
// Valid Parentheses: O(n)
bool isValid(const std::string& s) {
    std::stack<char> stk;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') {
            stk.push(c);
        } else {
            if (stk.empty()) return false;
            char top = stk.top(); stk.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    return stk.empty();
}

// Monotonic Stack — Next Greater Element: O(n)
std::vector<int> nextGreater(const std::vector<int>& nums) {
    int n = nums.size();
    std::vector<int> result(n, -1);
    std::stack<int> stk;  // Stack of indices
    for (int i = 0; i < n; ++i) {
        while (!stk.empty() && nums[stk.top()] < nums[i]) {
            result[stk.top()] = nums[i];
            stk.pop();
        }
        stk.push(i);
    }
    return result;
}
```

### 4.2 Queue Applications

```cpp
// BFS uses a queue (see Graph section)

// Deque — Sliding Window Maximum: O(n)
std::vector<int> maxSlidingWindow(const std::vector<int>& nums, int k) {
    std::deque<int> dq;  // Indices of candidates (decreasing order)
    std::vector<int> result;
    for (int i = 0; i < (int)nums.size(); ++i) {
        // Remove indices outside window
        while (!dq.empty() && dq.front() < i - k + 1)
            dq.pop_front();
        // Remove smaller elements (they can never be max)
        while (!dq.empty() && nums[dq.back()] < nums[i])
            dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1) result.push_back(nums[dq.front()]);
    }
    return result;
}
```

---

## Chapter 5: Trees

### 5.1 Binary Tree

A **binary tree** is a hierarchical structure where each node has at most two children. Trees are fundamental to many algorithms and data structures (BST, heaps, tries, segment trees).

```
             8            ← Root (depth 0)
           /   \
          3     10         ← depth 1  
         / \      \
        1   6      14      ← depth 2 (leaves have no children)
           / \    /
          4   7  13        ← depth 3

Height = 3 (longest path from root to leaf)
Nodes = 8
For a balanced tree: height ≈ log₂(n)
```

**Four traversal orders:**

```
         1                  Inorder   (L,Root,R): 4, 2, 5, 1, 3
        / \                 Preorder  (Root,L,R): 1, 2, 4, 5, 3
       2   3                Postorder (L,R,Root): 4, 5, 2, 3, 1
      / \                   Level-order (BFS):    1, 2, 3, 4, 5
     4   5

Mnemonic: "Inorder = Root in the middle"
          "Preorder = Root first"
          "Postorder = Root last"

  Inorder on a BST gives elements in SORTED order!
```

```cpp
struct TreeNode {
    int val;
    TreeNode *left, *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

// Traversals:
void inorder(TreeNode* root) {    // Left → Root → Right (sorted for BST)
    if (!root) return;
    inorder(root->left);
    std::cout << root->val << " ";
    inorder(root->right);
}

void preorder(TreeNode* root) {   // Root → Left → Right
    if (!root) return;
    std::cout << root->val << " ";
    preorder(root->left);
    preorder(root->right);
}

void postorder(TreeNode* root) {  // Left → Right → Root
    if (!root) return;
    postorder(root->left);
    postorder(root->right);
    std::cout << root->val << " ";
}

// Level-order (BFS): O(n)
void levelOrder(TreeNode* root) {
    if (!root) return;
    std::queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int size = q.size();
        for (int i = 0; i < size; ++i) {
            auto node = q.front(); q.pop();
            std::cout << node->val << " ";
            if (node->left) q.push(node->left);
            if (node->right) q.push(node->right);
        }
        std::cout << "\n";
    }
}

// Height: O(n)
int height(TreeNode* root) {
    if (!root) return 0;
    return 1 + std::max(height(root->left), height(root->right));
}
```

### 5.2 Binary Search Tree (BST)

**BST property:** For every node, all values in the left subtree are **less than** the node, and all values in the right subtree are **greater than** the node.

```
BST Insert(6):              BST Delete(8) with two children:

       8                           8           Find inorder successor
      / \                         / \          (smallest in right subtree)
     3   10         6 < 8       3   10         successor = 10
    / \    \        go left    / \    \        Replace 8 with 10,
   1   6    14                1   6    14      delete original 10
      /                          /
     4      6 > 3              4               Result:
            go right                              10
            6 = 6                                 / \
            FOUND spot!                          3   14
                                                / \
                                               1   6
                                                  /
                                                 4
```

**Performance depends on tree shape:**
- Balanced BST: O(log n) for search/insert/delete
- Skewed BST (insert sorted data): degenerates to O(n) — essentially a linked list
- Solution: self-balancing trees (AVL, Red-Black) guarantee O(log n)

```cpp
// Search: O(log n) average, O(n) worst
TreeNode* search(TreeNode* root, int val) {
    if (!root || root->val == val) return root;
    if (val < root->val) return search(root->left, val);
    return search(root->right, val);
}

// Insert: O(log n) average
TreeNode* insert(TreeNode* root, int val) {
    if (!root) return new TreeNode(val);
    if (val < root->val) root->left = insert(root->left, val);
    else root->right = insert(root->right, val);
    return root;
}

// Delete: O(log n) average
TreeNode* deleteNode(TreeNode* root, int val) {
    if (!root) return root;
    if (val < root->val) root->left = deleteNode(root->left, val);
    else if (val > root->val) root->right = deleteNode(root->right, val);
    else {
        if (!root->left) return root->right;
        if (!root->right) return root->left;
        // Node with two children: find inorder successor
        TreeNode* succ = root->right;
        while (succ->left) succ = succ->left;
        root->val = succ->val;
        root->right = deleteNode(root->right, succ->val);
    }
    return root;
}
```

### 5.3 Heap / Priority Queue

A **heap** is a complete binary tree stored in an array where every parent is greater (max-heap) or smaller (min-heap) than its children.

```
Max-Heap (tree view):          Array representation:
                                Index: 0  1  2  3  4  5  6
         90                     Value: 90 80 70 50 60 30 20
        /  \
      80    70                  Parent of i:    (i-1) / 2
     / \   / \                  Left child:     2*i + 1
   50  60 30  20                Right child:    2*i + 2

Insert(85):                    HeapifyUp:
  Add at end → swap up          85 > 70? swap!
  until heap property holds      85 > 90? no, stop.

         90                              90
        /  \                            /  \
      80    70      →                 80    85
     / \   / \                       / \   / \
   50  60 30  20                   50  60 30  20
              ^
             85 (added)            70 (swapped down)
```

```cpp
// Max-heap implementation:
class MaxHeap {
    std::vector<int> data;

    void heapifyUp(int i) {
        while (i > 0) {
            int parent = (i - 1) / 2;
            if (data[i] > data[parent]) {
                std::swap(data[i], data[parent]);
                i = parent;
            } else break;
        }
    }

    void heapifyDown(int i) {
        int n = data.size();
        while (2 * i + 1 < n) {
            int child = 2 * i + 1;
            if (child + 1 < n && data[child + 1] > data[child]) ++child;
            if (data[i] < data[child]) {
                std::swap(data[i], data[child]);
                i = child;
            } else break;
        }
    }

public:
    void push(int val) {
        data.push_back(val);
        heapifyUp(data.size() - 1);
    }

    int top() const { return data[0]; }

    void pop() {
        data[0] = data.back();
        data.pop_back();
        if (!data.empty()) heapifyDown(0);
    }
};
```

### 5.4 Trie (Prefix Tree)

```cpp
class Trie {
    struct Node {
        std::array<Node*, 26> children{};
        bool isEnd = false;
    };
    Node* root = new Node();

public:
    void insert(const std::string& word) {
        Node* node = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx])
                node->children[idx] = new Node();
            node = node->children[idx];
        }
        node->isEnd = true;
    }

    bool search(const std::string& word) {
        Node* node = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx]) return false;
            node = node->children[idx];
        }
        return node->isEnd;
    }

    bool startsWith(const std::string& prefix) {
        Node* node = root;
        for (char c : prefix) {
            int idx = c - 'a';
            if (!node->children[idx]) return false;
            node = node->children[idx];
        }
        return true;
    }
};
```

---

## Chapter 6: Graphs

### 6.1 Representations

**Adjacency List** — best for sparse graphs (most real-world graphs):

```
Graph:       0 ─── 1          Adjacency List:
             |   / |          0: [1, 2]
             |  /  |          1: [0, 2, 3]
             | /   |          2: [0, 1]
             2     3          3: [1]

Memory: O(V + E)    Lookup edge: O(degree)
```

**Adjacency Matrix** — best for dense graphs or when O(1) edge lookup needed:

```
             0  1  2  3       Matrix[i][j] = 1 means edge i→j
         0 [ 0  1  1  0 ]    
         1 [ 1  0  1  1 ]    Memory: O(V²)  — wasteful for sparse
         2 [ 1  1  0  0 ]    Lookup edge: O(1)
         3 [ 0  1  0  0 ]    
```

```cpp
// Adjacency List (most common):
std::vector<std::vector<int>> adj(n);
adj[0].push_back(1);  // Edge 0 → 1
adj[1].push_back(0);  // Undirected: add both

// Weighted:
std::vector<std::vector<std::pair<int,int>>> adj(n);
adj[0].push_back({1, 5});  // Edge 0 → 1, weight 5

// Adjacency Matrix:
std::vector<std::vector<int>> matrix(n, std::vector<int>(n, 0));
matrix[0][1] = 1;  // Edge 0 → 1
```

### 6.2 BFS — Breadth-First Search: O(V + E)

BFS explores the graph **level by level** using a queue. It finds the **shortest path** in unweighted graphs.

```
Graph:            BFS from node 0:          Queue trace:
    0                                       
   / \            Level 0: {0}              [0]
  1   2           Level 1: {1, 2}           [1, 2]
 / \   \          Level 2: {3, 4, 5}        [3, 4, 5]
3   4   5         Level 3: {6}              [6]
     \
      6           Visit order: 0, 1, 2, 3, 4, 5, 6
```

```cpp
void bfs(int start, const std::vector<std::vector<int>>& adj) {
    int n = adj.size();
    std::vector<bool> visited(n, false);
    std::queue<int> q;

    visited[start] = true;
    q.push(start);

    while (!q.empty()) {
        int u = q.front(); q.pop();
        std::cout << u << " ";
        for (int v : adj[u]) {
            if (!visited[v]) {
                visited[v] = true;
                q.push(v);
            }
        }
    }
}

// BFS gives shortest path in unweighted graphs
```

### 6.3 DFS — Depth-First Search: O(V + E)

DFS explores as **deep** as possible before backtracking. Uses a stack (or recursion).

```
Graph:            DFS from node 0:          Stack trace:
    0                                       
   / \            Visit 0 → go deep        [0]
  1   2           Visit 1 → go deep        [0, 1]
 / \   \          Visit 3 → no children     [0, 1, 3]
3   4   5         BACKTRACK to 1            [0, 1]
     \            Visit 4 → go deep        [0, 1, 4]
      6           Visit 6 → no children     [0, 1, 4, 6]
                  BACKTRACK to 0            [0]
                  Visit 2 → go deep        [0, 2]
                  Visit 5 → done            [0, 2, 5]

                  Visit order: 0, 1, 3, 4, 6, 2, 5
```

**BFS vs DFS Comparison:**

| Feature | BFS | DFS |
|---------|-----|-----|
| Data structure | Queue (FIFO) | Stack/Recursion (LIFO) |
| Shortest path? | Yes (unweighted) | No |
| Memory | O(width of graph) | O(depth of graph) |
| Use cases | Shortest path, level-order | Topological sort, cycle detection, connected components |
| On trees | Level-order traversal | Pre/In/Post-order traversal |

```cpp
void dfs(int u, const std::vector<std::vector<int>>& adj,
         std::vector<bool>& visited) {
    visited[u] = true;
    std::cout << u << " ";
    for (int v : adj[u]) {
        if (!visited[v]) {
            dfs(v, adj, visited);
        }
    }
}

// Applications: cycle detection, topological sort, connected components
```

### 6.4 Dijkstra's Shortest Path: O((V + E) log V)
Dijkstra’s algorithm finds the shortest path from a source to all other nodes in a **weighted graph with non-negative edges**. It uses a **min-heap** (priority queue) to always process the closest unvisited node.

```
Step-by-step example:

    A ──2── B          Distances from A:
    |       |          Start:  A=0  B=∞  C=∞  D=∞  E=∞
    4       3
    |       |          Process A: B=2, C=4
    C ──1── D          Process B: D=2+3=5
     \     /           Process C: D=min(5,4+1)=5, E=4+5=9
      5   2            Process D: E=min(9,5+2)=7
       \ /             Process E: done
        E
                       Final: A=0  B=2  C=4  D=5  E=7

                       Shortest path A→E: A→C→D→E (cost 7)
```

**Why Dijkstra fails with negative edges:** It assumes that once a node is processed (popped from the min-heap), its shortest distance is final. A negative edge could later provide a shorter path, violating this assumption. Use **Bellman-Ford** for graphs with negative edges.
```cpp
std::vector<int> dijkstra(int src, const std::vector<std::vector<std::pair<int,int>>>& adj) {
    int n = adj.size();
    std::vector<int> dist(n, INT_MAX);
    std::priority_queue<std::pair<int,int>,
        std::vector<std::pair<int,int>>,
        std::greater<>> pq;

    dist[src] = 0;
    pq.push({0, src});

    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;
        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

### 6.5 Topological Sort (DAG): O(V + E)

```cpp
// Kahn's Algorithm (BFS-based):
std::vector<int> topologicalSort(int n, const std::vector<std::vector<int>>& adj) {
    std::vector<int> inDegree(n, 0);
    for (int u = 0; u < n; ++u)
        for (int v : adj[u])
            ++inDegree[v];

    std::queue<int> q;
    for (int i = 0; i < n; ++i)
        if (inDegree[i] == 0) q.push(i);

    std::vector<int> order;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        order.push_back(u);
        for (int v : adj[u])
            if (--inDegree[v] == 0)
                q.push(v);
    }
    return order;  // If order.size() != n, cycle exists
}
```

### 6.6 Union-Find (Disjoint Set Union)

```cpp
class DSU {
    std::vector<int> parent, rank_;
public:
    DSU(int n) : parent(n), rank_(n, 0) {
        std::iota(parent.begin(), parent.end(), 0);
    }

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);  // Path compression
        return parent[x];
    }

    bool unite(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return false;
        if (rank_[px] < rank_[py]) std::swap(px, py);  // Union by rank
        parent[py] = px;
        if (rank_[px] == rank_[py]) ++rank_[px];
        return true;
    }
};
```

---

## Chapter 7: Sorting Algorithms

### 7.1 Summary Table

| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes |
| Radix Sort | O(d·(n+k)) | O(d·(n+k)) | O(d·(n+k)) | O(n+k) | Yes |

### 7.2 Merge Sort: O(n log n)

Merge Sort uses **divide and conquer**: split the array in half, sort each half recursively, then merge the two sorted halves.

```
Divide phase (top-down):              Merge phase (bottom-up):

  [38, 27, 43, 3, 9, 82, 10]         [3, 9, 10, 27, 38, 43, 82]
         /              \                     /              \
  [38, 27, 43]    [3, 9, 82, 10]       [27, 38, 43]    [3, 9, 10, 82]
    /      \         /       \           /      \         /       \
 [38,27] [43]    [3,9]   [82,10]      [27,38] [43]    [3,9]   [10,82]
  / \              / \                 / \              / \       
[38][27]         [3] [9]             [38][27]         [3] [9]

  Depth = log₂(n) levels
  Each level does O(n) work merging
  Total: O(n log n) — ALWAYS, regardless of input
```

**Merging two sorted arrays:**
```
[1, 3, 5]  [2, 4, 6]   →  Compare front elements, pick smaller
 ^           ^             Result: [1]
[1, 3, 5]  [2, 4, 6]   →  Result: [1, 2]
    ^        ^         
[1, 3, 5]  [2, 4, 6]   →  Result: [1, 2, 3]
    ^           ^      
...continue until both exhausted...  [1, 2, 3, 4, 5, 6]
```

```cpp
void mergeSort(std::vector<int>& arr, int left, int right) {
    if (left >= right) return;
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);

    std::vector<int> temp;
    int i = left, j = mid + 1;
    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) temp.push_back(arr[i++]);
        else temp.push_back(arr[j++]);
    }
    while (i <= mid) temp.push_back(arr[i++]);
    while (j <= right) temp.push_back(arr[j++]);

    for (int k = 0; k < (int)temp.size(); ++k)
        arr[left + k] = temp[k];
}
```

### 7.3 Quick Sort: O(n log n) average

Quick Sort picks a **pivot** element, partitions the array so all elements less than the pivot are on the left and all greater are on the right, then recursively sorts each side.

```
Partition example (pivot = last element = 4):

  [7, 2, 1, 8, 6, 3, 5, 4]   pivot = 4
   i                     p
   j

  j=7 > 4: skip           [7, 2, 1, 8, 6, 3, 5, 4]
  j=2 < 4: swap(arr[i],2) [2, 7, 1, 8, 6, 3, 5, 4]  i++
  j=1 < 4: swap(arr[i],1) [2, 1, 7, 8, 6, 3, 5, 4]  i++
  j=8 > 4: skip
  j=6 > 4: skip
  j=3 < 4: swap(arr[i],3) [2, 1, 3, 8, 6, 7, 5, 4]  i++
  j=5 > 4: skip
  Swap pivot into position: [2, 1, 3, 4, 6, 7, 5, 8]
                                    ↑
                              pivot in final position!
                 ┌────────┐  ┌────────┐
  All < 4:       [2, 1, 3]  4  [6, 7, 5, 8]  All > 4
                 Recurse!      Recurse!
```

**Why O(n²) worst case?** If the pivot is always the smallest or largest element (e.g., sorted input with last-element pivot), one partition has n−1 elements and the other has 0. This gives n levels instead of log n. **Fix:** use randomized pivot or median-of-three.

```cpp
int partition(std::vector<int>& arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; ++j) {
        if (arr[j] < pivot) {
            std::swap(arr[++i], arr[j]);
        }
    }
    std::swap(arr[i + 1], arr[high]);
    return i + 1;
}

void quickSort(std::vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}
```

---

## Chapter 8: Dynamic Programming

### 8.1 Pattern: Identify Overlapping Subproblems + Optimal Substructure

**Dynamic Programming** solves problems by breaking them into smaller overlapping subproblems. The key insight: if you’ve already solved a subproblem, don’t solve it again — store the result.

**Two approaches:**

```
Top-Down (Memoization):            Bottom-Up (Tabulation):

  Start from fib(5)                 Start from fib(0), build up
  │                                 
  fib(5)                            dp[0]=0  dp[1]=1
  ├─ fib(4)                         dp[2]=dp[0]+dp[1]=1
  │  ├─ fib(3)                      dp[3]=dp[1]+dp[2]=2
  │  │  ├─ fib(2) → computed       dp[4]=dp[2]+dp[3]=3
  │  │  └─ fib(1) → base case      dp[5]=dp[3]+dp[4]=5
  │  └─ fib(2) → CACHED!           
  └─ fib(3) → CACHED!               ┌──┬──┬──┬──┬──┬──┐
                                     │ 0│ 1│ 1│ 2│ 3│ 5│ ← dp table
  Recursive + cache                  └──┴──┴──┴──┴──┴──┘
  Natural for tree-shaped             Iterative, often faster
  subproblem graphs                   (no recursion overhead)
```

**How to recognize a DP problem:**
1. Can you break it into smaller subproblems? (optimal substructure)
2. Do the same subproblems appear multiple times? (overlapping subproblems)
3. Can you define a recurrence relation? (e.g., `dp[i] = dp[i-1] + dp[i-2]`)

```cpp
// Fibonacci — Top-Down (Memoization):
int fib(int n, std::vector<int>& memo) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    return memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
}

// Fibonacci — Bottom-Up (Tabulation):
int fib(int n) {
    if (n <= 1) return n;
    std::vector<int> dp(n + 1);
    dp[0] = 0; dp[1] = 1;
    for (int i = 2; i <= n; ++i)
        dp[i] = dp[i - 1] + dp[i - 2];
    return dp[n];
}

// Space-optimized:
int fib(int n) {
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; ++i) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

### 8.2 Classic DP Problems

**Longest Common Subsequence (LCS)** — DP Table Visualization:

```
Strings: a = "ABCB", b = "BDCB"

     ""  B  D  C  B
 ""   0  0  0  0  0
  A   0  0  0  0  0
  B   0  1  1  1  1     If a[i]==b[j]: dp[i][j] = dp[i-1][j-1] + 1
  C   0  1  1  2  2     Else:          dp[i][j] = max(dp[i-1][j], dp[i][j-1])
  B   0  1  1  2  3
                   ↑
              LCS length = 3 ("BCB")

Backtrack to find the actual subsequence:
Start at dp[4][4]=3, if chars match → include, go diagonal
                      if not → go direction of larger value
```

**0/1 Knapsack** — DP Table Visualization:

```
Items: weight=[1, 3, 4, 5], value=[1, 4, 5, 7], Capacity W=7

     W=0  1  2  3  4  5  6  7
  0:  0   0  0  0  0  0  0  0   (no items)
  1:  0   1  1  1  1  1  1  1   (item 1: w=1, v=1)
  2:  0   1  1  4  5  5  5  5   (item 2: w=3, v=4)
  3:  0   1  1  4  5  6  6  9   (item 3: w=4, v=5)
  4:  0   1  1  4  5  7  8  9   (item 4: w=5, v=7)
                              ↑
                   Maximum value = 9 (items 2+3: w=3+4=7, v=4+5=9)

For each cell: either SKIP item (take value from row above)
               or TAKE item (value + dp[i-1][w - weight[i]])
               Pick whichever is larger.
```

```cpp
// Longest Common Subsequence: O(n·m)
int lcs(const std::string& a, const std::string& b) {
    int n = a.size(), m = b.size();
    std::vector<std::vector<int>> dp(n + 1, std::vector<int>(m + 1, 0));
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= m; ++j)
            dp[i][j] = (a[i-1] == b[j-1])
                ? dp[i-1][j-1] + 1
                : std::max(dp[i-1][j], dp[i][j-1]);
    return dp[n][m];
}

// 0/1 Knapsack: O(n·W)
int knapsack(int W, const std::vector<int>& wt, const std::vector<int>& val) {
    int n = wt.size();
    std::vector<std::vector<int>> dp(n + 1, std::vector<int>(W + 1, 0));
    for (int i = 1; i <= n; ++i)
        for (int w = 0; w <= W; ++w) {
            dp[i][w] = dp[i-1][w];
            if (wt[i-1] <= w)
                dp[i][w] = std::max(dp[i][w], dp[i-1][w - wt[i-1]] + val[i-1]);
        }
    return dp[n][W];
}

// Longest Increasing Subsequence: O(n log n)
int lis(const std::vector<int>& nums) {
    std::vector<int> tails;
    for (int x : nums) {
        auto it = std::lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) tails.push_back(x);
        else *it = x;
    }
    return tails.size();
}
```

---

## Chapter 9: Backtracking

**Backtracking** builds solutions incrementally, abandoning a partial solution (“backtracking”) as soon as it determines the solution cannot be completed. Think of it as a **depth-first search on the decision tree**.

```
N-Queens Decision Tree (4×4 board):

                        [empty board]
                       /    |    \    \
               Q at     Q at   Q at   Q at
              (0,0)    (0,1)  (0,2)  (0,3)
              /  \       |       |      \
         Q at   ...    Q at    ...    Q at
        (1,2)         (1,3)          (1,0)
          |             |              |
        Q at        CONFLICT!       Q at
        (2,?)       backtrack!      (2,3)
        CONFLICT!                     |
        backtrack!                  Q at
                                   (3,1) ← SOLUTION!

Key idea: prune branches early when a constraint is violated,
rather than generating all permutations (n! vs pruned tree).
```

```cpp
// N-Queens: place N queens on N×N board with no attacks
void solveNQueens(int row, int n, std::vector<std::string>& board,
                  std::vector<std::vector<std::string>>& result,
                  std::vector<bool>& cols, std::vector<bool>& diag,
                  std::vector<bool>& antiDiag) {
    if (row == n) {
        result.push_back(board);
        return;
    }
    for (int col = 0; col < n; ++col) {
        if (cols[col] || diag[row - col + n] || antiDiag[row + col]) continue;
        board[row][col] = 'Q';
        cols[col] = diag[row - col + n] = antiDiag[row + col] = true;
        solveNQueens(row + 1, n, board, result, cols, diag, antiDiag);
        board[row][col] = '.';
        cols[col] = diag[row - col + n] = antiDiag[row + col] = false;
    }
}

// Subsets (Power Set):
void subsets(const std::vector<int>& nums, int idx,
             std::vector<int>& curr, std::vector<std::vector<int>>& result) {
    result.push_back(curr);
    for (int i = idx; i < (int)nums.size(); ++i) {
        curr.push_back(nums[i]);
        subsets(nums, i + 1, curr, result);
        curr.pop_back();
    }
}
```

---

## Chapter 10: Interview Questions

**Q1: Explain the difference between BFS and DFS.** — BFS explores level by level (queue, shortest path in unweighted graphs). DFS explores as deep as possible first (stack/recursion, topological sort, cycle detection).

**Q2: When would you use each sorting algorithm?** — `std::sort` (introsort) for general purpose. Merge sort when stability matters. Counting/radix sort for bounded integers. Insertion sort for nearly-sorted or small arrays.

**Q3: What is dynamic programming?** — Breaking a problem into overlapping subproblems with optimal substructure. Store results to avoid recomputation. Top-down (memoization) or bottom-up (tabulation).

**Q4: How does Union-Find work?** — Maintains disjoint sets with near O(1) operations via path compression and union by rank. Used for connected components, Kruskal's MST, cycle detection.

---

## Chapter 11: Practice & Mini Projects

### Project 1: Graph Visualizer — Implement BFS/DFS/Dijkstra with step-by-step output.
### Project 2: Auto-Complete System — Trie-based word suggestion engine.
### Project 3: Pathfinding (A*) — Implement A* on a grid with obstacles.

---

## Chapter 12: Mastery Checklist

- [ ] You can analyze time/space complexity of any algorithm
- [ ] You can implement linked list, stack, queue, tree, graph from scratch
- [ ] You know merge sort, quick sort, heap sort, and when to use each
- [ ] You can solve DP problems (knapsack, LCS, LIS, coin change)
- [ ] You understand BFS, DFS, Dijkstra, topological sort
- [ ] You can implement Union-Find, Trie, and priority queue
- [ ] You know two-pointer, sliding window, prefix sum techniques

---

*End of Book 16: Data Structures and Algorithms in C++*
