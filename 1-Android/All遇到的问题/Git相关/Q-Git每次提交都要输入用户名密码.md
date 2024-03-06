每次push的时候，都会提示输入github的用户名和密码，

上网查了一下原来是https协议每次都需要进行用户验证，当时我是用https协议进行本地仓库和远程仓库的连接的。


现在输入如下指令进行查询：

```
git remote -v
```


可以看到是否是https的原因。如果是，解决方法如下：

现在可以进行如下修改解决问题，换成ssh方式：

```
git remote rm origin
git remote add origin git@github.com:whbing147/learngit.git
git push -u origin master
```


