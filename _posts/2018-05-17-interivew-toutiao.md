---
layout: post
title:  今日头条算法面试题
# gh-repo: lttzzlll/leetcode-java
# gh-badge: [star, fork, follow]
tags: [Interview, Algorithm]
---


### 随选几道今日头条算法面试题

1. 打印一棵二叉树从左侧看到的所有节点

```
                     1
                2         3
            4      5   6      7 
                                  8
```

比如，上面这颗二叉树从左侧看到的所有节点为 1, 2, 4, 8


思路: 当作树的层次遍历，从上到下，从左到右，只遍历每层的第一个元素。

代码如下:

```Python
import unittest


class Node:
    def __init__(self, val):
        self.val = val
        self.left = None
        self.right = None

    def __str__(self):
        return self.val


class NodeWrapper:
    def __init__(self, node, level):
        self.node = node
        self.level = level

    def __str__(self):
        return '{}-{}'.format(self.node, self.level)


def left_visit(root):
    res = []
    que = []
    lastLevel = 0
    que.append(NodeWrapper(root, lastLevel + 1))
    while que:
        nw = que.pop(0)
        if nw.level > lastLevel:
            res.append(nw.node.val)
            lastLevel = nw.level

        if nw.node.left:
            que.append(NodeWrapper(nw.node.left, lastLevel + 1))
        if nw.node.right:
            que.append(NodeWrapper(nw.node.right, lastLevel + 1))

    return res


class LeftVisitTest(unittest.TestCase):
    @staticmethod
    def create_tree():
        root = Node(1)
        root.left = Node(2)
        root.right = Node(3)
        root.left.left = Node(4)
        root.left.right = Node(5)
        root.right.left = Node(6)
        root.right.right = Node(7)
        root.right.right.right = Node(8)
        return root

    def setUp(self):
        self.root = LeftVisitTest.create_tree()

    def test_left_vist(self):
        res = [1, 2, 4, 8]
        left_visit_node = left_visit(self.root)
        self.assertListEqual(res, left_visit_node)
        print(left_visit_node)

    def tearDown(self):
        self.root = None


if __name__ == '__main__':
    unittest.main()

```

2. 给定一个数组，其中超过一半以上都是同一个数字，找出这个数字。

思路: 设置一个计数器，记录当前元素作为该元素的次数。最后剩下的就是要找的元素。

优化: 如果当前元素的数量超过数组中剩余元素的数量，则提前可以判断出该元素就是要找的元素。


代码如下:

```Python
import random
import unittest


def most_num(nums):
    cur = nums[0]
    cnt = 1
    for i in range(1, len(nums)):
        if cnt == 0:
            cur = nums[i]
            cnt = 1
        else:
            if nums[i] == cur:
                cnt += 1
            else:
                cnt -= 1
        # print(cur, nums[i], cnt)

    return cur

class TestMostNumber(unittest.TestCase):
    def setUp(self):
        self.nums = random.sample(range(0, 100), 10)
        self.most_num = random.randint(100, 1000)
        self.nums += [self.most_num] * 11
        random.shuffle(self.nums)

    def test_most_num(self):
        # print(self.nums)
        res = most_num(self.nums)
        self.assertEqual(res, self.most_num)

    def tearDown(self):
        self.nums = None


if __name__ == '__main__':
    unittest.main()

```


3. 给定一个正整数，找出第一个大于该正整数的数位不重复的数字。比如: n = 889911, next_n = 890101.

思路: 感觉比较简单，按照定义来就行。


代码如下:

```Python
import unittest
import random


def is_repeat(n):
    n_str = str(n)
    for i in range(len(n_str) - 1):
        if n_str[i] == n_str[i + 1]:
            return False
    return True


def firstnum(n):
    while True:
        n += 1
        if is_repeat(n):
            return n
    return None


class FirstNumTest(unittest.TestCase):
    def test_first_num(self):
        n = 889911
        next_n = 890101
        res = firstnum(n)
        self.assertEqual(res, next_n)


if __name__ == '__main__':
    unittest.main()

```

4. 给定一个二维数组，找出该二维数组中的最长递增路径。比如:

```
nums = [
    [9, 9, 4],
    [6, 7, 8],
    [2, 1, 1]
]

longest_path = [1, 2, 6, 7, 9]
```

思路: 搜索/动态规划。一个简单的动态规划/搜索,leetcode中级题。

```
longest_path = max(dp[i][j] for j in range(n) for i in range(m))

dp[i][j] = max(
    dp[i-1][j] if i - 1 >= 0 and !visit[i-1][j] and nums[i-1][j] < nums[i][j], 
    dp[i][j-1] if j - 1 >= 0 and !visit[i][j-1] and nums[i][j-1] < nums[i][j], 
    dp[i+1][j] if i + 1 < m and !visit[i+1][j] and nums[i+1][j] < nums[i][j], 
    dp[i][j+1] if j + 1 < n and !visit[i][j+1] and nums[i][j+1] < nums[i][j]
) 

dp[i][j] represents longest_path of endpoint (i, j)

m = len(nums)
n = len(nums[0])

visit[m][n] = False if nums[i][j] not visited else True 
```


代码如下:

```Python
import unittest


def dp(nums, m, n, i, j, visit):
    ll, uu, rr, dd = [], [], [], []
    if i - 1 >= 0 and not visit[i - 1][j] and nums[i - 1][j] < nums[i][j]:
        visit[i][j] = True
        uu = dp(nums, m, n, i - 1, j, visit)
    if j - 1 >= 0 and not visit[i][j - 1] and nums[i][j - 1] < nums[i][j]:
        visit[i][j] = True
        ll = dp(nums, m, n, i, j - 1, visit)
    if i + 1 < m and not visit[i + 1][j] and nums[i + 1][j] < nums[i][j]:
        visit[i][j] = True
        dd = dp(nums, m, n, i + 1, j, visit)
    if j + 1 < n and not visit[i][j + 1] and nums[i][j + 1] < nums[i][j]:
        visit[i][j] = True
        rr = dp(nums, m, n, i, j + 1, visit)

    res = []
    if len(ll) > len(res):
        res = ll
    if len(rr) > len(res):
        res = rr
    if len(uu) > len(res):
        res = uu
    if len(dd) > len(res):
        res = dd
    return res + [nums[i][j]]


def longest(matrix):
    m = len(matrix)
    n = len(matrix[0])
    record = [[0 for i in range(n)] for j in range(m)]

    for i in range(m):
        for j in range(n):
            visit = [[False for i in range(n)] for j in range(m)]
            record[i][j] = dp(matrix, m, n, i, j, visit)

    res = []
    for row in record:
        for cell in row:
            if len(cell) > len(res):
                res = cell
    return res


class TestLongest(unittest.TestCase):

    def test_longest(self):
        nums = [[9, 9, 4],
                [6, 7, 8],
                [2, 1, 1]]
        longest_path = [1, 2, 6, 7, 9]
        res_path = longest(nums)
        self.assertEqual(len(longest_path), len(res_path))
        print(longest_path, res_path)


if __name__ == '__main__':
    unittest.main()

```