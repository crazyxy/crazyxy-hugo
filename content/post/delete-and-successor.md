---
title: "Delete And Successor"
date: 2017-07-14T17:30:59+08:00
draft: false
---

# Problem

Given a sequence a number `$S=\{1...n\}$` and a sequence of operations which contain two types of operation: delete and query. delte(x) delete x from the sequence and query(x) return the successor of the `$x$` in the sequence.

# Solution

Union-Find. When delete(x), union the subtree where `$x$` in and the subtree `$x+1$` in. The operation can be viewed as a fold operation which fold the `$x$` and `$x+1$`. When query(x), return the find(x+1). Suppose (x+1) was delete, find(x+1) actually returns find(x+2) and if `$x+2$` is deleted, find(x+3) will be returned. Until some `$y$` which has been deleted and `$y$` is the successor of `$x$`.

*C++ Implementation*

```c++
const int N = 10010;

int id[N];

void init(){
    for(int i = 0; i < N; i ++)
        id[i] = i;
}

int Find(int x){
    return id[x] = (x == id[x] ? x : Find(id[x]));
}

void Union(int x, int y){
    int pid = Find(x);
    int qid = Find(y);
    if(pid == qid) return;
    id[pid] = qid;
}

void Delete(int x){
    Union(x, x+1);
}

int Query(int x){
    return Find(x+1);
}
```