## 递归

```python
# Python
def recursion(level, param1, param2, ...):     
    # recursion terminator     
    if level > MAX_LEVEL: 	  
        # process_result 	   
        return     
    # process logic in current level     
    process(level, data...)     
    # drill down     
    self.recursion(level + 1, p1, ...)     
    # reverse the current level status if needed
```

```java
// Java
public void recur(int level, int param) {   
	// terminator   
    if (level > MAX_LEVEL) {     
        // process result     
        return;   
    }  
    // process current logic   
    process(level, param);   
    // drill down   
    recur(level: level + 1, newParam);   
    // restore current status 
```

```c
// C/C++
void recursion(int level, int param) {   
    // recursion terminator  
    if (level > MAX_LEVEL) {     
        // process result     
        return ;   
    }  
    // process current logic   
    process(level, param);  
    // drill down   
    recursion(level + 1, param);  
    // reverse the current level status if needed}
```

```javascript
// JavaScript
const recursion = (level, params) =>{   
    // recursion terminator   
    if(level > MAX_LEVEL) {     
        // process_result     
        return    
    }   
    // process current level   
    process(level, params)   
    //drill down   
    recursion(level+1, params)   
    //clean current level status if needed   
}
```

## 分治

```python
# Python
def divide_conquer(problem, param1, param2, ...): 
    # recursion terminator 
    if problem is None: 
        print_result 
        return 
	
    # prepare data 
    data = prepare_data(problem) 
    subproblems = split_problem(problem, data) 
 
    # conquer subproblems 
    subresult1 = self.divide_conquer(subproblems[0], p1, ...) 
    subresult2 = self.divide_conquer(subproblems[1], p1, ...) 
    subresult3 = self.divide_conquer(subproblems[2], p1, ...) 
    # ... ...
    
    # process and generate the final result 
    result = process_result(subresult1, subresult2, subresult3, …)

    # revert the current level states
```

```c
// C/C++
int divide_conquer(Problem *problem, int params) {
 // recursion terminator
 if (problem == nullptr) {
  process_result
  return return_result;
 } 

 // process current problem
 subproblems = split_problem(problem, data)
 subresult1 = divide_conquer(subproblem[0], p1)
 subresult2 = divide_conquer(subproblem[1], p1)
 subresult3 = divide_conquer(subproblem[2], p1)
 ...

 // merge
 result = process_result(subresult1, subresult2, subresult3)
 
 // revert the current level status
 return 0;
}
```

```java
// Java
private static int divide_conquer(Problem problem, ) {
  if (problem == NULL) {
	int res = process_last_result();
	return res;     
  }

  subProblems = split_problem(problem)
  
  res0 = divide_conquer(subProblems[0])
  res1 = divide_conquer(subProblems[1])

  result = process_result(res0, res1);
  
  return result;
}
```

```javascript
// Javascript
const divide_conquer = (problem, params) => {
 // recursion terminator
 if (problem == null) {
  process_result
  return
 } 

 // process current problem
 subproblems = split_problem(problem, data)
 subresult1 = divide_conquer(subproblem[0], p1)
 subresult2 = divide_conquer(subproblem[1], p1)
 subresult3 = divide_conquer(subproblem[2], p1)
 ...

 // merge
 result = process_result(subresult1, subresult2, subresult3)
 // revert the current level status
}
```

## BFS

```python
# Python

def BFS(root):
    visited = set()
	queue = [] 
	queue.append([root]) 

    while queue: 
        node = queue.pop() 
        visited.add(node)
        
        process(node) 
        nodes = generate_related_nodes(node) 
        queue.push(nodes)

    # other processing work 
```

```java
// Java
public class TreeNode {
	int val;
	TreeNode left;
	TreeNode right;
	TreeNode(int x) {
    	val = x;
	}
}

public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> allResults = new ArrayList<>();
    if (root == null) {
        return allResults;
    }

    Queue<TreeNode> nodes = new LinkedList<>();
    nodes.add(root);
    while (!nodes.isEmpty()) {
        int size = nodes.size();
        List<Integer> results = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = nodes.poll();
            results.add(node.val);
            if (node.left != null) {
                nodes.add(node.left);
            }

            if (node.right != null) {
                nodes.add(node.right);
            }
        }
        allResults.add(results);
    }
    return allResults;
}
```

