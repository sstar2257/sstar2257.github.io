---
title: word_break
date: 2019-04-13 11:10:53
updated: 2019-04-13 11:10:53
tags: [algorithm, leetcode]
---

### 139. Word Break
给定一个非空字符串S，以及一个字典集D，输出S是否能由D中的字符串拼接而成。  
![q139](/img/pic139.png)
一个比较好的算法是用动态规划去解决。新建一个S.size()长度的bool数组，对于数组的第N位，表示该位置能否由前面某个位置M加上一个N-M长度字符串拼接而成。  
```
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        int len_s = s.size();
        bool *dp = new bool[len_s+1]();
        int len_w = wordDict.size();
        dp[0] = true;
        for(int i=0;i<len_s;++i){
            for(int j=0;j<len_w;++j){
                if(dp[i]){
                    string tmp = s.substr(i, wordDict[j].size());
                    if(tmp == wordDict[j]){
                        dp[i + tmp.size()] = true;
                    }
                }
            }
        }
        return dp[len_s];
    }
};
```
