[TOC]

### 1.两数之和

![1591891479914](../../../../Software/Typora/Picture/1591891479914.png)

![1591893013977](../../../../Software/Typora/Picture/1591893013977.png)





### 2.三数之和

![1591932106878](../../../../Software/Typora/Picture/1591932106878.png)

![1591948099010](../../../../Software/Typora/Picture/1591948099010.png)



## 3. 转变数组后最接近目标值的数组和

![image-20200614171635170](../../../../Software/Typora/Picture/image-20200614171635170.png)

解法1：贪心算法

![image-20200614212048643](../../../../Software/Typora/Picture/image-20200614212048643.png)

解法2：二分查找+枚举

![image-20200614212159853](../../../../Software/Typora/Picture/image-20200614212159853.png)



## 4. 最长公共前缀

![image-20200615121709558](../../../../Software/Typora/Picture/image-20200615121709558.png)

```
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if(strs.length == 0)
            return "";
        String ans = strs[0];
        
        for(int i=0; i<strs.length; i++ ){
            int j = 0 ;
            for(;j<ans.length() && j<strs[i].length(); j++){
                if(ans.charAt(j) != strs[i].charAt(j))
                    break;
            }
            ans = ans.substring(0, j);
            if(Objects.equals("", ans))
                return "";
        }
        return ans;
    }
}
```

## 5. 二叉树的序列化与反序列化

请设计一个算法来实现二叉树的序列化与反序列化。这里不限定你的序列 / 反序列化算法执行逻辑，你只需要保证一个二叉树可以被序列化为一个字符串并且将这个字符串反序列化为原始的树结构。

![image-20200616132548640](../../../../Software/Typora/Picture/image-20200616132548640.png)

```
public class Codec {
    // 序列化为String
    public String rserialize(TreeNode root, String str){
        if(root == null){
            str += "None,"; // 遍历到叶子节点时
        }else{
            str += str.valueOf(root.val) + ",";
            str = rserialize(root.left, str);
            str = rserialize(root.right, str);
        }
        return str;
    }
    public String serialize(TreeNode root) {
        return rserialize(root, "");
    }
    // 反序列化为TreeNode
    public TreeNode rdeserialize(List<String> str){
        if(str.get(0).equals("None")){ 
            str.remove(0);
            return null; //根节点为None，直接返回null
        }
        TreeNode root = new TreeNode(Integer.valueOf(str.get(0)));
        str.remove(0);
        root.left = rdeserialize(str);
        root.right = rdeserialize(str);
        return root;
    }
    public TreeNode deserialize(String data) {
        String[] data_array = data.split(",");
        List<String> data_list = new LinkedList<String>(Arrays.asList(data_array));
        return rdeserialize(data_list);
    }
}
```



### 最佳观光组合

```
给定正整数数组 A，A[i] 表示第 i 个观光景点的评分，并且两个景点 i 和 j 之间的距离为 j - i。
一对景点（i < j）组成的观光组合的得分为（A[i] + A[j] + i - j）：景点的评分之和减去它们两者之间的距离。
返回一对观光景点能取得的最高分。
```

```
class Solution {
    public int maxScoreSightseeingPair(int[] A) {
        int max = 0, ans = 0;
        for(int i=0; i<A.length; i++){
            if(max + A[i] - i > ans)
                ans = max + A[i] - i;
            if(A[i]+i > max)
                max = A[i]+i;
        }
        return ans;
    }
}
```

### 从先序遍历还原二叉树

![image-20200618202259080](../../../../Software/Typora/Picture/image-20200618202259080.png)

```
class Solution {
    public TreeNode recoverFromPreorder(String S) {
        Stack<TreeNode> stack = new Stack<>();
        for(int i=0; i<S.length();){
            int level = 0; // 根结点在第0层
            while(S.charAt(i) == '-'){
                level++;  // level用来记录结点层数
                i++; // 记录结点在栈中的索引
            }
            int val = 0; // 记录当前结点数字
            while(i < S.length() && S.charAt(i) != '-'){
                val = val * 10 + (S.charAt(i) - '0'); // 拼接结点数字字符串
                i++;
            }
            // 找到新结点的父结点
            while(stack.size() > level){
                stack.pop();  // 将除了当前父结点之下的子结点出栈
            }
            // 创建当前结点
            TreeNode node = new TreeNode(val);
            if(!stack.isEmpty()){
                if(stack.peek().left == null){
                    stack.peek().left = node;  // 返回栈顶元素
                }else{
                    stack.peek().right = node;
                }
            }
            stack.add(node); // 新结点入栈
        }
        // 除了根节点，其他子节点全部出栈
        while(stack.size() > 1){
            stack.pop();  
        }
        return stack.pop(); // 出栈并返回根节点
    }
}
```

