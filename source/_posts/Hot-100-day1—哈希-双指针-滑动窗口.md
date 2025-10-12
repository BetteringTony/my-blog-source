---
title: Hot 100-day1—哈希&双指针&滑动窗口
date: 2025-10-12 17:59:11
mathjax: true
categories: [数据结构与算法]
tags: [leetcode hot100]
---

> 题解代码思路绝大部分参考leetcode灵神的方法

## 1、两数之和

> 给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 和为目标值 `target` 的那两个整数，并返回它们的数组下标。你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。你可以按任意顺序返回答案。

- 代码

  ```java
  class Solution {
      public int[] twoSum(int[] nums, int target) {
          // 哈希表保存已经遍历过的数，初始为空
          Map<Integer, Integer> visitedNum = new HashMap<>();
          for (int i = 0; i < nums.length; i++) {
              int wanted = target - nums[i];
              // 一些数中找某一个数
              if (visitedNum.containsKey(wanted)) {
                  return new int[] {visitedNum.get(wanted), i};
              }
              // 将这次的枚举值放入表中，便于后续遍历使用
              visitedNum.put(nums[i], i);
          }
          return new int[] {-1, -1};
      }
  }
  ```

- 思路

  首先很容易想到两层遍历$O(n^2)$的解法，不再赘述。对于`nums[i] + nums[j] = target`这种需要两个枚举变量来判断条件是否成立的，可以尝试着对式子变形`nums[i] = target - nums[j]`，这样在枚举`j`时，就变成了在一些数里面判断是否存在某一个数，哈希表或者哈希集合就非常适合这样的功能。之所以这里选择哈希表，是因为题目要求返回对应数字的下标，因此使用哈希表来保存数字和对应下标的映射。每次向右枚举时，先计算出我们期望的值，然后在哈希表中寻找是否存在，如果存在，就直接返回下标；如果不存在，就把这个枚举的值加入哈希表，便于后续判断时使用。

- 心得体会

  对于需要通过`a + b = c`这种条件来得到答案的题目，都可以联想到“两数之和”中引入哈希表这一数据结构，从而一些数中找某一个数，为了便于写代码，通常采用“枚举右边，在左边进行寻找是否存在特定的数”。

## 49、字母异位词分组

> 给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。

- 代码

  ```java
  class Solution {
      public List<List<String>> groupAnagrams(String[] strs) {
          // 初始化哈希表用于分组
          Map<String, List<String>> group = new HashMap<>();
          for (int i = 0; i < strs.length; i++) {
              String cur = strs[i];
              // 将当前字符串按照字典序排序
              char[] chars = cur.toCharArray();
              Arrays.sort(chars); 
              String sorted =new String(chars);
              // 判断该字典序分组key是否已经存在
              // 如果存在，返回已有的值，添加当前字符串到这个组
              // 如果不存在，按照箭头函数给这个key新插入初始化值，再添加当前字符串
              group.computeIfAbsent(sorted, k -> new ArrayList<>()).add(cur);
          }
          return new ArrayList<>(group.values());
      }
  }
  ```

- 思路

  关键在于对“字母异位词”的理解，如果两个字符串符合上述条件，那么它们按照字典序排序之后的字符串必然是相同的。也就是说只要多个字符串字典序相同，那么它们就属于一个组。很自然的想到使用哈希表这一数据结构，键就是字典序，值就是对应的同组字符串。依次遍历字符串数组，对每一个字符串进行字典序处理，再判断哈希表中是否已经有对应的分组，如果该键已经存在，就把当前字符串添加到对应组中；如果这个键不存在，说明是一个新的组，就新增一个键值对。最后取出哈希表的值集合。

- 心得体会

  这道题我们可以得出一个通用思路：存在一些字符串，如何判断这些字符串仅仅只是字符的排列顺序不同？首先判断字符串的长度是否相等。如果长度都不相等的话，说明不可能只是交换了字符顺序的差异。其次进一步判断每个字符串对应的字典序是否相同。得到一个字符串的字典序字符串步骤如下：1、将其转化为字符数组（`char[] s = str.toCharArray()`），2、对该字符数组进行排序，得到字符的字典序数组（`Arrays.sort(s)`），3、再将该字典序数组重新转为字符串（`String sortedStr = new String(s)`）。如果这些字符串确实是由同样的字符组成而仅仅是顺序不同的话，那么各自得到的字典序字符串内容必然是相同的。如果有进一步需求，比如需要将这类原始字符串放在一个列表，那么可以使用哈希表进行分组，`sortedStr`为键，同类原始字符串为值。

