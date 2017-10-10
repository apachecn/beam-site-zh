# 翻译流程
翻译流程如下，有问题请随时联系

## 具体步骤
1. 参考 [VSCode Windows 平台入门使用指南](https://github.com/apachecn/scikit-learn-doc-zh/blob/0.19.X/help/vscode-windows-usage.md) fork apachecn github repo: <https://github.com/apachecn/beam-site-zh> 到自己的 github repo.（熟悉的大佬可以略过）
2. clone 你的 github repo 的 asf-site-zh 分支:
```shell
    git clone https://github.com/wangyangting/beam-site-zh -b asf-site-zh
```
> 注意: 我的 repo 是 wangyangting
3. 切换分支到 asf-site-zh
```shell
cd beam-site-zh/
git checkout asf-site-zh
```
4. 翻译自己负责的文档章节.
这里可以参考 [VSCode Windows 平台入门使用指南](https://github.com/apachecn/scikit-learn-doc-zh/blob/0.19.X/help/vscode-windows-usage.md) 使用 VSCode 编辑器来操作，也可以使用其它编辑器的工具。

5. 推送到你自己的 repo 中：
```shell
git push
```
6. 在你的 github 的 repo 页面中，提一个 pull request ，可参考下图:  
![](https://github.com/apachecn/scikit-learn-doc-zh/raw/0.19.X/help/img/git-branch_9.png) 
7. 等待 ApacheCN 审核合并