```c
// C/C++
void bfs(Node* root) {
 map<int, int> visited;
 if(!root) return ;
    
 queue<Node*> queueNode;
 queueNode.push(root);
    
 while (!queueNode.empty()) {
  Node* node = queueNode.top();
  queueNode.pop();
  if (visited.count(node->val)) continue;
  visited[node->val] = 1;
     
  for (int i = 0; i < node->children.size(); ++i) {
	queueNode.push(node->children[i]);
  }
 }
    
 return ;
}
```

```javascript
// JavaScript
const bfs = (root) => {
 let result = [], queue = [root]
 while (queue.length > 0) {
  let level = [], n = queue.length
  for (let i = 0; i < n; i++) {
   let node = queue.pop()
   level.push(node.val) 
   if (node.left) queue.unshift(node.left)
   if (node.right) queue.unshift(node.right)
  }
  result.push(level)
 }
 return result
};
```

## DFS

递归写法

```python
# python
visited = set()

def dfs(node, visited):    
    if node in visited: 
        # terminator    	
        # already visited     	
        return
    
    visited.add(node)   
    # process current node here.  
    ... 
    
    for next_node in node.children():  	
        if next_node not in visited:  		
            dfs(next_node, visited)
```

非递归写法

```python
def DFS(self, root):   
    if tree.root is None:  	
        return []   
    
    visited, stack = [], [root]  
    
    while stack:  	
        node = stack.pop()  	
        visited.add(node)  
        
        process (node)         
        # 生成相关的节点 	
        nodes = generate_related_nodes(node)  	
        stack.push(nodes)   
        
        # other processing work 
        ... ...
```

```c
//C/C++
//递归写法：
map<int, int> visited;

void dfs(Node* root) {
  // terminator
  if (!root) return ;

  if (visited.count(root->val)) {
    // already visited
    return ;
  }

  visited[root->val] = 1;

  // process current node here. 
  // ...
  for (int i = 0; i < root->children.size(); ++i) {
    dfs(root->children[i]);
  }
  return ;
}

//非递归写法：
void dfs(Node* root) {
  map<int, int> visited;
  if(!root) return ;

  stack<Node*> stackNode;
  stackNode.push(root);

  while (!stackNode.empty()) {
    Node* node = stackNode.top();
    stackNode.pop();
    if (visited.count(node->val)) continue;
    visited[node->val] = 1;


    for (int i = node->children.size() - 1; i >= 0; --i) {
        stackNode.push(node->children[i]);
    }
  }

  return ;
}
```

```javascript
	// Java
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> allResults = new ArrayList<>();
        if(root==null){
            return allResults;
        }
        travel(root,0,allResults);
        return allResults;
    }

    private void travel(TreeNode root,int level,List<List<Integer>> results){
        if(results.size()==level){
            results.add(new ArrayList<>());
        }
        results.get(level).add(root.val);
        if(root.left!=null){
            travel(root.left,level+1,results);
        }
        if(root.right!=null){
            travel(root.right,level+1,results);
        }
    }
```

```javascript
// javascript
const visited = new Set()
const dfs = node => {
  if (visited.has(node)) return
  visited.add(node)
  dfs(node.left)
  dfs(node.right)
}
```

## 二分查找

```python
# Python
left, right = 0, len(array) - 1 
while left <= right: 
	mid = (left + right) / 2 
	if array[mid] == target: 
        # find the target!! 
        break or return result 
	elif array[mid] < target: 
		left = mid + 1 
	else: 
		right = mid - 1
```

```c
// C/C++
int binarySearch(const vector<int>& nums, int target) {
	int left = 0, right = (int)nums.size()-1;

    while (left <= right) {
        int mid = left + (right - left)/ 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    
	return -1;
}
```

```java
// Java
public int binarySearch(int[] array, int target) {
    int left = 0, right = array.length - 1, mid;
    
    while (left <= right) {
        mid = (right - left) / 2 + left;
        if (array[mid] == target) {
            return mid;
        } else if (array[mid] > target) {
     		right = mid - 1;
        } else {
     		left = mid + 1;
        }
    }
    return -1;
}
```

```javascript
// JavaScript
let left = 0, right = len(array) - 1

while (left <= right) {
 let mid = (left + right) >> 1
 if (array[mid] === target) { /*find the target*/; return }
 else if (array[mid] < target) left = mid + 1
 else right = mid - 1
}
```

