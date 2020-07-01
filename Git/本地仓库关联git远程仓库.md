# 本地仓库关联git远程仓库

## 主要步骤

* 创建github 私有仓库；注意在创建仓库的时候不要初始化 readme文件；
* 把现有仓库通过命令行上传；
* git remote add origin git@github.com:name/repo.git 添加远程索引；
* git push -u origin master 把本地master 推送到远程；
* 查看现有仓库的所有的远程代码库地址：git remote -v 
* 如果当前本地仓库已经设置了 origin 的地址；使用下列命令进行删除：git remote remove origin 并再次使用git remote -v 确定；
* 设置origin 索引地址：git remote add origin git@github.com:name/repo.git；
* 将本地 master 分支，推送到远程仓库的 master 分支：git push -u origin master；
* 推送其他分支到远程仓库；git push --set-upstream origin 分支名称；在远程建立分支并推送本地分支；
* 推送git子仓库到远程仓库；如果本地仓库体积过大，可以选择不推送；直接使用打包支持；
* 项目转移完毕；
* git push: git push origin 本地分支名称:远程分支名称 ， 使用一个，默认本地分支和远程分支相同

## 可能出现的错误:

```git
error: src refspec master does not match any.
error: failed to push some refs to
```

原因：
1. 本地git仓库目录下为空
2. 本地仓库add后未commit
3. git init错误
4. 没有配置邮箱和用户名

配置邮箱用户名：
```
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

![error2]($resource/error2.png)

原因：远程仓库与本地内容不一致(建议建一个空远程仓库)。

解决：
1. 尝试先pull远程内容
`git pull origin master`
再push
`git push origin master`

2. 强制push(会用本地仓库内容完全替换远程仓库)
```
git push -f origin master
```

## 配置SSH KEY

[参考](https://www.cnblogs.com/yihen-dian/p/8760756.html)
