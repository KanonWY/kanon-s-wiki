# git_cred

git 凭证管理

## 1、https 如何缓存账号密码在系统中
```Bash

git config --global credential.helper store

# 清理 http 凭证
git credential reject
```

## 2、修改子库为 https
```Bash
git config --file=.gitmodules submodule.<submodule-path>.url https://github.com/username/repository.git
git submodule sync
git submodule update --init --recursive

```