### 回文串

给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。

**说明：**本题中，我们将空字符串定义为有效的回文串。

```
class Solution {
    public boolean isPalindrome(String s) {
        StringBuffer sbuf = new StringBuffer();
        int len = s.length();
        char ch;
        for(int i=0; i<len; i++){
            ch = s.charAt(i);
            if(Character.isLetterOrDigit(ch))
                sbuf.append(Character.toLowerCase(ch));
        }        
        int n = sbuf.length();
        int left = 0, right = n-1;
        while(left < right){
            if(Character.toLowerCase(sbuf.charAt(left)) != Character.toLowerCase(sbuf.charAt(right))){
                return false;
            }
            ++left;
            --right;
        }
        return true;
    }
}
```

### [ 二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

给定一个**非空**二叉树，返回其最大路径和。

本题中，路径被定义为一条从树中任意节点出发，达到任意节点的序列。该路径**至少包含一个**节点，且不一定经过根节点。

```
输入: [1,2,3]
       1
      / \
     2   3
输出: 6
```

```
class Solution {
    int maxSum = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        maxGain(root);
        return maxSum;
    }
    public int maxGain(TreeNode node) {
        if (node == null) {
            return 0;
        }
        // 递归计算左右子节点的最大贡献值
        // 只有在最大贡献值大于 0 时，才会选取对应子节点
        int leftGain = Math.max(maxGain(node.left), 0);
        int rightGain = Math.max(maxGain(node.right), 0);
        // 节点的最大路径和取决于该节点的值与该节点的左右子节点的最大贡献值
        int priceNewpath = node.val + leftGain + rightGain;
        // 更新答案
        maxSum = Math.max(maxSum, priceNewpath);
        // 返回节点的最大贡献值
        return node.val + Math.max(leftGain, rightGain);
    }
}
```

### [模式匹配](https://leetcode-cn.com/problems/pattern-matching-lcci/)

```
你有两个字符串，即pattern和value。 pattern字符串由字母"a"和"b"组成，用于描述字符串中的模式。例如，字符串"catcatgocatgo"匹配模式"aabab"（其中"cat"是"a"，"go"是"b"），该字符串也匹配像"a"、"ab"和"b"这样的模式。但需注意"a"和"b"不能同时表示相同的字符串。编写一个方法判断value字符串是否匹配pattern字符串。
```

```
    class Solution {
        public boolean patternMatching(String pattern, String value) {
            String s[]=new String[2];
            return solve(s,pattern,0,value,0);
        }
        /**
         * 回溯遍历设置a,b的对应值，尝试每一种可能。
         * @param s   s[0]=a对应的字符串 s[1]=b对应的字符串
         * @param pattern 模式串
         * @param index1 模式串匹配位置
         * @param value 匹配串（待匹配的字符串）
         * @param index2 匹配串匹配位置
         * @return
         */
        public boolean solve(String []s,String pattern,int index1,String value,int index2){
            //匹配完成
            if(index1==pattern.length()&&index2==value.length()) return true;
            //注意匹配串匹配位置等于长度的时候也可以继续匹配，因为模式串的a，b可以匹配空串。
            if(index1>=pattern.length()||index2>value.length()) return false;
            int num=pattern.charAt(index1)-'a';
            if(s[num]==null){
                //从当前尝试a或b对应的字符串的每一种可能
                for(int i=index2;i<=value.length();i++){
                    s[num]=value.substring(index2,i);
                    //(s[num]==null||s[num^1]==null||!s[num].equals(s[num^1]))  [是指a，b对应的字符串不可相等]
                    if((s[num]==null||s[num^1]==null||!s[num].equals(s[num^1]))&&solve(s,pattern,index1+1,value,i)) return true;
                }
                //失配时应将设置过的对应字符串为空
                s[num]=null;
                return false;
            }else{
                //若此前a或b已有对应的字符串匹配了，则查看当前位置时候能够匹配上。
                int end=index2+s[num].length();
                if(end> value.length()||!value.substring(index2,end).equals(s[num])) return false;
                return solve(s,pattern,index1+1,value,end);
            }
        }
    }
```

