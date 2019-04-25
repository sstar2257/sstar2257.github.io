---
title: max_rectangle
date: 2019-04-12 21:42:51
updated: 2019-04-12 21:42:51
tags: [algorithm, leetcode]
---

## 刷leetcode遇到的求二维矩阵最大面积的题目，记录下
### [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) 给定一个一维数组代表柱状图，求最大的矩形面积。  
![q84](/img/pic84.png)
<!-- more -->
```
class Solution {
public:
    int max(int a, int b){
        return a > b ? a : b;
    }

    int largestRectangleArea(vector<int>& heights) {
        int len = heights.size();
        int i = 0;
        int s = 0;
        int tmp;
        stack<int> sta;

        while(i < len){
            while(!sta.empty() && heights[i] < heights[sta.top()]){
                tmp = sta.top();
                sta.pop();
                s = max(s, heights[tmp] * (i - (sta.empty() ? 0: sta.top() + 1)));
            }
            sta.push(i++);
        }

        while(!sta.empty()){
            tmp = sta.top();
            sta.pop();
            s = max(s, heights[tmp] * (len - (sta.empty() ? 0: sta.top() + 1)));
        }

        return s;
    }
};
```
### [Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/)给定一个二维矩阵，求最大的矩形面积。  
可以参考上面的做法，按行遍历，然后对每一行进行上面的算法。  
leetcode讨论区还有一种动态规划做法，没看明白之后再说。  
![q85](/img/pic85.png)

### [Maximal Square](https://leetcode.com/problems/maximal-square/)给定一个二位矩阵，求最大的正方形。  
这道题比上面简单不少，可以使用动态规划求解。  
![q221](/img/pic221.png)
```
class Solution {
public:
    int min(int a, int b, int c){
        a = a < b ? a : b;
        return a < c ? a : c;
    }
    int maximalSquare(vector<vector<char>>& matrix) {
        int row = matrix.size();
        if(row == 0)
            return 0;
        int col = matrix[0].size();
        int **table = new int*[row+1]();
        for(int i=0;i<row+1;i++){
            table[i] = new int[col+1]();
        }
        int len = 0;
        for(int i=1;i<=row;++i){
            for(int j=1;j<=col;++j){
                if(matrix[i-1][j-1] == '1'){
                    table[i][j] = min(table[i-1][j], table[i-1][j-1], table[i][j-1]) + 1;
                    len = max(len, table[i][j]);
                }
            }
        }
        return len*len;
    }
};
```