## 128、最长连续序列

> 给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。请你设计并实现时间复杂度为$O(n)$的算法解决此问题。

- 代码

  ```java
  class Solution {
      public int longestConsecutive(int[] nums) {
          Set<Integer> diffNums = new HashSet<>();
          // 利用哈希集合对原数组去重
          for (int i = 0; i < nums.length; i++) {
              diffNums.add(nums[i]);
          }
  
          int res = 0;
          // 需要遍历去重后的哈希集而不是原数组
          for (int x : diffNums) {
              // 如果x - 1在集合中，那么就结束当前x为起点的序列
              if (diffNums.contains(x - 1)) {
                  continue;
              }
              // x为当前序列起点，开始计算长度
              int next = x + 1;
              while (diffNums.contains(next)) {
                  next++;
              }
              // 更新答案，取最大值
              res = Math.max(res, next - x);
          }
          return res;
      }
  }
  ```

- 思路

  第一反应可能是先排序再处理，而内置排序算法复杂度为$O(nlog(n))$，题目要求$O(n)$复杂度，不符合。核心的思路是遍历`nums`中的每一个数x，去判断`x + 1`，`x + 2`，·······是否在`nums`中，维护一个最大序列长度。为了满足要求的时间复杂度，需要考虑两个优化：1、原数组中数字可能有重复，而针对这道题，重复性对正确答案没有影响，反而会降低原数组的遍历效率。因此想到引入哈希集合来去重，后续遍历集合即可。2、对于集合中每一个`x`，没有必要都遍历一遍，如果`x - 1`在集合中，那么必然以`x - 1`为起点的序列会比`x`长，因此该情况下可以提前终止`x`的遍历。

- 心得体会

  分析题目之后，如果数组的重复性对答案没有影响，可以引入哈希集合去重。由于集合的无序性，需要注意哈希集的遍历方式：`Set<Integer> set = new HashSet<>(); for (int x : set)`

## 283、移动零

> 给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。请注意 ，必须在不复制数组的情况下原地对数组进行操作。

- 代码

  ```java
  class Solution {
      public void moveZeroes(int[] nums) {
          // 初始化一个指针指向需要补0的起始位置
          int zeroIdx = 0;
          for (int i = 0; i < nums.length; i++) {
              if (nums[i] != 0) {
                  nums[zeroIdx] = nums[i];
                  // 更新指针位置
                  zeroIdx++;
              }
          }
          // 根据指针补全末尾0
          Arrays.fill(nums, zeroIdx, nums.length, 0);
      }
  }
  ```

- 思路

  首先维护一个指针，指向需要补0的起始索引，初始化默认为0。然后遍历原始数组，如果不是0，那么就把这个值移动到该索引位置，同时这个指针需要向后移动一位，表示前面有非零数，需要更新补0的位置。遍历完成之后，根据这个指针位置来补全末尾0即可。

## 11、盛最多水的容器

> 给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。返回容器可以储存的最大水量。说明：你不能倾斜容器。

- 代码

  ```java
  class Solution {
      public int maxArea(int[] height) {
          // 初始化两个指针位于数组两端
          int left = 0;
          int right = height.length - 1;
          int area = 0;
          // 指针不能重合，否则搜索完成
          while (left < right) {
              int h = Math.min(height[left], height[right]);
              area = Math.max(area, (right - left) * h);
              // 左边 <= 右边，只有向右移动左边才有可能增大面积，相反情况类似
              if (height[left] <= height[right]) {
                  left++;
              } else {
                  right--;
              }
          }
          return area;
      }
  }
  ```

- 思路

  典型的双指针问题。比较费解的地方在于为什么能够想到一开始就将两个指针分别放在左右两端而不是某个其他位置？个人理解如下：题目要求最大的面积，根据面积公式$S = l * h$，有两个变量来决定该面积。在搜索过程中，高度`h`的变化趋势是不知道的，而将指针初始放在两端，这样宽度`l`初始化就最大，后续移动边时，`l`的变化趋势是固定减小的，也就是说只有可能搜索到更大的较短边时才会让该面积增大。此外，计算出一个面积之后，移动哪条边需要注意：如果`left <= right`，已知宽度`l`必定减小，如果将`right`向左移动，那么比较标准依然是`left`，这样面积必然是会减小的，而如果将`left`向右移动，虽然宽度`l`减小了，但是有可能会出现更长的较短边，面积有一定的概率增大。`left > right`的情况思考同理。这样搜索完成就能找出最大的面积。