### 二进制求和

```
给你两个二进制字符串，返回它们的和（用二进制表示）。
输入为 非空 字符串且只包含数字 1 和 0。

class Solution {
    public String addBinary(String a, String b) {
        int jinwei = 0;  // 代表进位
        StringBuilder ans = new StringBuilder();
        int sum;
        // 二进制相加：从右开始遍历相加，jinwei存放前一次计算的进位
        for(int i = a.length() - 1, j = b.length() - 1; i >= 0 || j >= 0; i--, j--){
            sum = jinwei; 
            sum += i >= 0 ? a.charAt(i) - '0' : 0;
            sum += j >= 0 ? b.charAt(j) - '0' : 0;
            ans.append(sum % 2);  // %2 为二进制相加   
            jinwei = sum / 2;  // 
        }
        ans.append(jinwei == 1 ? jinwei : "");  // 最后的进位
        return ans.reverse().toString(); // 结果反转
    }
}
```

### [最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest/)

```
给定一个包括 n 个整数的数组 nums 和 一个目标值 target。找出 nums 中的三个整数，使得它们的和与 target 最接近。返回这三个数的和。假定每组输入只存在唯一答案。

class Solution {
    public int threeSumClosest(int[] nums, int target) {
        Arrays.sort(nums);
        int ans = nums[0] + nums[1] + nums[2];
        for(int i=0;i<nums.length;i++) { // i从0开始遍历，start总在i之后，在end之前
            int start = i+1, end = nums.length - 1;
            while(start < end) {
                int sum = nums[start] + nums[end] + nums[i];
                if(Math.abs(target - sum) < Math.abs(target - ans))
                    ans = sum;  // 逐渐接近target
                if(sum > target)
                    end--;  // 右指针左移
                else if(sum < target)
                    start++;  // 左指针右移
                else
                    return ans;  // 若sum和target相等表示最接近，直接返回结果
            }
        }
        return ans; 
    }
}
```

### [209. 长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

给定一个含有 n 个正整数的数组和一个正整数 s ，找出该数组中满足其和 ≥ s 的长度最小的连续子数组，并返回其长度。如果不存在符合条件的连续子数组，返回 0。

```
// 思路：通过队列的思想，在nums上定义左右指针，从最左遍历nums并比较相加后是否>=s，>=s则右指针右移，然后记录相加的最小数目min
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        int left = 0, right = 0, sum = 0, min = Integer.MAX_VALUE;
        while(right < nums.length){ // 右指针开始向右移动
            sum += nums[right++]; // 开始从左向右一个个相加
            while(sum >= s){ // 当前几个数相加大于等于s时
                min = Math.min(min, right - left); // 多次过程记录数量最小的
                sum -= nums[left++]; // 左指针在sums上向右移动，移除最左的数
            }
        }
        return min == Integer.MAX_VALUE ? 0 : min; // 输出相加大于等于s的最小数量
    } 
}
```

### 40 回溯算法+剪枝

### 剪枝

##### 1、简介

  在搜索算法中优化中，剪枝，就是通过某种判断，避免一些不必要的遍历过程，形象的说，就是剪去了搜索树中的某些“枝条”，故称剪枝。应用剪枝优化的核心问题是设计剪枝判断方法，即确定哪些枝条应当舍弃，哪些枝条应当保留的方法。

##### 2、剪枝优化三原则: 正确、准确、高效.原则

   搜索算法,绝大部分需要用到剪枝.然而,不是所有的枝条都可以剪掉,这就需要通过设计出合理的判断方法,以决定某一分支的取舍. 在设计判断方法的时候,需要遵循一定的原则.

##### 3、分类

  剪枝算法按照其判断思路可大致分成两类:可行性剪枝及最优性剪枝.

3.1 可行性剪枝 —— 该方法判断继续搜索能否得出答案，如果不能直接回溯。

3.2 最优性剪枝

  最优性剪枝，又称为上下界剪枝，是一种重要的搜索剪枝策略。它记录当前得到的最优值，如果当前结点已经无法产生比当前最优解更优的解时，可以提前回溯。

### 组合总和II


