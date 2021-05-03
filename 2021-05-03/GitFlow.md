## 简述 Git 的工作流

#### 1. GitFlow的优势
GitFlow工作流程的优势在于：

- 还处于半成品状态的feature不会影响到主干
- 各个开发人员之间做自己的分支，互不干扰
- 主干永远处于可编译、可运行的状态

#### 2. GitFlow分支简述
##### 2.1 主干分支（master和develop）
1. master分支存储了发布版本的历史，各个版本通过tag来标记（git tag -a v0.1）
2. develop分支是一个集成分支，用来整合各个功能feature分支，也方便给master分支上的所有提交分配一个版本号。
主干分支

除了master和develop主分支线，其他的分支都是临时的分支，有一定的生命周期的，其余的工作流程分支都是围绕这两个分支之间的区别进行的。

##### 2.2 功能分支(feature)
###### 功能分支
开发每个功能都必须新开个feature分支，feature分支派生自develop分支。开发完成后要合并回develop分支。feature分支永远不会和master分支打交道。

##### 2.3 待发布分支(release)
###### 待发布分支
release分支不是一个放正式发布产品的分支，可以理解为“预发布”或者“待发布”分支。
当开发的功能完成并满足发布的条件时，将这些满足条件的feature分支合并到develop分支上，然后从develop分支开出一个release分支，开始准备一个发布版本。
在release分支上，不能再添加新的功能，但是我们可以:

- 将分支打包给测试人员测试
- 在这个分支上修改bug
- 编写发布文档

当到发布日时，发布相关的工作都完成后，release分支合并回master分支，并打出版本标签，发布完成后，release分支合还要并回develop分支。

##### 2.4 维护分支(hotfix)
###### 维护分支
维护分支也就是bug修复分支，用来快速修复生产环境的紧急问题。

项目发布后或多或少会有一些bug存在，而bug的修复工作并不适合在develop上做，这是因为：

develop分支上可能包含还未验证过的feature
用户未必需要develop上的feature
develop还不能马上发布，而客户急需这个bug的修复。
这个分支是唯一一个开放过程中直接从master分支派生来的分支。
快速的修复问题后，hotfix分支应该被合并回master分支，同时也要合并回develop分支，这样develop分支也能享受到bug修复的好处。然后master分支需要打一个版本标签，例如v0.11。

#### 3.GitFlow的命名约定
- 主分支名称：master
- 主开发分支名称：develop
- 标签（tag）名称：v##，如：v1.0.0
- 新功能开发分支名称：feature-## or feature/##，如：feature-games或feature/games
- 发布分支名称：release-## or release/##，如：release-1.0.0或release/1.0.0
- 维护分支名称：hotfix-## or hotfix/## ，如：hotfix-update或hotfix/update

#### 4.GitFlow的工作流程
##### 4.1 创建develop分支
在本地master基础上创建一个develop分支，然后push到服务器；


```
git branch develop
git push -u origin develop
```

以后这个分支将会包含项目的全部历史，而master分支将只包含了部分历史，建好develop分支的跟踪分支：


```
git clone ssh://user@host/path/to/repo.git
git checkout -b develop origin/develop
```

##### 4.2 新建feature分支
###### 新建feature分支
基于develop分支创建新功能分支：


```
git checkout -b feature/demo develop
```

推送到远程仓库，共享：


```
git push
```

在此分支上开发提交代码:


```
git status
git add
git commit -m '***'
```

##### 4.3 完成新功能开发（合并feature分支到develop）
###### 完成新功能开发
当确定新功能开发完成，且联调测试通过，合并feature分支到develop。


```
git pull origin develop
git checkout develop
git merge --no-ff feature/demo
git push
git branch -d feature/demo
```

##### 4.4 新建待发布分支release
###### 新建待发布分支
项目准备发布时，基于develop分支新建一个待发布分支release,确立版本号:


```
git checkout -b release/v0.1 develop
```

推送到远程仓库共享：


```
git push
```


##### 4.5 release分支合并到master发布
如果准备好了对外发布，就将release分支合并到master分支和develop分支上，并删除发布分支。
发布分支
release分支合并到master分支:


```
git checkout master
git merge --on-off release/v0.1
git push
```

release分支合并到develop分支，合并完成后并删除发布分支:


```
git checkout develop
git merge --on-off release/v0.1
git push
git branch -d release/v0.1
```

合并回develop分支很重要，因为在发布分支中已经提交的更新需要在后面的新功能中也要是可用的。

发布分支是作为功能开发（develop分支）和对外发布（master分支）间的缓冲。只要有合并到master分支，就应该打好Tag以方便跟踪。


```
git tag -a v0.1 -m 'Initial public release' master
git push --tags
```

##### 4.6 线上Bug修复流程
###### 线上Bug修复
为了处理bug，需要从master中创建出维护分支hotfix，等到bug修复完成，需要合并回master

基于master新建hotfix分支：


```
git checkout -b hotfix/v0.1.0.1 master
```

当问题修复完成，并测试通过后，将hotfix分支合并到master分支和develop分支，并打出一个标签。

将hotfix分支合并到master分支:


```
git checkout master
git merge --on-off hotfix/v0.1.0.1
git push
```

将hotfix分支合并到develop分支,合并完成后删除hotfix分支：


```
git checkout develop
git merge --on-off hotfix/v0.1.1
git push
git branch -d hotfix/v0.1.1
```

###### 打标签：


```
git tag -a v0.1.1 -m 'Initial public release' master
git push --tags
```
