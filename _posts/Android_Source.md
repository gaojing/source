title: Android Source 
categories: Android
tags: [Android]
---

 通过国内镜像获取Android源码

###  下载repo

    curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o ～/bin/repo
    chmod +x repo

 将下面内容加进.bashrc中

    export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
    
### 使用每月更新的初始化包

下载 https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar

repo sync


### 参考
[repo](https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/)