给定一个数组 `candidates` 和一个目标数 `target` ，找出 `candidates` 中所有可以使数字和为 `target` 的组合。

`candidates` 中的每个数字在每个组合中只能使用一次。

**说明：**

- 所有数字（包括目标数）都是正整数。
- 解集不能包含重复的组合。 

```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Deque;
import java.util.List;

public class Solution {

    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        int len = candidates.length;
        List<List<Integer>> res = new ArrayList<>();
        if (len == 0) {
            return res;
        }

        // 关键步骤
        Arrays.sort(candidates);

        Deque<Integer> path = new ArrayDeque<>(len);
        dfs(candidates, len, 0, target, path, res);
        return res;
    }

    /**
     * @param candidates 候选数组
     * @param len        冗余变量
     * @param begin      从候选数组的 begin 位置开始搜索
     * @param target     表示剩余，这个值一开始等于 target，基于题目中说明的"所有数字（包括目标数）都是正整数"这个条件
     * @param path       从根结点到叶子结点的路径
     * @param res
     */
    private void dfs(int[] candidates, int len, int begin, int target, Deque<Integer> path, List<List<Integer>> res) {
        if (target == 0) {
            res.add(new ArrayList<>(path));
            return;
        }
        for (int i = begin; i < len; i++) {
            // 大剪枝：减去 candidates[i] 小于 0，减去后面的 candidates[i + 1]、candidates[i + 2] 肯定也小于 0，因此用 break
            if (target - candidates[i] < 0) {
                break;
            }

            // 小剪枝：同一层相同数值的结点，从第 2 个开始，候选数更少，结果一定发生重复，因此跳过，用 continue
            if (i > begin && candidates[i] == candidates[i - 1]) {
                continue;
            }

            path.addLast(candidates[i]);
            // 调试语句 ①
            // System.out.println("递归之前 => " + path + "，剩余 = " + (target - candidates[i]));

            // 因为元素不可以重复使用，这里递归传递下去的是 i + 1 而不是 i
            dfs(candidates, len, i + 1, target - candidates[i], path, res);

            path.removeLast();
            // 调试语句 ②
            // System.out.println("递归之后 => " + path + "，剩余 = " + (target - candidates[i]));
        }
    }

    public static void main(String[] args) {
        int[] candidates = new int[]{10, 1, 2, 7, 6, 1, 5};
        int target = 8;
        Solution solution = new Solution();
        List<List<Integer>> res = solution.combinationSum2(candidates, target);
        System.out.println("输出 => " + res);
    }
}

```

## 回溯算法集合

### 39.组合总和

给定一个 **没有重复** 数字的序列，返回其所有可能的全排列。

```java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
    	List<List<Integer>> res = new ArrayList<>();
    	if(nums.length == 0) {
    		return res;
    	}
    	Deque<Integer> path = new ArrayDeque<>();
    	boolean[] used = new boolean[nums.length];
    	dfs(nums, 0, path, used, res);
    	return res;
    }
    
    /**
     * 
     * @param nums   
     * @param depth  当前递归的层数
     * @param path   双向链表
     * @param used   标识某个数是否已经被使用过
     * @param res
     */
    public void dfs(int[] nums, int depth, Deque<Integer> path, boolean[] used, List<List<Integer>> res) {
    	int len = nums.length;
    	// 递归终止条件
     	if(depth == len) {
            // 搜索完毕，加入结果集
     		res.add(new ArrayList<>(path));
     		return;
     	}
     	for(int i = 0; i < len; i++) {
     		if(used[i]) {
     			// 去重操作：防止每次遍历后出现重复数字的序列
     			continue;
     		}
     		path.addLast(nums[i]);
     		used[i] = true;
     		dfs(nums, depth + 1, path, used, res);
     		// 回溯
     		path.removeLast();
     		used[i] = false;
     	}
    }
}
```

### 78 子集

```
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        if(nums.length == 0) {
            return res;
        }
        
        List<Integer> temp = new ArrayList<>();
        dfs(nums, 0, res, temp);
        return res;
    }

    public void dfs(int[] nums, int index, List<List<Integer>> res, List<Integer> temp) {
        
        res.add(new ArrayList<>(temp));
        for(int i = index; i < nums.length; i++) {
            temp.add(nums[i]);
            dfs(nums, i + 1, res, temp);
            // 回溯
            temp.remove(temp.size() - 1); 
        }
    }
}
```









