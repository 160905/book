# 😃

GitHub Pages 的静态资源支持下面 3 个来源：

+ `master`分支
+ `master`分支的`/docs`目录
+ `gh-pages`分支

执行下面命令，将 `_book` 目录推送到 GitHub 仓库的 `gh-pages` 分支。

``` shell
$ git subtree push --prefix=_book origin gh-pages
```

或者在生成静态网页时，将保存的目录指定为 `./docs`
``` shell
$ gitbook build ./ ./docs
```

然后直接推送到 GitHub 仓库的。

``` shell
git push origin master
```

