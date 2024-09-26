#### 仓库

1. 创建远程仓库
2. 创建本地仓库

   ```
   git init
   ```
3. 拉取远程仓库

   ```
   git clone
   ```

#### 分支

1. 查看本地分支，-a查看包括远程的所有分支

   ```
   git branch -a
   ```
2. 创建分支

   ```
   git branch [branch_name]
   ```
3. 切换分支

   ```
   git checkout [branch_name]
   ```
4. 将内容添加到暂存区

   ```
   git add [file_path]
   ```
5. 提交文件到当前分支

   ```
   git commit -m [log_message]
   ex: git commit -m "Add myfile"
   ```
6. 删除分支

   ```
   1. 添加d31imitetr远程仓库
   git remote add d31imiter https://github.com/D31imiter/D31imiter.github.io
   2. 获取最新的数据
   git fetch d31imiter
   3. 删除分支
   git push d31imiter --delete <branch_name>
   ```

#### 推送

将本地分支推送到远程仓库

```
git push origin [branch_name]
```

克隆特定分支：`git clone --branch [branch_name] https://github.com/username/repository.git`

删除远程分支不需要加origin，`branch_name`直接为远程分支名：`git push origin --delete [branch_name]`

当文件夹内容发生变动无法提交到暂存区时，可能是还有.git文件存在

添加远程仓库：`git remote add public-repo https://github.com/TyutCxsys/Netsafe2023.git`

拉取公共仓库的soskun分支

```
git fetch public-repo
git checkout soskun
git pull public-repo soskun
```

将本地的远程分组织推送到远程仓库：`git push public-reop soskun(wangkunyi)`

### 搭建github page

1. 创建[username].github.io储存仓库
2. 将你想要的Jekyll主题克隆到你的[username].github.io储存仓库

   ```
   1. 克隆源仓库
   git clone https://github.com/bencentra/centrarium.git
   2. 将目标仓库作为远程仓库d31imiter添加到本地
   git remote add d31imiter https://github.com/D31imiter/d31imiter.github.io
   3. 将本地仓库内容推送到远程仓库的master分支
   git push https://D31imiter@github.com/D31imiter/d31imiter.github.io.git master
   ```
3. 更新后的同步

   ```
   1. git add .
   2. git commit -m "message"
   3. git push d31imiter master
   ```
