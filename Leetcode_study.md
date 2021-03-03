### LC291 Word Pattern II

```java
   class Solution {
   public boolean wordPatternMatch(String pattern, String str) {
        return dfs(new HashMap<Character, String>(), pattern, str, 0, 0);
    }
    private boolean dfs(HashMap<Character, String> map, String pattern, String str, int pi, int si){
        int plen = pattern.length();
        int slen = str.length();
        if(pi == plen && si == slen){
            return true;
        }
        if(pi == plen || si == slen){
            return false;
        }
        if(map.containsKey(pattern.charAt(pi))){
            if(!str.startsWith(map.get(pattern.charAt(pi)), si)){
                  return false;
            }
            else{
                return dfs(map, pattern, str, pi + 1, si + map.get(pattern.charAt(pi)).length());
            }
        }
        else{
            for(int i = si + 1; i <= slen - plen + pi + 1; i++){
                String sub = str.substring(si, i);
                if(map.containsValue(sub)) continue;
                map.put(pattern.charAt(pi), sub);
                if(dfs(map, pattern, str, pi + 1, i)) return true;
                map.remove(pattern.charAt(pi), sub);
            }
            return false;
        }
    }
}
```

这段代码如果把 `i <= slen` 换成 ` i <= slen - plen + pi + 1`, 会快很多。`plen - 1 - pi` 代表的是如果pattern的index到了pi这里，最后还有`plen - 1 - pi`个pattern字母是没有traverse到的，那如果str需要成立的话，至少也需要留下`plen - 1 - pi `个字母使得后面的pattern都能被match。这样就减少了for loop的长度。

结果可以从30ms到1ms



### LC291 Word Pattern II ###

DP做法

Define **state** as:
state[i\][j] is true if s[0..i) matches p[0..j)

Then **goal state** is:
state[slen-1\][plen-1]

**state transition**

```
if s[i] == p[j] , state[i][j] = state[i - 1][j - 1]
else {
  if p[j] == '.', state[i][j] = state[i - 1][j - 1]
  else if p[j] == '*', // to expand
  else state[i][j] = false
}

To expand: 
			if (s[i] == p[j-1] || p[j-1] == .) state[i][j] = true if
				state[i][j-2] = true // if * means 0 preceding element       
				|| state[i][j-1] = true // if * means 1 preceding element        
				|| state[i-1][j] = true and (s[i] = p[j-1] or p[j-1] = .) // if * means >1 preceding elements # to explain
			else // * can only mean 0 preceding element   
				state[i][j] = state[i][j-2]  
```

> Explanation for `# to explain`

```
e.g. when i = 2, j = 1,
s = aaa
      i
p = a*
     j
> p = "a*" can be represented as p = "aa"
> the last 'a' in p matches s[2], and it may keep matching s[1] and s[0]
> thus,
> state[2][1] should be true if state[1][1] is true
> state[1][1] should be true if state[0][1] is true
```

> **note**
> in the code below, we add one more row and one more column to the state array to avoid if - elses.
> In this way,

------

```java
    public boolean isMatch(String s, String p) {
        boolean[][] state = new boolean[s.length() + 1][p.length() + 1];
        state[0][0] = true;
        for (int j = 2; j <= p.length(); j++)
            state[0][j] = p.charAt(j - 1) == '*' && state[0][j - 2];
        
        for (int i = 1; i <= s.length(); i++) {
            for (int j = 1; j <= p.length(); j++ ) {
                if (s.charAt(i - 1) == p.charAt(j - 1)) {
                    state[i][j] = state[i - 1][j - 1];
                } else {
                    if (p.charAt(j - 1) == '.') {
                        state[i][j] = state[i - 1][j - 1];
                    } else if (p.charAt(j - 1) == '*') {
                        // * matches 0, 1, > 1 preceding elements
                        state[i][j] = (j > 1 && state[i][j - 2])
                            || state[i][j - 1] 
                            || (state[i - 1][j] && (s.charAt(i - 1) == p.charAt(j - 2) || p.charAt(j - 2) == '.'));
                    }
                    // else {
                    //     state[i][j] = false;
                    // }
                }
            }
        }
        return state[s.length()][p.length()];
    }
```

