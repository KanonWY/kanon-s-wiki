# gitsubmodule

## 删除某一个子模块

```Bash
git submodule deinit -f <path_to_submodule>
git rm -f <path_to_submodule>
rm -rf .git/modules/<path_to_submodule>
```
