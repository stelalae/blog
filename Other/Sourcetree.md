# SourceTree使用记录

QQQ：打开/关闭 git执行的完整信息

![eHkVgs](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/09/22/eHkVgs.png)

QQQ：在Mac下git pre-commit钩子无法使用

在项目中`yarn add -D husky`的pre-commit钩子，提交代码时没有触发钩子执行，发现提示

```bash
Can't find npx in PATH: /Applications/Xcode.app/Contents/Developer/usr/libexec/git-core:/Applications/SourceTree.app/Contents/Resources/bin:/usr/bin:/Applications/SourceTree.app/Contents/Resources/git_local/gitflow:/Applications/SourceTree.app/Contents/Resources/git_local/git-lfs:/usr/bin:/bin:/usr/sbin:/sbin
Skipping pre-commit hook
Completed successfully
```

还有可能提示`Can't find npx in PATH`等等，这些问题大致是①通过nvm安装node②自行安装yarn等导致bin路径的变化，然后未找到对应的可执行文件。经分析方案找到`.git/hooks/husky.sh`里有下面代码：

```bash
# Source user ~/.huskyrc
if [ -f ~/.huskyrc ]; then
  debug "source ~/.huskyrc"
  . ~/.huskyrc
fi
```

所以说把解决步骤是：

```
$ which yarn
/Users/xxx/.yarn/bin/yarn

$ which node
/Users/xxx/.nvm/versions/node/v12.18.3/bin/node

$ vim ~/.huskyrc
PATH="/Users/xxx/.nvm/versions/node/v12.18.3/bin:/Users/xxx/.yarn/bin:$PATH"
```

其他问题可参考此思路：首先通过报错定位问题，然后分析报错前的执行步骤，提前解决错误原因即可。另外可参考[Sourcetree在Mac下git pre-commit钩子无法使用node问题解决](https://www.jianshu.com/p/e70d735358eb)。