# LC312 #

https://www.cnblogs.com/grandyang/p/5006441.html





# 解法一 递归之分治

S 中的每个字母就是两种可能选他或者不选他。我们用递归的常规思路，将大问题化成小问题，也就是分治的思想。

如果我们求 `S[0，S_len - 1]` 中能选出多少个 `T[0，T_len - 1]`，个数记为 `n`。那么分两种情况，

- `S[0] == T[0]`，需要知道两种情况

  - 从 `S` 中选择当前的字母，此时 `S` 跳过这个字母, `T` 也跳过一个字母。

    去求 `S[1，S_len - 1]` 中能选出多少个 `T[1，T_len - 1]`，个数记为 `n1`

  - `S` 不选当前的字母，此时`S`跳过这个字母，`T` 不跳过字母。

    去求`S[1，S_len - 1]` 中能选出多少个 `T[0，T_len - 1]`，个数记为 `n2`

- `S[0] ！= T[0]`

  `S` 只能不选当前的字母，此时`S`跳过这个字母， `T` 不跳过字母。

  去求`S[1，S_len - 1]` 中能选出多少个 `T[0，T_len - 1]`，个数记为 `n1`

也就是说如果求 `S[0，S_len - 1]` 中能选出多少个 `T[0，T_len - 1]`，个数记为 n。转换为数学式就是

```java
if(S[0] == T[0]){
    n = n1 + n2;
}else{
    n = n1;
}
Copy
```

推广到一般情况，我们可以先写出递归的部分代码。

```java
public int numDistinct(String s, String t) {
    return numDistinctHelper(s, 0, t, 0);
}

private int numDistinctHelper(String s, int s_start, String t, int t_start) {
    int count = 0;
    //当前字母相等
    if (s.charAt(s_start) == t.charAt(t_start)) {
        //从 S 选择当前的字母，此时 S 跳过这个字母, T 也跳过一个字母。
        count = numDistinctHelper(s, s_start + 1, t, t_start + 1, map)
        //S 不选当前的字母，此时 S 跳过这个字母，T 不跳过字母。
                + numDistinctHelper(s, s_start + 1, t, t_start,  map);
    //当前字母不相等  
    }else{ 
       //S 只能不选当前的字母，此时 S 跳过这个字母， T 不跳过字母。
       count = numDistinctHelper(s, s_start + 1, t, t_start,  map);
    }
    return count; 
}
Copy
```

递归出口的话，因为我们的`S`和`T`的开始下标都是增长的。

如果`S[s_start, S_len - 1]`中， `s_start` 等于了 `S_len` ，意味着`S`是空串，从空串中选字符串`T`，那结果肯定是`0`。

如果`T[t_start, T_len - 1]`中，`t_start`等于了 `T_len`，意味着`T`是空串，从`S`中选择空字符串`T`，只需要不选择 `S` 中的所有字母，所以选法是`1`。

综上，代码总体就是下边的样子

```java
public int numDistinct(String s, String t) {
    return numDistinctHelper(s, 0, t, 0);
}

private int numDistinctHelper(String s, int s_start, String t, int t_start) {
    //T 是空串，选法就是 1 种
    if (t_start == t.length()) { 
        return 1;
    }
    //S 是空串，选法是 0 种
    if (s_start == s.length()) {
        return 0;
    }
    int count = 0;
    //当前字母相等
    if (s.charAt(s_start) == t.charAt(t_start)) {
        //从 S 选择当前的字母，此时 S 跳过这个字母, T 也跳过一个字母。
        count = numDistinctHelper(s, s_start + 1, t, t_start + 1)
        //S 不选当前的字母，此时 S 跳过这个字母，T 不跳过字母。
              + numDistinctHelper(s, s_start + 1, t, t_start);
    //当前字母不相等  
    }else{ 
        //S 只能不选当前的字母，此时 S 跳过这个字母， T 不跳过字母。
        count = numDistinctHelper(s, s_start + 1, t, t_start);
    }
    return count; 
}
Copy
```

