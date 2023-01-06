---
title: Trie树
date: 2021-03-21 13:05:30
tags: 
- 哈希
- 树
categories: 
- 算法
- 数据结构
---
<!--
 * @Author: JNJYan
 * @LastEditors: JNJYan
 * @Email: jjy20140825@gmail.com
 * @Date: 2021-03-21 11:08:03
 * @LastEditTime: 2021-03-21 13:28:38
 * @Description: Modify here please
 * @FilePath: /blog/source/_posts/2021/03/Trie树.md
-->
# Trie树的定义

Trie树，又被称为字典树、单词查找树，是一种哈希树变种。常用于统计、排序和保存大量的字符串，如搜索引擎系统中的文本词频统计、拼写的智能补全、字符串排序、最长公共前缀等。优点在于，利用字符串的公共前缀减少查询时间，查找效率比哈希树高。



# Trie树的实现
## Trie结构
对于Trie树的每一个节点，应当有一个标志位表示从根节点到当前节点的字符串是否存在，每个节点应当有26个儿子节点（与字符数量相同）。
```c++
class TrieNode{
private:
    bool isEnd;
    TrieNode* next[26];
public:
    void TrieNode();
    void insert(string str);
    bool search(string str);
    bool startWith(string prefix);
}
```

## Init
初始化一个Trie树，树的根节点应为空。

```c++
void TrieNode(){
    isEnd = false;
    memset(next, 0, sizeof(next));
}
```

## Insert
```c++
void insert(string str){
    Trie* node = this;
    for(char ch: str){
        if(node->next[ch-'a'] == nullptr)
            node->next[ch-'a'] = new Trie();
        node = node->next[ch-'a'];
    }
    node->isEnd = true;
}
```

## Search
```c++
bool search(string str){
    Trie* node = this;
    for(char ch: str){
        node = node->next[ch-'a'];
        if(node == nullptr)
            return false;
    }
    return node->isEnd;
}
```

## StartWith
```c++
bool StartWith(string str){
    Trie* node = this;
    for(char ch: str){
        node = node->next[ch-'a'];
        if(node == nullptr)
            return false;
    }
    return true;
}
```


# 哈希树

哈希树（HashTree）是一种持久性数据结构，用于实现集合和映射。通过质数分辨法建立哈希树，从第二层开始，每层的节点数为连续质数(2,3,5,7...)，从左至右分别为对于该质数的余数。