## 15、三数之和

> 给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。注意：答案中不可以包含重复的三元组。输出的顺序和三元组的顺序并不重要。

- 代码

  ```java
  class Solution {
      public List<List<Integer>> threeSum(int[] nums) {
          // 顺序不重要，同时要解决重复性问题，因此先排序来优化后续搜索
          Arrays.sort(nums);
          List<List<Integer>> res = new ArrayList<>();
          int len = nums.length;
          // 由于是三数之和，需要查看当前x后面两个数，因此边界条件设置为len - 2
          for (int i = 0; i < len - 2; i++) {
              int x = nums[i];
              // 如果x和前一个数相同，说明之前已经处理过，避免重复，直接continue
              if (i > 0 && x == nums[i - 1]) {
                  continue;
              }
              // 优化一：如果当前最小的组合都大于0，并且由于x会递增，不再会有符合条件的三数，break
              if (x + nums[i + 1] + nums[i + 2] > 0) {
                  break;
              }
              // 优化二：如果当前最大的组合都小于0，说明这一轮可以提前终止
              // 但是x递增，后续可能出现符合条件的三数，因此只是continue结束当前
              if (x + nums[len - 2] + nums[len - 1] < 0) {
                  continue;
              }
              // 两个指针，当前x的下一个和最后一个
              int j = i + 1;
              int k = len - 1;
              while (j < k) {
                  int s = x + nums[j] + nums[k];
                  if (s > 0) {
                      k--; // 大于目标值0，需要将k左移探索更小的数
                  } else if (s < 0) {
                      j++; // 小于目标值0，需要将j右移探索更大的数
                  } else {
                      res.add(List.of(x, nums[j], nums[k]));
                      // 找到一个三元组，同时更新j、k
                      // 循环条件进行防重复处理：如果重复，继续移动
                      for (j++; j < k && nums[j] == nums[j - 1]; j++);
                      for (k--; k > j && nums[k] == nums[k + 1]; k--);
                  }
              }
          }
          return res;
      }
  }
  ```

- 思路

  在三数之和时，先枚举一个数`nums[i]`，那么就转化为了`nums[j] + nums[k] = -nums[i]`，就回到了熟悉的“两数之和”。同时，该题不要求三元组的返回顺序以及不能重复，相比于普遍的“两数之和”，可以对数组进行排序来优化两个数的寻找以及解决重复搜索的问题。默认按照从小到大的排序之后，整个数组总体上就是单调递增的趋势。针对优化一，对于枚举的`x = nums[i]`，`nums[i + 1]  + nums[i + 2]`是待搜索数中最小的两个数，如果`x + nums[i + 1] + nums[i + 2]` 都已经大于0，同时排序之后`x`的枚举变化趋势是递增的，那么就意味着后续不可能再次出现满足条件的三元组了，因此直接`break`进行返回。针对优化二，对于枚举的`x = nums[i]`，`nums[len - 2] + nums[len - 1]`是待搜索数中最大的两个数，如果`x + nums[len - 2] + nums[len - 1]`都小于0，就表明当前`x`没有必要在后续的数中搜索另外两个数，但是由于`x`的趋势是变大，后续新的枚举`x`可能会让`x + nums[len - 2] + nums[len - 1]`大于等于0，需要进一步搜索满足条件的两个数，因此只是`continue`结束当前`x`的处理，后续对于新的增大的`x`值，还是需要继续处理。

## 42、接雨水

> 给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

- 代码

  ```java
  class Solution {
      public int trap(int[] height) {
          int len = height.length;
          // preMax[i]表示height[i]前面出现的最大值
          int[] preMax = new int[len];
          // sufMax[i]表示height[i]后面出现的最大值
          int[] sufMax = new int[len];
  
          // 初始化值: 第一个左边和最后一个右边均设置为0
          preMax[0] = 0;
          sufMax[len - 1] = 0;
          for (int i = 1; i < len; i++) {
              preMax[i] = Math.max(preMax[i - 1], height[i - 1]);
          }
          for (int i = len - 2; i >= 0; i--) {
              sufMax[i] = Math.max(sufMax[i + 1], height[i + 1]);
          }
  
          int ans = 0;
          for (int i = 0; i < len; i++) {
              // 木桶效应: 取出当前柱子左右两边较小值，如果当前高度小于该值，说明可以装水
              int minHeight = Math.min(preMax[i], sufMax[i]);
              if (minHeight > height[i]) {
                  ans += minHeight - height[i];
              }
          }
          return ans;
      }
  }
  ```

