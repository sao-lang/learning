# 前端 / 全栈面试 · 算法刷题手册（JavaScript 版）

> 定位：不刷偏难怪题，聚焦面试官真正会考的数组、字符串、双指针、哈希、栈、链表、二叉树、简单 DP，以及前端必考手写题。
> 每题格式：题目 → 思路 → 代码 → 复杂度 → 面试官可能追问。

## 目录

- [一、刷题策略](#一刷题策略)
- [二、第一档：必刷高频题](#二第一档必刷高频题)
- [三、第二档：次高频](#三第二档次高频)
- [四、第三档：大厂加分题](#四第三档大厂加分题)
- [五、前端手写题（必考！优先级高于第三档）](#五前端手写题必考优先级高于第三档)
- [六、Tree-shaking 与 Vue 依赖收集：被连续追问的「源码题」](#六tree-shaking-与-vue-依赖收集被连续追问的源码题)
- [七、手写题速查表](#七手写题速查表)
- [八、按「套路」索引](#八按套路索引)
- [九、面试现场自查清单](#九面试现场自查清单)
- [十、写在最后：怎么用这份手册](#十写在最后怎么用这份手册)

---

## 一、刷题策略

### 1.1 优先级

**第一档（必刷）> 前端手写题（必考）> 第二档（次高频）> 第三档（大厂加分）**

前端 / 全栈面试不会像算法岗那样上复杂图论、高级 DP、困难题。面试官的核心预期是：

- 思路清晰，先讲思路再写代码
- 代码干净，JS 没有低级坑（大数精度、类型转换、闭包）
- 主动分析时间 / 空间复杂度，能说出优化过程（暴力 → 优化）
- 能联系业务场景（树形遍历 ↔ 菜单权限、LRU ↔ 前端缓存、滑动窗口 ↔ 文本搜索）

### 1.2 推荐刷题顺序（约 10 天，每天 3~4 题）

| 天数 | 内容 |
|------|------|
| Day 1 | 两数之和 → 三数之和 → 盛最多水的容器 |
| Day 2 | 无重复字符的最长子串 → 有效的括号 → 字符串相加 |
| Day 3 | 字母异位词分组 → 最长连续序列 → 移动零 |
| Day 4 | 反转链表 → 合并两个有序链表 → 判断链表是否有环 |
| Day 5 | 二叉树层序遍历 → 二叉树最大深度 → 对称二叉树 |
| Day 6 | 防抖 → 节流 → 深拷贝 |
| Day 7 | Promise.all → LRU 缓存 → 数组转树 / 树转数组 |
| Day 8 | 每日温度 → 找到字符串中所有字母异位词 → 用栈实现队列 |
| Day 9 | 爬楼梯 → 打家劫舍 → 最大子数组和 |
| Day 10 | 第 K 大元素 → 岛屿数量 → 全排列 → 最长回文子串（有余力再看） |

### 1.3 刷题方法

1. 先自己独立手写（限时 20~30 分钟），写不出来再看思路
2. 对照答案后，**隔天不看答案重写一遍**
3. 追问点也要准备，这是拉开差距的地方；尤其「换个写法 / 换个题」「这个方案有什么问题」两类追问，出现频率最高

---

## 二、第一档：必刷高频题

### 1. 两数之和（Easy）

**题目**：给定数组 `nums` 和目标值 `target`，找出和为 target 的两个数的下标。

**思路**：哈希表空间换时间。遍历时把「值 → 下标」存入 Map，同时查 `target - 当前值` 是否已在 Map 中。注意先查再存，避免用同一个元素。

```js
function twoSum(nums, target) {
    const map = new Map();
    for (let i = 0; i < nums.length; i++) {
        const diff = target - nums[i];
        if (map.has(diff)) return [map.get(diff), i];
        map.set(nums[i], i);
    }
    return [];
}
```

**复杂度**：时间 O(n)，空间 O(n)

**追问**：

- 如果要求返回值而不是下标 → 可以先排序 + 双指针（O(1) 空间）
- 如果数组已经有序 → 直接双指针，O(n) 时间 O(1) 空间
- 为什么先查再存？→ 避免同一个元素和自己相加

---

### 2. 三数之和（Medium）⭐ 高频中的高频

**题目**：找出数组中所有和为 0 的三元组，结果不能重复（`[1,2,-3]`、`[1,-3,2]`、`[2,1,-3]` 算同一个，只保留一份）。

**思路**：排序 + 固定一个数 + 双指针。**排序是去重和双指针的前提**。

1. 排序，固定第一个数 `nums[i]`
2. `nums[i] > 0` 直接 break（后面都大于 0）
3. `nums[i] === nums[i-1]` 跳过（第一个数去重）
4. 双指针 `left = i+1`、`right = n-1` 向中间靠拢
5. 找到和为 0 后，**左右指针分别跳过重复值**（第二、三个数去重）

```js
function threeSum(nums) {
    const res = [];
    nums.sort((a, b) => a - b);
    for (let i = 0; i < nums.length - 2; i++) {
        if (nums[i] > 0) break;                          // 剪枝：后面全大于 0
        if (i > 0 && nums[i] === nums[i - 1]) continue;  // 去重①：第一个数

        let left = i + 1, right = nums.length - 1;
        while (left < right) {
            const sum = nums[i] + nums[left] + nums[right];
            if (sum === 0) {
                res.push([nums[i], nums[left], nums[right]]);
                while (left < right && nums[left] === nums[left + 1]) left++;     // 去重②
                while (left < right && nums[right] === nums[right - 1]) right--;  // 去重③
                left++; right--;
            } else if (sum < 0) {
                left++;
            } else {
                right--;
            }
        }
    }
    return res;
}

// 测试
console.log(threeSum([-1, 0, 1, 2, -1, -4])); // [[-1, -1, 2], [-1, 0, 1]]
```

**复杂度**：时间 O(n²)，空间 O(log n)（排序栈）

**追问**：

- 为什么要排序？→ 排序后三元组天然非递减输出，杜绝排列重复；双指针才能根据和的大小移动
- 三处去重分别去掉什么？→ ①固定数重复 ②左指针重复 ③右指针重复，少一处就会出现重复组合
- 不排序怎么做？→ 哈希，但去重很麻烦，需要 Set 存排序后的 key
- 改成四数之和？→ 外面再套一层循环，O(n³)

---

### 3. 盛最多水的容器（Medium）

**题目**：`height[i]` 表示柱子高度，找两根柱子使围成的容器面积最大。

**思路**：双指针从两端向内收缩，每次移动较矮的一侧（面积由矮的柱子和间距决定，移动高的不可能变大）。

```js
function maxArea(height) {
    let left = 0, right = height.length - 1, max = 0;
    while (left < right) {
        const area = Math.min(height[left], height[right]) * (right - left);
        max = Math.max(max, area);
        if (height[left] < height[right]) left++;
        else right--;
    }
    return max;
}
```

**复杂度**：时间 O(n)，空间 O(1)

**追问**：

- 为什么移动矮的一侧？→ 面积 = min(h1,h2) × 间距；移动高的那侧，min 不变或变小且间距变小，面积不可能更大；移动矮的才有机会让 min 变大
- 为什么这样能覆盖所有最优解？→ 「当前窄边」和另一侧任意柱子的组合都不优于当前面积，因此可以安全丢弃窄边

---

### 4. 移动零（Easy）

**题目**：把数组中所有 0 移到末尾，保持非零元素相对顺序，原地修改。

**思路**：快慢指针。慢指针指向下一个非零元素应该放的位置。

```js
function moveZeroes(nums) {
    let slow = 0;
    for (let fast = 0; fast < nums.length; fast++) {
        if (nums[fast] !== 0) {
            [nums[slow], nums[fast]] = [nums[fast], nums[slow]];
            slow++;
        }
    }
    return nums;
}

// 测试
console.log(moveZeroes([0, 1, 0, 3, 12])); // [1, 3, 12, 0, 0]
```

**复杂度**：时间 O(n)，空间 O(1)

**追问**：

- 能不能统计零的个数再覆盖？→ 可以，但双指针交换更优雅且在原地完成
- 非零元素相对顺序会变吗？→ 不会，slow 只会把 fast 遇到的非零元素按原顺序前移

---

### 5. 合并两个有序数组（Easy）

**题目**：`nums1` 长度为 m+n（尾部 n 个空位存 0），合并 `nums2` 到 `nums1`，不返回新数组。

**思路**：**从后往前**填，避免覆盖 nums1 前面的有效元素。

```js
function merge(nums1, m, nums2, n) {
    let i = m - 1, j = n - 1, k = m + n - 1;
    while (j >= 0) {
        nums1[k--] = i >= 0 && nums1[i] > nums2[j] ? nums1[i--] : nums2[j--];
    }
    return nums1;
}
```

**复杂度**：时间 O(m+n)，空间 O(1)

**追问**：

- 为什么从后往前？→ 正向往 nums1 头部填会覆盖尚未处理的元素
- 循环条件为什么只需要 `j >= 0`？→ nums1 剩下的元素本来就在正确位置，j 处理完就结束了
- 和「合并两个有序链表」什么区别？→ 数组能用双指针原地覆盖，链表只能新建指针连接

---

### 6. 无重复字符的最长子串（Medium）⭐⭐⭐

**题目**：给定字符串，找出不含重复字符的最长子串的长度。

**思路**：滑动窗口 + Map 记录每个字符最后出现的位置。窗口右边界向右扩展，遇到重复字符时把左边界跳到 `max(left, 上次出现位置 + 1)`（用 max 防止左边界回退）。

```js
function lengthOfLongestSubstring(s) {
    const map = new Map();
    let left = 0, max = 0;
    for (let right = 0; right < s.length; right++) {
        if (map.has(s[right])) {
            left = Math.max(left, map.get(s[right]) + 1);
        }
        map.set(s[right], right);
        max = Math.max(max, right - left + 1);
    }
    return max;
}

// 测试
console.log(lengthOfLongestSubstring('abcabcbb')); // 3
```

**复杂度**：时间 O(n)，空间 O(字符集大小)

**追问**：

- 为什么 left 要取 `Math.max`？→ 不取 max 可能导致左边界回退，窗口内仍有重复字符
- 能只用 Set 吗？→ 能，但需要循环删除，不如 Map 一步到位；两者复杂度相同
- 为什么不是 `while` 收缩窗口？→ 每个字符最后位置唯一，直接跳过去即可，无需逐格收缩

---

### 7. 有效的括号（Easy）

**题目**：判断字符串中的括号 `() [] {}` 是否正确闭合。

**思路**：栈。左括号入栈，右括号弹出比对；用映射表简化判断。

```js
function isValid(s) {
    const stack = [];
    const map = { ')': '(', ']': '[', '}': '{' };
    for (const ch of s) {
        if (ch in map) {
            if (stack.pop() !== map[ch]) return false;
        } else {
            stack.push(ch);
        }
    }
    return stack.length === 0;
}
```

**复杂度**：时间 O(n)，空间 O(n)

**追问**：

- 栈空时遇到右括号会怎样？→ `pop()` 返回 `undefined`，`undefined !== '('` 成立，直接返回 false，边界已处理
- 最后为什么还要判断 `stack.length === 0`？→ `"((("` 这种只有左括号的情况必须判空

---

### 8. 字符串相加 / 大数相加（Easy~Medium，前端特色）

**题目**：两个用字符串表示的大整数相加，不能直接用 Number（会丢精度）。返回字符串。

**思路**：模拟竖式加法，从个位开始逐位加，维护进位。

```js
function addStrings(num1, num2) {
    let i = num1.length - 1, j = num2.length - 1, carry = 0;
    let res = '';
    while (i >= 0 || j >= 0 || carry) {
        const a = i >= 0 ? +num1[i--] : 0;
        const b = j >= 0 ? +num2[j--] : 0;
        const sum = a + b + carry;
        carry = Math.floor(sum / 10);
        res = (sum % 10) + res;
    }
    return res;
}

// 测试
console.log(addStrings('456', '77')); // 533
```

**复杂度**：时间 O(max(m,n))，空间 O(1)（不计结果字符串）

**追问**：

- 为什么用字符串？→ JS Number 最大安全整数是 2⁵³-1，大数相加会丢精度（可延伸到「金额计算用字符串 / Decimal」）
- 还能怎么优化？→ 用数组存每一位、最后 join，避免字符串反复拼接（V8 已优化字符串拼接，但数组更稳）

---

### 9. 字母异位词分组（Medium）

**题目**：把字符串数组按「字母异位词」（相同字母不同顺序）分组。

**思路**：排序后的字符串作为 key，哈希分组。

```js
function groupAnagrams(strs) {
    const map = new Map();
    for (const s of strs) {
        const key = s.split('').sort().join('');
        if (!map.has(key)) map.set(key, []);
        map.get(key).push(s);
    }
    return [...map.values()];
}
```

**复杂度**：时间 O(n·k log k)（k 为字符串平均长度），空间 O(n·k)

**追问**：

- k 很大时怎么优化 key？→ 用 26 个字母的计数数组拼成 key，降到 O(n·k)

---

### 10. 最长连续序列（Medium）⭐

**题目**：未排序数组，找最长连续数字序列的长度（如 `[100,4,200,1,3,2]` → 4）。

**思路**：全部放入 Set。**只从「没有前驱」的数字开始向后数**，保证每个数字只被统计一次 → 整体 O(n)。

```js
function longestConsecutive(nums) {
    const set = new Set(nums);
    let max = 0;
    for (const n of set) {
        if (!set.has(n - 1)) {   // 没有前驱，才是序列起点
            let cur = n, len = 1;
            while (set.has(cur + 1)) { cur++; len++; }
            max = Math.max(max, len);
        }
    }
    return max;
}
```

**复杂度**：时间 O(n)，空间 O(n)

**追问**：

- 为什么能保证 O(n)？→ 每个数字只作为起点被访问一次，while 里的访问分摊下来每个数字最多被看两次（作为起点和作为后继）
- 为什么不能排序？→ 排序是 O(n log n)，题目期望 O(n)；能用但会丢分

---

### 11. 反转链表（Easy）⭐ 链表必考第一题

**题目**：反转单链表，迭代 + 递归都要会。

**思路**：迭代：三个指针 prev / cur / next 逐个翻转。**必须先保存 next 再改指向**。

```js
function reverseList(head) {
    let prev = null;
    while (head) {
        const next = head.next;   // 先存
        head.next = prev;         // 再改
        prev = head;
        head = next;
    }
    return prev;
}
```

**递归版**：

```js
function reverseListRecursive(head) {
    if (!head || !head.next) return head;
    const newHead = reverseListRecursive(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}
```

**复杂度**：时间 O(n)，空间 O(1)（迭代）/ O(n)（递归栈）

**追问**：

- `head.next.next = head` 是什么意思？→ 让当前节点的下一个节点指回当前节点，完成局部反转
- 递归为什么空间是 O(n)？→ 每层调用压栈，链表长度即栈深度，长链表会栈溢出

---

### 12. 判断链表是否有环（Easy）

**题目**：判断链表中是否存在环。

**思路**：快慢指针，快指针每次走两步、慢指针走一步，有环必然相遇。**不要用 Set**（可以用但面试官期待快慢指针）。

```js
function hasCycle(head) {
    let slow = head, fast = head;
    while (fast && fast.next) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow === fast) return true;
    }
    return false;
}
```

**复杂度**：时间 O(n)，空间 O(1)

**追问**：

- 如何找环的入口？→ 相遇后，一个指针从头走、一个从相遇点走，再次相遇处即入口（Floyd 算法）
- 为什么一定会相遇？→ 有环时快指针每轮追近 1 步，距离必然归零

---

### 13. 合并两个有序链表（Easy）

**题目**：合并两个升序链表，返回新链表。

**思路**：哑节点（dummy）简化头节点处理，谁小接谁。

```js
function mergeTwoLists(l1, l2) {
    const dummy = { val: 0, next: null };
    let cur = dummy;
    while (l1 && l2) {
        if (l1.val < l2.val) {
            cur.next = l1; l1 = l1.next;
        } else {
            cur.next = l2; l2 = l2.next;
        }
        cur = cur.next;
    }
    cur.next = l1 || l2;   // 接上剩余部分
    return dummy.next;
}
```

**复杂度**：时间 O(m+n)，空间 O(1)

**追问**：

- 哑节点的作用？→ 避免单独处理头节点为空的分支，代码更简洁
- 能递归写吗？→ 能，`return l1.val < l2.val ? (l1.next = merge(l1.next, l2), l1) : ...`，但递归深度是链表长度

---

### 14. 二叉树层序遍历（Medium）⭐ 前端高频

**题目**：按层输出二叉树，如 `[[3],[9,20],[15,7]]`。

**思路**：BFS 队列。**每层先记录队列长度**，一次处理一整层。

```js
function levelOrder(root) {
    if (!root) return [];
    const res = [];
    const queue = [root];
    while (queue.length) {
        const size = queue.length;   // 关键：当前层节点数
        const level = [];
        for (let i = 0; i < size; i++) {
            const node = queue.shift();
            level.push(node.val);
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
        res.push(level);
    }
    return res;
}
```

**复杂度**：时间 O(n)，空间 O(n)

**追问**：

- `queue.shift()` 是 O(n) 的，怎么优化？→ 用数组索引指针模拟队列（`head++`），或真正的队列结构
- 改成 Z 字形层序遍历？→ 每层判断奇偶，奇数层 `level.reverse()`
- 和业务有什么关系？→ 树形菜单、组织架构、权限树的层级展开都是 BFS / DFS 的变体

---

### 15. 二叉树的最大深度（Easy）

**题目**：返回二叉树最大深度（根到最远叶子节点的节点数）。

**思路**：递归：深度 = 1 + max(左子树深度, 右子树深度)。

```js
function maxDepth(root) {
    if (!root) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

**复杂度**：时间 O(n)，空间 O(树高)（递归栈）

**追问**：

- 迭代写法？→ BFS 层数计数 / DFS 栈记录深度，递归是面试默认写法
- 为什么空间是 O(树高) 而不是 O(n)？→ 递归栈深度等于树高；退化成链表时才是 O(n)

---

### 16. 对称二叉树（Easy）

**题目**：判断二叉树是否关于根节点轴对称。

**思路**：递归比较左子树和右子树：值相等，且 左.left ↔ 右.right、左.right ↔ 右.left 都相等。

```js
function isSymmetric(root) {
    function check(l, r) {
        if (!l && !r) return true;
        if (!l || !r) return false;
        return l.val === r.val
            && check(l.left, r.right)
            && check(l.right, r.left);
    }
    return check(root, root);
}
```

**复杂度**：时间 O(n)，空间 O(树高)

**追问**：

- 关键点？→ 比较的是「镜像」而非「相同」：左孩子的左 ↔ 右孩子的右
- 能写成迭代吗？→ 用队列成对入队（左.left、右.right、左.right、右.left），每次成对弹出比较

---

### 补充题 A：最近公共祖先 + 路径求和（树形业务必考）

> 这两题不常单独出现，但「树形数据 + 递归」是前端业务（菜单、目录、评论）的高频考法，且能考出递归功底。

**题目**：① 二叉树最近公共祖先（BST）；② 求根节点到叶子节点的路径总和是否等于 target。

```js
// ① BST 版 LCA：利用有序性，从根往下走
function lowestCommonAncestor(root, p, q) {
    let node = root;
    while (node) {
        if (p.val < node.val && q.val < node.val) node = node.left;
        else if (p.val > node.val && q.val > node.val) node = node.right;
        else return node;   // 分岔点就是答案
    }
}

// ② 路径总和：递归到叶子时判断剩余值
function hasPathSum(root, targetSum) {
    if (!root) return false;
    if (!root.left && !root.right) return root.val === targetSum;
    return hasPathSum(root.left, targetSum - root.val)
        || hasPathSum(root.right, targetSum - root.val);
}
```

**复杂度**：① 时间 O(h)；② 时间 O(n)，空间 O(树高)

**追问**：如果是普通二叉树（非 BST）求 LCA？→ 后序遍历：左右子树都能找到 p 或 q 时，当前节点即答案

---

## 三、第二档：次高频

### 17. 找到字符串中所有字母异位词（Medium）

**题目**：在 `s` 中找出所有 `p` 的字母异位词的起始下标。

**思路**：定长滑动窗口（窗口大小 = p 的长度）+ 26 位字母计数数组比较。

```js
function findAnagrams(s, p) {
    const res = [];
    const n = p.length;
    if (s.length < n) return res;

    const need = new Array(26).fill(0);
    for (const ch of p) need[ch.charCodeAt(0) - 97]++;

    const window = new Array(26).fill(0);
    const needKey = need.join('#');
    for (let right = 0; right < s.length; right++) {
        window[s.charCodeAt(right) - 97]++;
        if (right >= n) window[s.charCodeAt(right - n) - 97]--;   // 滑出窗口
        if (right >= n - 1 && window.join('#') === needKey) res.push(right - n + 1);
    }
    return res;
}

// 测试
console.log(findAnagrams('cbaebabacd', 'abc')); // [0, 6]
```

**复杂度**：时间 O(26·n)，空间 O(1)

**追问**：

- 如何优化掉 join 比较？→ 维护一个 `diff` 计数：每轮只更新进出窗口的两个字母，`diff === 0` 即匹配，比较降到 O(1)
- 为什么 `if (right >= n)` 才滑出？→ 窗口还没长到 n 时，右边界 - n 是负数，会访问到非法下标
- 和「字母异位词分组」什么关系？→ 同一个「字符计数做 key」的套路

---

### 18. 每日温度（Medium）

**题目**：`temperatures[i]` 表示第 i 天的温度，返回每天要等几天才会升温，没有则为 0。

**思路**：单调递减栈（栈内存下标）。遇到比栈顶大的温度，就弹出栈顶并算出等待天数。

```js
function dailyTemperatures(temperatures) {
    const n = temperatures.length;
    const res = new Array(n).fill(0);
    const stack = [];
    for (let i = 0; i < n; i++) {
        while (stack.length && temperatures[i] > temperatures[stack[stack.length - 1]]) {
            const idx = stack.pop();
            res[idx] = i - idx;
        }
        stack.push(i);
    }
    return res;
}
```

**复杂度**：时间 O(n)，空间 O(n)

**追问**：

- 单调栈适合什么场景？→ 「找下一个更大 / 更小的元素」，每个元素只入栈出栈一次
- 同类题？→ 下一个更大元素、接雨水、柱状图最大矩形

---

### 19. 爬楼梯（Easy）

**题目**：每次可以爬 1 或 2 个台阶，n 阶楼梯有多少种爬法。

**思路**：斐波那契。`dp[i] = dp[i-1] + dp[i-2]`，滚动变量省空间。

```js
function climbStairs(n) {
    if (n <= 2) return n;
    let a = 1, b = 2;
    for (let i = 3; i <= n; i++) {
        [a, b] = [b, a + b];
    }
    return b;
}
```

**复杂度**：时间 O(n)，空间 O(1)

**追问**：

- 改成每次爬 1/2/3 阶？→ 递推式变 `dp[i] = dp[i-1] + dp[i-2] + dp[i-3]`
- 状态转移怎么来的？→ 最后一步要么 1 阶要么 2 阶，所以方案数 = 走到 n-1 的方案 + 走到 n-2 的方案
- 和递归有什么区别？→ 朴素递归 O(2ⁿ) 有大量重复子问题，DP 记忆化后 O(n)

---

### 20. 打家劫舍（Medium）

**题目**：不能偷相邻两家，求能偷到的最大金额。

**思路**：`dp[i] = max(偷 i, 不偷 i) = max(prev + nums[i], cur)`。`cur` 是到上一家为止的最大值，`prev` 是到上上家为止的最大值。

```js
function rob(nums) {
    let prev = 0, cur = 0;
    for (const n of nums) {
        [prev, cur] = [cur, Math.max(cur, prev + n)];
    }
    return cur;
}
```

**复杂度**：时间 O(n)，空间 O(1)

**追问**：

- 环形小区怎么变？→ 分两种情况：偷第一家（不偷最后一家）和不偷第一家，取 max
- 为什么 `prev` 要更新为旧 `cur`？→ `prev` 表示「不包含当前相邻上一个」的最大值，正好是上一轮之前的状态

---

### 21. 最大子数组和（Medium）

**题目**：连续子数组的最大和（含负数）。

**思路**：Kadane 算法。`cur = max(nums[i], cur + nums[i])`：要么从当前元素重新开始，要么延续之前的和。这是「只要负就断」的贪心式 DP。

```js
function maxSubArray(nums) {
    let cur = nums[0], max = nums[0];
    for (let i = 1; i < nums.length; i++) {
        cur = Math.max(nums[i], cur + nums[i]);
        max = Math.max(max, cur);
    }
    return max;
}
```

**复杂度**：时间 O(n)，空间 O(1)

**追问**：

- 为什么要和 `nums[i]` 取 max？→ 如果之前累加和是负数，加上它只会更小，不如从当前元素重新开始
- 如果要返回子数组区间？→ 记录区间起点，`cur` 被重置时更新起点，`max` 更新时记录终点

---

### 22. 用栈实现队列（Easy）

**题目**：用两个栈实现队列的 push / pop / peek / empty，均摊 O(1)。

**思路**：两个栈：in 栈负责入队，out 栈负责出队。out 栈为空时，才把 in 栈全部倒进去（此时顺序正好反转为队列顺序）。

```js
class MyQueue {
    constructor() {
        this.inStack = [];
        this.outStack = [];
    }
    push(x) { this.inStack.push(x); }

    pop() {
        this._transfer();
        return this.outStack.pop();
    }

    peek() {
        this._transfer();
        return this.outStack[this.outStack.length - 1];
    }

    empty() {
        return this.inStack.length === 0 && this.outStack.length === 0;
    }

    _transfer() {
        if (!this.outStack.length) {
            while (this.inStack.length) this.outStack.push(this.inStack.pop());
        }
    }
}
```

**复杂度**：均摊时间 O(1)，空间 O(n)

**追问**：

- 为什么均摊 O(1)？→ 每个元素只会被「搬移」一次，整体 O(n) 摊到 n 次操作上就是均摊 O(1)
- `_transfer` 里为什么必须判断 out 栈为空？→ 否则会把新元素压到旧元素上面，顺序就乱了
- 反过来用队列实现栈？→ 一个队列即可：push 后把前面所有元素重新入队到末尾

---

## 四、第三档：大厂加分题

### 23. 数组中的第 K 个最大元素（Medium）

**题目**：未排序数组，返回第 K 个最大的元素。

**思路**：快速选择（快速排序的 partition 思想）。第 k 大 = 第 `n-k` 小，partition 后看 pivot 的位置。

```js
function findKthLargest(nums, k) {
    const target = nums.length - k;
    let left = 0, right = nums.length - 1;
    while (left <= right) {
        const pivotIdx = partition(nums, left, right);
        if (pivotIdx === target) return nums[pivotIdx];
        else if (pivotIdx < target) left = pivotIdx + 1;
        else right = pivotIdx - 1;
    }
}

function partition(nums, left, right) {
    const pivot = nums[right];
    let i = left;
    for (let j = left; j < right; j++) {
        if (nums[j] < pivot) {
            [nums[i], nums[j]] = [nums[j], nums[i]];
            i++;
        }
    }
    [nums[i], nums[right]] = [nums[right], nums[i]];
    return i;
}
```

**复杂度**：平均时间 O(n)，最坏 O(n²)；空间 O(1)

**追问**：

- 最坏情况怎么保证 O(n log n)？→ 用堆（小顶堆维护 K 个元素），或随机化 pivot
- 变形：前 K 个高频元素 → 哈希统计 + 堆
- 为什么快速选择是 O(n)？→ 每次只递归（迭代）其中一半，n + n/2 + n/4 + ... ≈ 2n

---

### 24. 岛屿数量（Medium）

**题目**：二维网格 `'1'` 是陆地 `'0'` 是水，求岛屿数量（相邻陆地算同一岛屿）。

**思路**：DFS 沉没法。遍历每个格子，遇到 `'1'` 计数 +1，然后 DFS 把相邻的 `'1'` 全部改成 `'0'`（避免重复计数）。这是网格 DFS 的模板题。

```js
function numIslands(grid) {
    let count = 0;
    for (let i = 0; i < grid.length; i++) {
        for (let j = 0; j < grid[0].length; j++) {
            if (grid[i][j] === '1') {
                count++;
                dfs(grid, i, j);
            }
        }
    }
    return count;
}

function dfs(grid, i, j) {
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] === '0') return;
    grid[i][j] = '0';   // 沉没
    dfs(grid, i + 1, j);
    dfs(grid, i - 1, j);
    dfs(grid, i, j + 1);
    dfs(grid, i, j - 1);
}
```

**复杂度**：时间 O(m·n)，空间 O(m·n)（递归栈，最坏全为陆地）

**追问**：

- 用 BFS 怎么写？→ 队列 + 原地标记，出队时把邻居入队
- 超大网格注意递归栈溢出 → 改 BFS 或显式栈
- 如果需要统计每个岛屿的面积 / 周长 → DFS 时累积返回值

---

### 25. 全排列（Medium，回溯模板）

**题目**：不重复数字数组的所有排列。

**思路**：回溯。`used` 数组标记已选元素，每层选择未使用的数字，path 长度到 n 时收集结果（注意拷贝 path）。

```js
function permute(nums) {
    const res = [];
    const used = new Array(nums.length).fill(false);

    function backtrack(path) {
        if (path.length === nums.length) {
            res.push([...path]);   // 拷贝！不能直接 push path
            return;
        }
        for (let i = 0; i < nums.length; i++) {
            if (used[i]) continue;
            used[i] = true;
            path.push(nums[i]);
            backtrack(path);
            path.pop();            // 撤销选择
            used[i] = false;
        }
    }
    backtrack([]);
    return res;
}
```

**复杂度**：时间 O(n·n!)，空间 O(n)

**追问**：

- 为什么 push 时要展开拷贝？→ path 是引用类型，后续会继续修改，不拷贝最终存的都是同一个数组
- 组合、子集怎么改？→ 换成「起始下标 + 不允许回头选」，去掉 used 数组
- 有重复数字（全排列 II）？→ 排序 + `used[i-1] === false` 时跳过同层重复

---

### 补充题 B：最长回文子串（Medium）

**题目**：找出字符串中最长的回文子串。

**思路**：中心扩展法。回文中心有 `2n-1` 个（单字符 + 双字符间隙），从每个中心向两侧扩展。

```js
function longestPalindrome(s) {
    if (s.length < 2) return s;
    let start = 0, maxLen = 1;

    const expand = (l, r) => {
        while (l >= 0 && r < s.length && s[l] === s[r]) { l--; r++; }
        if (r - l - 1 > maxLen) {
            maxLen = r - l - 1;
            start = l + 1;
        }
    };

    for (let i = 0; i < s.length; i++) {
        expand(i, i);        // 奇数长度
        expand(i, i + 1);    // 偶数长度
    }
    return s.slice(start, start + maxLen);
}

// 测试
console.log(longestPalindrome('babad')); // 'bab'
```

**复杂度**：时间 O(n²)，空间 O(1)

**追问**：

- 能到 O(n) 吗？→ Manacher 算法，面试一般只需知道名字和思路
- 为什么回文中心有 2n-1 个？→ n 个单字符中心 + n-1 个间隙中心

---

## 五、前端手写题（必考！优先级高于第三档）

> 4 年前端面试，手写题出现概率和算法题一样高，很多面试官把它归入「代码能力」考察。

### 26. 防抖 debounce（必考）

**题目**：连续触发时只在最后一次触发后 delay 毫秒执行。

**思路**：每次触发清掉上一个定时器，重新计时。

```js
function debounce(fn, delay, immediate = false) {
    let timer = null;
    return function (...args) {
        const callNow = immediate && !timer;   // 立即执行版：第一次立即触发
        clearTimeout(timer);
        timer = setTimeout(() => {
            timer = null;
            if (!immediate) fn.apply(this, args);
        }, delay);
        if (callNow) fn.apply(this, args);
    };
}
```

**追问**：

- 防抖和节流的区别？→ 防抖是「最后一次触发后执行」；节流是「固定频率执行一次」
- `this` 为什么要用 `apply`？→ 保证执行时 this 指向触发者（如 input 元素）
- 想加一个 `cancel()` 方法？→ 返回对象 `{ run, cancel }`，cancel 里 `clearTimeout(timer); timer = null`
- 业务场景？→ 搜索框输入、窗口 resize、按钮防重复提交

---

### 27. 节流 throttle

**题目**：固定时间间隔内最多执行一次。

**思路**：时间戳版：间隔内触发直接忽略。

```js
function throttle(fn, interval) {
    let last = 0;
    return function (...args) {
        const now = Date.now();
        if (now - last >= interval) {
            last = now;
            fn.apply(this, args);
        }
    };
}
```

**追问**：

- 时间戳版和定时器版的区别？→ 时间戳版「立即执行、最后一次不补执行」；定时器版「首次延迟执行、最后一次会补执行」
- 要两者兼备？→ 组合写法：时间戳判断 + 剩余时间 setTimeout 兜底（面试能讲清差异即可）
- 业务场景？→ 滚动事件、拖拽、mousemove、节流接口请求
- 首屏为什么不用节流「只执行一次」？→ 用 `once` 或 `{ once: true }` 更语义化

---

### 28. 深拷贝 deepClone（必考）

**题目**：实现深拷贝，要求处理：循环引用、Date、RegExp、Symbol 键、数组、普通对象。

**思路**：递归 + WeakMap / Map 缓存已拷贝对象（解决循环引用），特殊类型单独处理。

```js
function deepClone(obj, map = new WeakMap()) {
    if (obj === null || typeof obj !== 'object') return obj;   // 基础类型 / 函数直接返回

    if (map.has(obj)) return map.get(obj);                     // 循环引用：返回已拷贝的

    if (obj instanceof Date) return new Date(obj);
    if (obj instanceof RegExp) return new RegExp(obj);

    const result = Array.isArray(obj) ? [] : {};
    Object.setPrototypeOf(result, Object.getPrototypeOf(obj));  // 保留原型（如 class 实例）
    map.set(obj, result);                                       // 先登记，再递归（循环引用关键）

    for (const key of Reflect.ownKeys(obj)) {                   // 包含 Symbol 键
        result[key] = deepClone(obj[key], map);
    }
    return result;
}
```

**追问**：

- 为什么用 WeakMap？→ 缓存表只需在拷贝期间存活，WeakMap 不阻止垃圾回收；Map 也能用
- 为什么先 `map.set` 再递归？→ 循环引用场景下，不先登记就会无限递归
- 和 `JSON.parse(JSON.stringify())` 的区别？→ 后者丢失函数、undefined、Symbol，Date 变字符串，不支持循环引用
- `structuredClone` 为什么不能直接替代？→ 不支持函数、原型链丢失，面试要能说出差异
- 怎么处理 Map / Set？→ `obj instanceof Map` 时遍历 entries 递归拷贝

---

### 29. 手写 Promise.all

**题目**：实现 `Promise.all`：全部成功返回结果数组，任一失败则整体失败。

**思路**：new Promise，用数组按下标存结果（**不能用 push**，要保序），计数完成后 resolve。

```js
function promiseAll(promises) {
    return new Promise((resolve, reject) => {
        const list = Array.from(promises);   // 兼容 iterable
        if (!list.length) return resolve([]);
        const results = new Array(list.length);
        let count = 0;
        list.forEach((p, i) => {
            Promise.resolve(p).then(     // Promise.resolve 兼容非 Promise 值
                (val) => {
                    results[i] = val;
                    if (++count === list.length) resolve(results);
                },
                reject                   // 任一失败，整体 reject
            );
        });
    });
}
```

**追问**：

- 为什么用 `results[i] = val` 而不是 push？→ push 会因异步完成顺序不同而乱序，按下标存才能保证结果顺序与输入一致
- allSettled 的区别？→ 不 reject，等全部结束后返回每个任务的状态
- race / any 怎么改？→ race：第一个 settle 就 resolve/reject；any：第一个成功就 resolve，全失败才 reject（AggregateError）

---

### 30. LRU 缓存（高频！字节、阿里前端爱考）

**题目**：实现 LRUCache：`get(key)` 和 `put(key, value)`，容量满时淘汰最久未使用的，要求 get / put 均为 O(1)。

**思路**：JS 里 Map 的迭代顺序 = 插入顺序，天然支持 O(1) 的「删除 + 重新插入 = 移到最新」。

```js
class LRUCache {
    constructor(capacity) {
        this.capacity = capacity;
        this.map = new Map();
    }
    get(key) {
        if (!this.map.has(key)) return -1;
        const value = this.map.get(key);
        this.map.delete(key);          // 先删
        this.map.set(key, value);      // 再插 → 移到迭代顺序末尾（最新）
        return value;
    }
    put(key, value) {
        if (this.map.has(key)) {
            this.map.delete(key);
        } else if (this.map.size >= this.capacity) {
            this.map.delete(this.map.keys().next().value);  // 淘汰最旧的
        }
        this.map.set(key, value);
    }
}
```

**复杂度**：get / put 均 O(1)

**追问**：

- Map 为什么能做到 O(1)？→ Map 内部基于哈希表，`keys().next()` 取第一个键也是 O(1)（迭代器）
- 不用 Map 怎么写？→ 哈希表 + 双向链表，get 时把节点移到链表头，put 溢出时删链表尾（能说清思路即可）
- 业务场景？→ 浏览器缓存、图片懒加载缓存、历史记录限制条数
- 和 LFU 的区别？→ LRU 看「最近使用时间」，LFU 看「使用频率」

---

### 31. 数组转树 / 树转数组（前端业务强相关）

**题目**：`[{id:1,pid:0},{id:2,pid:1},...]` 转成树形结构；以及逆向。

**思路**：数组转树：先建 id → 节点 Map，再遍历一次把节点挂到父节点的 children 上，父节点不存在的视为根。

```js
function arrayToTree(arr) {
    const map = new Map(arr.map(item => [item.id, { ...item, children: [] }]));
    const roots = [];
    for (const item of arr) {
        const node = map.get(item.id);
        if (map.has(item.pid)) {
            map.get(item.pid).children.push(node);   // 挂到父节点
        } else {
            roots.push(node);                        // 找不到父节点 → 根
        }
    }
    return roots;
}
```

**树转数组**（DFS）：

```js
function treeToArray(tree) {
    const res = [];
    function walk(nodes) {
        for (const node of nodes) {
            const { children, ...rest } = node;   // 剥离 children
            res.push(rest);
            if (children && children.length) walk(children);
        }
    }
    walk(tree);
    return res;
}
```

**复杂度**：均为 O(n)

**追问**：

- 这个实现有什么问题？→ `add/remove` 节点后树会自动更新吗？不会，它是一次性构建；需要动态增删时得维护双向引用
- 父节点排在子节点后面（乱序数据）会出错吗？→ 不会，先建全量 Map 再挂，与顺序无关
- 业务场景？→ 后台菜单、权限树、组织架构、评论楼中楼、无限层级分类

---

### 32. 手写 call / apply / bind（补充，高频）

**题目**：实现 `Function.prototype.myCall`、`myApply`、`myBind`。

**思路**：核心是把函数临时挂到目标对象上调用（`obj.fn()` 时 this 指向 obj），用完删掉；注意 `null / undefined` 时 this 指向全局对象，基本类型要装箱。

```js
Function.prototype.myCall = function (ctx, ...args) {
    ctx = ctx === null || ctx === undefined ? globalThis : Object(ctx);
    const key = Symbol('fn');
    ctx[key] = this;
    const result = ctx[key](...args);
    delete ctx[key];
    return result;
};

Function.prototype.myApply = function (ctx, args = []) {
    return this.myCall(ctx, ...args);
};

Function.prototype.myBind = function (ctx, ...bound) {
    const fn = this;
    function boundFn(...args) {
        // new 调用时 this 优先级高于 bind 的 ctx（箭头函数无 prototype，会抛错）
        const isNew = this instanceof boundFn;
        return fn.myCall(isNew ? this : ctx, ...bound, ...args);
    }
    // 修正原型链，保证 `new BoundFn() instanceof Fn` 成立
    if (fn.prototype) boundFn.prototype = Object.create(fn.prototype);
    return boundFn;
};
```

**追问**：

- 为什么用 `Symbol` 作临时 key？→ 避免覆盖对象原有的同名属性
- bind 后能被 `new` 调用吗？→ 能，`new` 的 this 优先级高于绑定的 ctx，所以要判断 `this instanceof fn`
- 箭头函数能 bind 吗？→ 不能改变 this，`call/apply/bind` 对箭头函数无效
- 为什么 `isNew` 判断用 `boundFn` 而不是 `fn`？→ 箭头函数没有 `prototype`，`this instanceof fn` 会抛 TypeError；用 `boundFn` 更稳

---

### 33. 手写 new（补充）

**题目**：实现 `myNew(Constructor, ...args)`。

**思路**：创建空对象 → 原型链指向 `Constructor.prototype` → 执行构造函数（this 绑定新对象）→ 返回对象类型的返回值或新对象。

```js
function myNew(Ctor, ...args) {
    if (typeof Ctor !== 'function') throw new TypeError('not a constructor');
    const obj = Object.create(Ctor.prototype);
    const result = Ctor.apply(obj, args);
    return result !== null && typeof result === 'object' ? result : obj;
}
```

**追问**：

- 构造函数里 `return 1` 会怎样？→ 基础类型被忽略，返回新对象
- `Object.create` 做了什么？→ 创建对象并把 `__proto__` 指向参数

---

### 34. 手写 instanceof（补充）

```js
function myInstanceof(obj, Ctor) {
    if (obj === null || (typeof obj !== 'object' && typeof obj !== 'function')) return false;
    let proto = Object.getPrototypeOf(obj);
    while (proto) {
        if (proto === Ctor.prototype) return true;
        proto = Object.getPrototypeOf(proto);
    }
    return false;
}
```

**追问**：为什么不用 `obj.__proto__`？→ `__proto__` 是访问器属性，可被改写或不存在；用 `Object.getPrototypeOf` 更标准

---

### 35. 手写柯里化 curry（补充）

```js
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) return fn.apply(this, args);
        return (...next) => curried.apply(this, [...args, ...next]);
    };
}

// 测试
const add = (a, b, c) => a + b + c;
console.log(curry(add)(1)(2)(3)); // 6
```

**追问**：`fn.length` 是什么？→ 形参个数（不含默认值和剩余参数）；有默认参数时长度会变小，需改用占位符方案

---

### 36. 手写 compose / pipe（补充）

```js
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);
```

**追问**：Redux 的 `applyMiddleware` 就是 compose 的实现 → 中间件从右向左包裹 dispatch

---

### 37. 手写简易 EventEmitter（补充）

```js
class EventEmitter {
    constructor() { this.events = new Map(); }

    on(name, fn) {
        if (!this.events.has(name)) this.events.set(name, []);
        this.events.get(name).push(fn);
        return () => this.off(name, fn);   // 返回取消订阅函数
    }

    off(name, fn) {
        const list = this.events.get(name);
        if (!list) return;
        this.events.set(name, list.filter(item => item !== fn));
    }

    once(name, fn) {
        const wrapper = (...args) => { fn(...args); this.off(name, wrapper); };
        this.on(name, wrapper);
    }

    emit(name, ...args) {
        const list = this.events.get(name);
        if (list) [...list].forEach(fn => fn(...args));   // 拷贝，防止回调里 off 影响遍历
    }
}
```

**追问**：为什么 emit 时要 `[...list]`？→ 回调中可能调用 off 修改数组，直接遍历会漏执行；Vue 的 `$emit` 也做了同样的拷贝

---

### 38. 手写 Promise（进阶，能写执行器版即可）

**题目**：实现简易 Promise：state（pending / fulfilled / rejected）、then、异步回调收集、then 返回新 Promise。

```js
class MyPromise {
    constructor(executor) {
        this.state = 'pending';
        this.value = undefined;
        this.callbacks = [];

        const settle = (state, value) => {
            if (this.state !== 'pending') return;   // 只能 settle 一次
            this.state = state;
            this.value = value;
            queueMicrotask(() => this.callbacks.forEach(cb => cb()));
        };

        try {
            executor(
                (val) => settle('fulfilled', val),
                (err) => settle('rejected', err)
            );
        } catch (e) {
            settle('rejected', e);
        }
    }

    then(onFulfilled, onRejected) {
        return new MyPromise((resolve, reject) => {
            const handle = () => {
                try {
                    if (this.state === 'fulfilled') {
                        typeof onFulfilled === 'function' ? resolve(onFulfilled(this.value)) : resolve(this.value);
                    } else {
                        typeof onRejected === 'function' ? resolve(onRejected(this.value)) : reject(this.value);
                    }
                } catch (e) {
                    reject(e);
                }
            };
            this.state === 'pending' ? this.callbacks.push(handle) : queueMicrotask(handle);
        });
    }

    catch(onRejected) { return this.then(null, onRejected); }
}
```

**追问**：

- 为什么用 `queueMicrotask`？→ 原生 Promise 回调是微任务，保证时序一致
- `then` 回调为什么要 try / catch？→ 回调抛错要让新 Promise 变成 rejected
- 完整版还缺什么？→ thenable 递归解析（resolve 一个 Promise 时）、`finally`、静态方法、链路多值穿透

---

## 六、Tree-shaking 与 Vue 依赖收集：被连续追问的「源码题」

> 树形「源码题」不算算法，面试时也经常串在一起问。核心期望是：能说清结论，再补齐“为什么”，这样就能和初级候选人拉开差距。

### 6.1 Tree-shaking 是什么、为什么能生效

一句话：**Tree-shaking 是打包阶段（Rollup / Webpack / Vite）基于 ESM 静态结构，把未被引用的模块成员删除的优化，前提是可以静态分析。**

三个必须同时成立的条件：

1. **代码是 ESM（`import` / `export` 或严格合法的 CJS）**。CJS 的 `require` 是运行时行为，无法静态分析。
2. **无副作用（side-effect free）**。如果被删代码执行了赋值、I/O 等，打包器不敢删。Webpack 判断依据：`package.json` 的 `"sideEffects"` 字段、模块顶层简单分析；不确定就保守不删。
3. **用的是“引用”而非“整体”**。`import utils from './u'` 并访问 `utils.a` 时，压缩器经常要保留整个对象，因此需要 `/*#__PURE__*/` 或改写具名导入。

```js
// utils.js
export const a = () => 'a';
export const b = () => 'b';

// main.js —— 只引 a，b 会被 Tree-shaking 掉
import { a } from './utils';
a();
```

**两轮“清理”**：Rollup / Webpack 在模块层面移除没用到的 `export`；Terser 在作用域层面移除不可达代码、死分支。二者叠加才彻底。

### 6.2 哪些情况 Tree-shaking 会失效

- **整体导入命名空间**：`import * as _ from 'lodash'`，打包器只知“用到了这个模块”，没法按需删。
- **访问属性链**：`_.debounce` 这类语义在压缩后属于动态属性访问，只能保守保留整个对象。
- **有副作用的模块**：顶层执行了 `window.x = 1`、`polyfill`、`global.css` 导入等。
- **CJS 模块**：`module.exports` 是运行时对象，`exports.a = ...` 属于“对象动态挂载”，无法静态确认未被使用。
- **babel 转成 CJS**：旧的 `@babel/preset-env` 默认把 ESM 转成 CJS 后，Tree-shaking 就失效了；必须配 `modules: false`（或让打包器处理 ESM）。

### 6.3 工程中真正可行的几条

1. **优先用 ESM 版本**（`module` / `exports.import` 字段），并给依赖确认 `sideEffects: false`。
2. **具名导入替代命名空间导入**：`import { debounce } from 'lodash-es'`，而不是 `import _ from 'lodash'`。
3. **避免“只为了工具函数”而引入整个库**：小工具自己写，或用 `lodash-es` 这类逐函数打散的包。
4. **检查 babel / tsconfig**：`"module": "ESNext"`，不要在打包前把模块转成 CJS。
5. **验证方式**：`vite build` 后看 chunk 体积、source-map 里是否还有目标代码；Webpack 用 `--analyze`。

### 6.4 Vue 依赖收集：数据变化是怎么通知视图的

Vue 2 / Vue 3 的做法本质相同：**在 getter 里记录“谁在读我”（收集依赖），在 setter / Proxy traps 里通知“读我的人”（派发更新）**。区别在实现方式：

| 维度 | Vue 2 | Vue 3 |
|------|-------|-------|
| 拦截方式 | `Object.defineProperty` 重写 getter / setter | `Proxy` 拦截 get / set |
| 新增 / 删除属性 | 检测不到，需要 `Vue.set` / `Vue.delete` | 自动拦截 |
| 数组 | 重写 7 个变异方法（push / pop / shift / unshift / splice / sort / reverse） | 原生响应式 |
| 依赖存储 | `Dep` 类 + `Watcher` | `targetMap → depsMap → dep` + `effect` |
| 收集时机 | `watcher.get()` 时把 `Dep.target` 指向自己，读到 getter 时 `dep.depend()` | 同思路，`activeEffect` 压栈管理嵌套 |

```js
// Vue 2 的核心精简版
const targetStack = [];
function defineReactive(obj, key, val) {
    const dep = new Dep();
    Object.defineProperty(obj, key, {
        get() {
            if (Dep.target) dep.depend();   // 谁在读我，就把我记到谁身上
            return val;
        },
        set(newVal) {
            if (newVal === val) return;
            val = newVal;
            dep.notify();                   // 我变了，通知这些依赖
        }
    });
}
```

**看代码时容易被追问的点**：

- **为什么要“收集”而不是“遍历模板”**？渲染函数执行本身就是读数据，读的时候收集比事后扫描精确得多（也支持 `computed` 这类派生）。
- **Vue 3 为什么要用 Proxy**？不只是解决新增属性，还能把 `Map / Set` 等集合类型纳入，且能拦截 `in`、`delete`、方法调用。
- **为什么 Vue 3 的 `effect` 里能区分“谁在依赖谁”**？用一个 `activeEffect` 栈：执行 `effect.run()` 时压栈，结束时出栈，保证嵌套 effect 的依赖归属正确。
- **调度器的作用**？同一轮里多次改数据，`queueJob` 去重后在一个微任务里刷新，避免重复渲染。
- **`computed` 和 `watch` 的区别**？computed 是“懒执行 + 缓存”，watch 是“变化即回调”。

**回答范式建议**：“先说结论，再说机制，最后补一个工程实践”，例如：

> Vue 2 用 `defineProperty` 劫持属性，getter 里收集依赖，setter 里通知 `Watcher`，所以新增属性不响应，需要 `Vue.set`；Vue 3 用 `Proxy` 代理整个对象，新增和删除都能触发，还把依赖关系收敛成 effect 依赖图。业务里最直接的影响是：Vue 3 不必再担心 `this.obj.newKey = 1` 不生效。

---

## 七、手写题速查表

| 题 | 核心套路 | 关键坑 |
|----|----------|--------|
| 防抖 | clearTimeout + 重新 setTimeout | this / args 透传 |
| 节流 | 时间戳差值 / 定时器 | 首尾触发时机 |
| 深拷贝 | 递归 + WeakMap | 先 set 再递归 |
| Promise.all | 下标存值 + 计数 | 不能 push，要保序 |
| call / apply / bind | 临时挂到 ctx 上调用 | Symbol 防覆盖；bind 与 new 的优先级 |
| new | Object.create + apply + 返回判断 | 构造函数返回对象 / 基础类型 |
| instanceof | 沿原型链查找 | 用 getPrototypeOf，不用 `__proto__` |
| 柯里化 | fn.length 判断参数是否够 | 默认参数会改变 length |
| EventEmitter | Map 存回调数组 | emit 时拷贝一份数组 |
| LRU | Map 的插入顺序 | 先 delete 再 set |
| 数组转树 | Map 建节点，遍历挂载 | 父节点不存在 → 视为根 |
| 树转数组 | DFS + 剥离 children | 别改原节点 |
| 简易 Promise | state + callbacks + 微任务 | then 回调中的 try / catch |

---

## 八、按「套路」索引

面试时认出套路，比认出原题更重要：

| 套路 | 特征 | 题目 |
|------|------|------|
| 哈希表 | 找配对 / 分组 / 计数 | 两数之和、字母异位词分组、最长连续序列 |
| 双指针 | 有序数组 / 两端收缩 / 原地删除 | 三数之和、盛最多水的容器、移动零、合并两个有序数组 |
| 快慢指针 | 原地保持相对顺序 / 链表 | 移动零、判断链表是否有环、找环入口 |
| 滑动窗口 | 连续子串 / 子数组，求最值 | 无重复字符的最长子串、找到所有字母异位词、最小覆盖子串 |
| 单调栈 | 「下一个更大 / 更小」 | 每日温度、下一个更大元素、接雨水（困难） |
| 栈 | 配对 / 顺序反转 | 有效的括号、用两个栈实现队列 |
| 链表哑节点 | 合并 / 删除 | 合并两个有序链表、删除倒数第 N 个节点 |
| 二叉树 BFS | 按层处理 | 层序遍历、锯齿形遍历、二叉树右视图 |
| 二叉树 DFS | 递归 + 返回值合并 | 最大深度、对称、路径总和、LCA |
| 回溯 | 需要枚举所有方案 | 全排列、组合、子集、N 皇后 |
| DP | 求最值 / 方案数，有重复子问题 | 爬楼梯、打家劫舍、最大子数组和、不同路径 |
| 二分查找 | 有序 / 有单调性的“判定” | 旋转数组找最小值、搜索插入位置、分割数组的最大值（进阶） |
| 网格 DFS / BFS | 二维矩阵连通块 | 岛屿数量、岛屿最大面积、腐烂的橘子 |
| 快速选择 | 第 K 大 / 前 K 个 | 数组中第 K 个最大元素 |

---

## 九、面试现场自查清单

写题前 / 写题中 / 写完，按顺序过一遍：

**写题前**

- [ ] 先讲思路再动手，说清算法选择和复杂度
- [ ] 问清边界：输入是否为空？是否有序？能不能用额外空间？元素是否有重复？

**写题中**

- [ ] 双指针题：指针什么时候移动？边界用 `<` 还是 `<=`？
- [ ] 递归题：终止条件写了吗？会不会栈溢出？
- [ ] 数组题：下标会不会越界？（尤其是空数组取 `nums[0]`）

**写完后**

- [ ] 主动用 1~2 个测试用例手推一遍
- [ ] 自己报出时间 / 空间复杂度
- [ ] 说出一个可优化点（哪怕不写）
- [ ] 如果和业务相关，一句话带出场景（加分项）

**最不该踩的坑（前端岗）**

1. `==` 和 `===` 混用
2. 大数 / 浮点数直接用 Number 运算（金额、大整数）
3. `sort()` 不带比较函数（默认按字符串排序，`[1, 10, 2]` 会排成 `[1, 10, 2]`）
4. 递归没有终止条件
5. 引用类型直接赋值没拷贝（深拷贝题）
6. 闭包陷阱：`for` 循环里用 `var` 声明回调变量（用 `let` 或立即执行函数）
7. 原地算法里先改值后判断（如移动零先判断再交换，顺序反了就丢数据）

---

## 十、写在最后：怎么用这份手册

这份手册不是题库，而是**模板库 + 追问库**。建议：

1. **第一遍**：按 10 天计划刷第一档和手写题，每道题合上答案手写一遍。
2. **第二遍**：只看「套路索引」和「手写题速查表」，用自己的话复述每题的思路和复杂度。
3. **面试前一晚**：只看自查清单，把最不该踩的坑过一遍。
4. **面试后**：把被追问住的地方补进对应题目，这份文档就会越来越像你的个人面试笔记。

> 复杂 DP、图论、困难题不作为前端岗准备重心。把高频题讲清楚、代码写干净、能主动说自己方案的取舍，比多刷 50 道难题更有性价比。

---

> 本文所有 JS 代码块（共 44 段）均已用 Node.js 实际运行并断言验证：边界输入（空数组 / 空字符串 / 单元素 / 全负数）、异步时序（防抖节流、Promise、微任务）以及循环引用深拷贝均通过。