遗憾的是，这个解法对于如果`S`太长的 `case` 会超时。

![img](https://windliang.oss-cn-beijing.aliyuncs.com/115_2.jpg)

原因就是因为递归函数中，我们多次调用了递归函数，这会使得我们重复递归很多的过程，解决方案就很简单了，`Memoization` 技术，把每次的结果利用一个`map`保存起来，在求之前，先看`map`中有没有，有的话直接拿出来就可以了。

`map`的`key`的话就标识当前的递归，`s_start` 和 `t_start` 联合表示，利用字符串 `s_start + '@' + t_start`。

`value`的话就保存这次递归返回的`count`。

```java
public int numDistinct(String s, String t) {
    HashMap<String, Integer> map = new HashMap<>();
    return numDistinctHelper(s, 0, t, 0, map);
}

private int numDistinctHelper(String s, int s_start, String t, int t_start, HashMap<String, Integer> map) {
    //T 是空串，选法就是 1 种
    if (t_start == t.length()) { 
        return 1;
    }
    //S 是空串，选法是 0 种
    if (s_start == s.length()) {
        return 0;
    }
    String key = s_start + "@" + t_start;
    //先判断之前有没有求过这个解
    if (map.containsKey(key)) {
        return map.get(key); 
    }
    int count = 0;
    //当前字母相等
    if (s.charAt(s_start) == t.charAt(t_start)) {
        //从 S 选择当前的字母，此时 S 跳过这个字母, T 也跳过一个字母。
        count = numDistinctHelper(s, s_start + 1, t, t_start + 1, map)
        //S 不选当前的字母，此时 S 跳过这个字母，T 不跳过字母。
              + numDistinctHelper(s, s_start + 1, t, t_start, map);
    //当前字母不相等  
    }else{ 
        //S 只能不选当前的字母，此时 S 跳过这个字母， T 不跳过字母。
        count = numDistinctHelper(s, s_start + 1, t, t_start, map);
    }
    //将当前解放到 map 中
    map.put(key, count);
    return count; 
```

# 解法三 动态规划

让我们来回想一下解法一做了什么。`s_start` 和 `t_start` 不停的增加，一直压栈，压栈，直到

```java
//T 是空串，选法就是 1 种
if (t_start == t.length()) { 
    return 1;
}
//S 是空串，选法是 0 种
if (s_start == s.length()) {
    return 0;
}
Copy
```

`T` 是空串或者 `S` 是空串，我们就直接可以返回结果了，接下来就是不停的出栈出栈，然后把结果通过递推关系取得。

递归的过程就是由顶到底再回到顶。

动态规划要做的就是去省略压栈的过程，直接由底向顶。

这里我们用一个二维数组 `dp[m][n]` 对应于从 `S[m，S_len)` 中能选出多少个 `T[n，T_len)`。

当 `m == S_len`，意味着`S`是空串，此时`dp[S_len][n]`，n 取 0 到 `T_len - 1`的值都为 `0`。

当 `n == T_len`，意味着`T`是空串，此时`dp[m][T_len]`，m 取 0 到 `S_len`的值都为 `1`。

然后状态转移的话和解法一分析的一样。如果求`dp[s][t]`。

- `S[s] == T[t]`，当前字符相等，那就对应两种情况，选择`S`的当前字母和不选择`S`的当前字母

  `dp[s][t] = dp[s+1][t+1] + dp[s+1][t]`

- `S[s] != T[t]`，只有一种情况，不选择`S`的当前字母

  `dp[s][t] = dp[s+1][t]`

代码就可以写了。

```java
public int numDistinct(String s, String t) {
    int s_len = s.length();
    int t_len = t.length();
    int[][] dp = new int[s_len + 1][t_len + 1];
    //当 T 为空串时，所有的 s 对应于 1
    for (int i = 0; i <= s_len; i++) {
        dp[i][t_len] = 1;
    }

    //倒着进行，T 每次增加一个字母
    for (int t_i = t_len - 1; t_i >= 0; t_i--) {
        dp[s_len][t_i] = 0; // 这句可以省去，因为默认值是 0
        //倒着进行，S 每次增加一个字母
        for (int s_i = s_len - 1; s_i >= 0; s_i--) {
            //如果当前字母相等
            if (t.charAt(t_i) == s.charAt(s_i)) {
                //对应于两种情况，选择当前字母和不选择当前字母
                dp[s_i][t_i] = dp[s_i + 1][t_i + 1] + dp[s_i + 1][t_i];
            //如果当前字母不相等
            } else {
                dp[s_i][t_i] = dp[s_i + 1][t_i];
            }
        }
    }
    return dp[0][0];
}
Copy
```

对比于解法一和解法二，如果`Memoization` 技术我们不用`hash`，而是用一个二维数组，会发现其实我们的递归过程，其实就是在更新下图中的二维表，只不过更新的顺序没有动态规划这么归整。这也是不用`Memoization` 技术会超时的原因，如果把递归的更新路线画出来，会发现很多路线重合了，意味着我们进行了很多没有必要的递归，从而造成了超时。

我们画一下动态规划的过程。

```
S = "babgbag", T = "bag"
```

T 为空串时，所有的 s 对应于 1。 S 为空串时，所有的 t 对应于 0。

![img](https://windliang.oss-cn-beijing.aliyuncs.com/115_3.jpg)

此时我们从 `dp[6][2]` 开始求。根据公式，因为当前字母相等，所以 `dp[6][2] = dp[7][3] + dp[7][2] = 1 + 0 = 1 。`

接着求`dp[5][2]`，当前字母不相等，`dp[5][2] = dp[6][2] = 1`。

一直求下去。

![img](https://windliang.oss-cn-beijing.aliyuncs.com/115_4.jpg)

求当前问号的地方的值的时候，我们只需要它的上一个值和斜对角的值。

换句话讲，求当前列的时候，我们只需要上一列的信息。比如当前求第`1`列，第`3`列的值就不会用到了。

所以我们可以优化算法的空间复杂度，不需要二维数组，需要一维数组就够了。

此时需要解决一个问题，就是当求上图的`dp[1][1]`的时候，需要`dp[2][1]`和`dp[2][2]`的信息。但是如果我们是一维数组，`dp[2][1]`之前已经把`dp[2][2]`的信息覆盖掉了。所以我们需要一个`pre`变量保存之前的值。

```java
public int numDistinct(String s, String t) {
    int s_len = s.length();
    int t_len = t.length();
    int[]dp = new int[s_len + 1];
    for (int i = 0; i <= s_len; i++) {
        dp[i] = 1;
    }
  //倒着进行，T 每次增加一个字母
    for (int t_i = t_len - 1; t_i >= 0; t_i--) {
        int pre = dp[s_len];
        dp[s_len] = 0; 
         //倒着进行，S 每次增加一个字母
        for (int s_i = s_len - 1; s_i >= 0; s_i--) {
            int temp = dp[s_i];
            if (t.charAt(t_i) == s.charAt(s_i)) {
                dp[s_i] = dp[s_i + 1] + pre;
            } else {
                dp[s_i] = dp[s_i + 1];
            }
            pre = temp;
        }
    }
    return dp[0];
}
Copy
```

利用`temp`和`pre`两个变量实现了保存之前的值。

其实动态规划优化空间复杂度的思想，在 [5题](https://leetcode.windliang.cc/leetCode-5-Longest-Palindromic-Substring.html)，[10题](https://leetcode.windliang.cc/leetCode-10-Regular-Expression-Matching.html)，[53题](https://leetcode.windliang.cc/leetCode-53-Maximum-Subarray.html?h=动态规划)，[72题 ](https://leetcode.wang/leetCode-72-Edit-Distance.html)等等都已经用了，是非常经典的。

上边的动态规划是从字符串末尾倒着进行的，其实我们只要改变`dp`数组的含义，用`dp[m][n]`表示`S[0,m)`和`T[0,n)`，然后两层循环我们就可以从 `1` 往末尾进行了，思想是类似的，`leetcode` 高票答案也都是这样的，如果理解了上边的思想，代码其实也很好写。这里只分享下代码吧。

```java
public int numDistinct(String s, String t) {
    int s_len = s.length();
    int t_len = t.length();
    int[] dp = new int[s_len + 1];
    for (int i = 0; i <= s_len; i++) {
        dp[i] = 1;
    }
    for (int t_i = 1; t_i <= t_len; t_i++) {
        int pre = dp[0];
        dp[0] = 0;
        for (int s_i = 1; s_i <= s_len; s_i++) {
            int temp = dp[s_i];
            if (t.charAt(t_i - 1) == s.charAt(s_i - 1)) {
                dp[s_i] = dp[s_i - 1] + pre;
            } else {
                dp[s_i] = dp[s_i - 1];
            }
            pre = temp;
        }
    }
    return dp[s_len];
}

```



LC98



来利用另一种思路，参考[官方题解](https://leetcode.com/problems/validate-binary-search-tree/solution/)。

解法一中，我们是判断根节点是否合法，找到了左子树中最大的数，右子树中最小的数。 由左子树和右子树决定当前根节点是否合法。

但如果正常的来讲，明明先有的根节点，按理说根节点是任何数都行，而不是由左子树和右子树限定。相反，根节点反而决定了左孩子和右孩子的合法取值范围。

所以，我们可以从根节点进行 DFS，然后计算每个节点应该的取值范围，如果当前节点不符合就返回 false。

```java
      10
    /    \
   5     15
  / \    /  
 3   6  7 

   考虑 10 的范围
     10(-inf,+inf)

   考虑 5 的范围
     10(-inf,+inf)
    /
   5(-inf,10)

   考虑 3 的范围
       10(-inf,+inf)
      /
   5(-inf,10)
    /
  3(-inf,5)  

   考虑 6 的范围
       10(-inf,+inf)
      /
   5(-inf,10)
    /       \
  3(-inf,5)  6(5,10)

   考虑 15 的范围
      10(-inf,+inf)
    /          \
    5(-inf,10) 15(10,+inf）
    /       \
  3(-inf,5)  6(5,10)  

   考虑 7 的范围，出现不符合返回 false
       10(-inf,+inf)
     /              \
5(-inf,10)           15(10,+inf）
  /       \             /
3(-inf,5)  6(5,10)   7（10,15）
Copy
```

可以观察到，左孩子的范围是 （父结点左边界，父节点的值），右孩子的范围是（父节点的值，父节点的右边界）。

还有个问题，java 里边没有提供负无穷和正无穷，用什么数来表示呢？

方案一，假设我们的题目的数值都是 Integer 范围的，那么我们用不在 Integer 范围的数字来表示负无穷和正无穷。用 long 去存储。

```java
public boolean isValidBST(TreeNode root) {
    long maxValue = (long)Integer.MAX_VALUE + 1;
    long minValue = (long)Integer.MIN_VALUE - 1;
    return getAns(root, minValue, maxValue);
}

private boolean getAns(TreeNode node, long minVal, long maxVal) {
    if (node == null) {
        return true;
    }
    if (node.val <= minVal) {
        return false;
    }
    if (node.val >= maxVal) {
        return false;
    }
    return getAns(node.left, minVal, node.val) && getAns(node.right, node.val, maxVal);
}
Copy
```

方案二：传入 Integer 对象，然后 null 表示负无穷和正无穷。然后利用 JAVA 的自动装箱拆箱，数值的比较可以直接用不等号。

```java
public boolean isValidBST(TreeNode root) {
    return getAns(root, null, null);
}

private boolean getAns(TreeNode node, Integer minValue, Integer maxValue) { 
    if (node == null) {
        return true;
    }
    if (minValue != null && node.val <= minValue) {
        return false;
    }
    if (maxValue != null && node.val >= maxValue) {
        return false;
    }
    return getAns(node.left, minValue, node.val) && getAns(node.right, node.val, maxValue);
}

```



LC124

A lot of the discussions here left me wondering with the magic that was happening between the lines and did not clearly explain the concept of the solution.
Here is a step by step solution of what would probably be easier to understand.



Our goal is to find the maximum path sum. Now consider the following example.



```
   10
  /  \
null null
```



In this simple case we know that the max sum would be just the root node itself and the answer would be 10. So fo all `leaf` nodes the max sum path is the value of the node itself.



Now let's consider the following example.



```
   20
  /  \
10    30
```



Here there are multiple possibilities and we need to take care of the following **FOUR PATHS** that could be our max.



1. The root iself : `20`

2. The root with the maximum from it's left subTree :

   ```
   	20
   	/
     10
   ```

```
3. The root with the maximum from it's right subTree : 
	
		20
		  \
	       30
```

1. The root with it's left, right and itself

```
   	 20
   	 / \
    10  30
```



In case you are wondering why we did not choose the root.left (10) or root.right(30) alone in the calculation ( like I wondered ), that's because we would have already computed the result of it as a node in our recursion separately.



This actually breaks down our code to a very simple pseudo code:



```
if( root == null) return 0;
left = recurse(leftChild);
right = recurse(rightChild);

// now find the max of all the four paths
leftPath = root.value + left;
rightPath = root.value + right;
completePath = root.value + right + left;

result = max( root.value, leftPath, rightPath, completePath );

return max(leftPath, rightPath);
```



*What's interesting to note here is the last line of the code* :



```
return max(leftPath, rightPath);
```



**Wondering why did we do that ?**



Well, we know that we did all the calculations possible if the tree only consists of the current node as root in any possible recursion cycle. And the result of that cycle would have been stored in the `result` variable.
But, what if the current node is just a child of it's parent. Then it needs to return a value, such that the root had to be part of the answer.
So if the root has to be part of the answer, it should return what's the maximum value it can return if it's part of it.
That would be either of the three cases here :



1. The root iself : `20`

2. The root with the maximum from it's left subTree :

   ```
   	20
   	/
     10
   ```

```
3. The root with the maximum from it's right subTree : 
	
		20
		  \
	       30
```



This concludes us to the following code :



```java
public class BinaryTreeMaximumPathSum {
    private int maxSum;

    public int maxSumHelper(TreeNode root) {
		
		// base case
        if (root == null) return 0; 
		
		// recursing through left and right subtree
        int leftMax = maxSumHelper(root.left);
        int rightMax = maxSumHelper(root.right);

		// finding all the four paths and the maximum between all of them
        int maxRightLeft = Math.max(leftMax, rightMax);
        int maxOneNodeRoot = Math.max(root.val, (root.val + maxRightLeft));
        int maxAll = Math.max(maxOneNodeRoot, leftMax + rightMax + root.val);
		
		// Storing the result in the maxSum holder
        maxSum = Math.max(maxSum, maxAll);
		
		// returning the value if root was part of the answer
        return maxOneNodeRoot;

    }

    public int maxPathSum(TreeNode root) {
        maxSum = Integer.MIN_VALUE;
        maxSumHelper(root);
        return maxSum; // as maxSum will always store the result

    }
}
```