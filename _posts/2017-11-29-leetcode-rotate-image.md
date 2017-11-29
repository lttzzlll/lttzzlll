---
layout: post
title: Rotate Image
gh-repo: lttzzlll/leetcode-java
# gh-badge: [star, fork, follow]
tags: [leetcode, algorithm, matrix]
---

### [Rotate Image](https://leetcode.com/problems/rotate-image/description/)

图片旋转.

先以横轴为对称线做一次上下变换,然后再以斜对角线为对称线做一次变换.

```
0.

1 2 3
4 5 6
7 8 9

1.

7 8 9
4 5 6
1 2 3

2.

7 4 1
8 5 2
9 6 3
```

所以, 代码就很容易得出:
{% highlight java linenos %}
public void rotate(int[][] matrix) {
    int m = matrix.length;
    for (int row = 0; row < m / 2; row++) {
        for (int col = 0; col < m; col++) {
            int temp = matrix[row][col];
            matrix[row][col] = matrix[m - 1 - row][col];
            matrix[m - 1 - row][col] = temp;
        }
    }
    for (int row = 1; row < m; row++) {
        for (int col = 0; col < row; col++) {
            int temp = matrix[row][col];
            matrix[row][col] = matrix[col][row];
            matrix[col][row] = temp;
        }
    }
}
{% endhighlight %}


但是,为什么要这样做变换?

接下来, 线性代数, 矩阵运算.