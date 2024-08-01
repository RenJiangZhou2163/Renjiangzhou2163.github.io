git clone -b uat_outerpub https://

 

|                                           |                                                    |
| ----------------------------------------- | -------------------------------------------------- |
| git config  --global http.sslVerify false |                                                    |
| 设置用户名                                | git config  --global user.name *username*          |
| 设置邮箱 (没有双引号)                     | git config  --global user.email *useremail@qq.com* |
| 查看用户名和密码                          | git config  user.name<br />git config  user.email  |
| 查看其他配置信息(git设置列表)             | git config  --list                                 |

 

git config credential.helper store



# 提交代码

```
git pull
git add .
git commit -m "update"
git push
```