- 思路

  以“列向”视角来看待数组中每个高度，对于一个高度值，如果它左边和右边的最大值均大于该高度，说明当前能够积累水，根据木桶效应，能够装多少水取决于左右最大值中较小值与当前高度的差值。因此，需要遍历该数组，存储每一个高度左边和右边的最大值，就很自然的想到了前后缀分解方法，以空间换时间，保存数组每个元素的最大前缀和最大后缀。

## 3、无重复字符的最长字串

> 给定一个字符串 `s` ，请你找出其中不含有重复字符的最长子串的长度。

- 代码

  ```java
  class Solution {
      public int lengthOfLongestSubstring(String s) {
          int res = 0;
          char[] c = s.toCharArray();
          // 初始化左边界位置0
          int left = 0;
          // 用哈希表来存储左右边界构成的子串的每一个字符出现频次
          HashMap<Character, Integer> cnt = new HashMap<>();
          // 枚举右边界，探索最长字串长度
          for (int right = 0; right < c.length; right++) {
              char in = c[right];
              // 对于新进入窗口的元素，更新对应的出现频次
              cnt.put(in, cnt.getOrDefault(in, 0) + 1);
              // 如果新进入的元素破坏了条件，需要收缩左边界，直到新的符合条件窗口形成
              while (cnt.get(in) > 1) {
                  cnt.put(c[left], cnt.get(c[left]) - 1);
                  if(cnt.get(c[left]) == 0) {
                      cnt.remove(c[left]);
                  }
                  // 收缩左边界，需要循环判断收缩之后是否满足条件
                  left++;
              }
              // 对于符合条件的窗口，更新最终结果
              res = Math.max(res, right - left + 1);
          }
          return res;
      }
  }
  ```

- 思路

  经典的不定长滑动窗口问题，不定长滑动窗口主要分为3类：求最长子数组，求最短子数组，以及求子数组个数。不定长滑动窗口往往由`left`、`right`来表示窗口的左右两端，窗口长度`L`不固定，由`right - left + 1`来动态计算。往往是初始化`left`为数组起始位置0，依次枚举右端点`right`。当每次新元素加入时，都需要结合题目问题更新某种条件（本题是要更新子串中对应字符的出现频次），然后判断是否破坏了窗口的约束条件，如果破坏了，就可能需要移除左端元素（移除时需要对条件进行更新），同时对左端点`left`进行右移，缩小窗口，循环判断新的窗口内元素是否满足约束条件。最后完全遍历右端点`right`结束。

- 心得体会

  滑动窗口需要满足窗口内部“单调性”，也就是说，当右端点元素进入窗口时，窗口元素和是不能减少的，新元素的加入必须使得窗口向着不满足约束条件的方向前进。如果不满足这个性质，可以考虑使用前缀和思想。

## 438、找到字符串中所有字母异位词

> 给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 异位词 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。`s` 和 `p` 仅包含小写字母

- 代码

  ```java
  class Solution {
      public List<Integer> findAnagrams(String s, String p) {
          List<Integer> ans = new ArrayList<>();
          // 统计 p 的每种字母的出现次数
          int[] cntP = new int[26]; 
          // 统计 s 的长为 p.length() 的子串 s' 的每种字母的出现次数
          int[] cntS = new int[26]; 
          for (char c : p.toCharArray()) {
              cntP[c - 'a']++; // 统计 p 的字母
          }
          for (int right = 0; right < s.length(); right++) {
              // 右端点字母进入窗口
              cntS[s.charAt(right) - 'a']++; 
              int left = right - p.length() + 1;
              // 窗口长度不足 p.length()，尚未构成第一个窗口；构成第一个窗口之后，开始滑动进行判断
              if (left < 0) { 
                  continue;
              }
              // 构成窗口之后，每个窗口都需要判断
              if (Arrays.equals(cntS, cntP)) { // s' 和 p 的每种字母的出现次数都相同
                  ans.add(left); // s' 左端点下标加入答案
              }
              // 当前窗口判断完毕，无论是否满足条件，由于窗口长度固定，左端点字母均要离开窗口
              cntS[s.charAt(left) - 'a']--; 
          }
          return ans;
      }
  }
  ```

- 思路

  可以采用定长滑动窗口思想。枚举 `s` 的所有长为 `p.length()` 的子串 `s'` ，如果 `s'` 的每种字母的出现次数，和 `p` 的每种字母的出现次数都相同，那么 `s` 是 `p` 的异位词。
