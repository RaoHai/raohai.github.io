title: LeetCode OJ:Restore IP Addresses 
date: 2014-08-28 16:01:25
tags: 技术流
---
虽然水平不高，但对算法总有一种莫名的喜欢，一看到算法相关的博客或文章，不管看得懂看不懂，都会有意多看几眼。却又不是那种无怨无悔的喜欢，比如说大学的时候被老师布置了刷ACM题的任务的时候，就会感到排斥和厌倦。

最近倒喜欢在LeetCode上刷题了。之前玩过类似的Codewars。相比之下，Codewars从kyu到dan的“打怪升级”模式让人特别有动力。但就有一点不好：计时。好像被人追着赶着一样，不自在。还是LeetCode好。闲来没事选一题，知道怎么写就写，不知道就想思路。实在想不出就放着，搞不好上个厕所就想起来了。

这些天遇到的第一个挑战，就是这题 [Restore IP Addresses](https://oj.leetcode.com/problems/restore-ip-addresses/) 了。


>Given a string containing only digits, 
>restore it by returning all possible valid IP address combinations.

>For example:
>Given "25525511135",

>return ["255.255.11.135", "255.255.111.35"]. (Order does not matter)

还算是比较简单的Enumerate Combinations，列出这串数字所有合法的IP组合。

所以应该如何枚举呢？题目的用例其实不好，因为对我而言，255在IP段上是一个定式。改用这个例子比较好分析："123001"这个数字串能有多少种有效的IP地址组合。

列出"123001"这个数字串可能的IP地址组合，可以这么一步步来：因为IP地址一共有4位，每段最多3个字符：

1. IP地址第一位有取"1","12","123"三种取法
2. 第一位取"1"时，第二位有"2","23",""230"三种取法
2.1 第二位取"2"时，第三位有"3","30"两种取法
2.2 第二位取"23"时，……
3. 第一位取"12"时，第二位有"3","30"两种取法
……

当取完数字串中的全部数字，并且同时填满IP地址的所有位时，得到一个合法的IP地址组合。

所以这个问题的解空间可以用一棵树来表示。

在确定了解空间的组织结构后，可以使用回溯法(Backtracking)来解决这个问题。回溯法从根节点出发，以深度优先搜索方式搜索整个解空间。回溯法以这种工作方式递归地在解空间中搜索，直到找到所要求的解或解空间所有解都被遍历过为止。

事实上，上面画出的这棵树，已经使用了剪枝函数剪去了无效的搜索：

**约束函数：**每一次分支中，不符合有效IP约束的数字组合直接被剪掉，比如第一位取"12"之后第二位的"300"，前两位取"12""3"之后第三位的"00"，都是属于不符合有效IP约束的分支。

**界限函数：** 每个有效的IP仅有四位。如上图中的"1.2.3.\*", 和"1.2.30.\*"，之后，给定的数字串还没用完，但IP位已经填满，可以断定该结点为跟的子树中不含解，所以将该子树剪去。

所以这一题使用回溯法，用递归函数实现。因为LeetCode要求把解答写成Solution类的成员函数，所以比起直接写递归函数，稍微麻烦了一点：
```cpp
#include <vector>
#include <string>
#include <iostream>

using namespace std;

//剪枝函数
bool isIp(string s) {
  int value = 0;
  if (s.size() > 1 && s[0] == '0')
  {
    return false;
  }
  for (size_t i = 0; i < s.size(); i++)
  {
    value = value * 10 + s[i] - 48;
  }
  return value >= 0 && value <= 255;
}

//使用vect来保存每一步，其实有点麻烦。
string traceVect(vector<string> vect) {
  string concatString = "";
  for (size_t i = 0; i < vect.size(); i++)
  {
    concatString += vect[i];
    if (i != vect.size() -1)
    {
      concatString += ".";
    }
  }
  return concatString;
}

class Solution {
public:
  void backtrace(vector<string> ips, string s, int pos){
    if (pos > s.size())
    {
      return;
    }
    if (ips.size() >= 4)
    {
      if (pos == s.size())
      {
        string result = traceVect(ips);
        this->_ips.push_back(result);
      }
      return;
    }
    for (size_t i = 1; i <= 3; i++)
    {
      string _s = s.substr(pos, i);
      if (isIp(_s))
      {
        vector<string> _ips = ips;
        _ips.push_back(_s);
        backtrace(_ips, s, pos + i);
      }
    }
  }

  vector<string> restoreIpAddresses(string s) {
    this->_ips.clear();
    vector<string> ips;
    backtrace(ips, s, 0);

    return this->_ips;
  }

private:
  vector<string> _ips;
};
```

剪枝函数额外说明一下，主要是剔除多位的开头为0的，比如000, 01，002这样的IP，勉强算是一个Corner Case吧。

