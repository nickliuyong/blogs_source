---
title: git tag
date: 2017-01-24 16:22:56
tags:
    - git 
---

### git tag的使用

   一般来说，当我们发布一个版本时或者一个大版本时，需要打相应的tag来记录对应的版本号，以此来方便我们回归整个
   commit log，对于git 来说，当我们发布以release版本后，可以在master分支上通过 git tag来记录相应标签

``` bash
   建立标签： git tag -a 0.0.1 -m "test git tag 0.0.1"
   推送到远程： git push origin --tags
   根据tag拉取代码并新建分支： git checkout -b 0.0.1-branch  0.0.1
```

### 推荐:
    
   一般来说，我们的工程都是采用maven编译的，建议每一次tag的建立和maven工程的version进行一一对应，如此可以清晰
   的拉取到对应的源码


### 总结如下

    当一个分支合并到master时，需做如下几件事情：

    1. 打相应version的jar、war包

    2. 打对应version的tag


