---
title: CircleCI实践gh-pages部署
date: 2019-01-31 22:22:35
categories: CircleCI
tags:
- 自动化
- git
- GitHub
- CircleCI
---
这里展示一下CircleCI集成与github之间联系。
<!-- more -->

当你在Circle中添加一个项目时。相关的 GitHub 设置中将会使用您在注册时为CircleCI提供的权限。

* `deloy key` 用来从 GitHub 中查看项目源代码。
* `service hook` 用来通知 CircleCI ，当你向 GitHub 中提交代码。

常见 push hooks：

另外 两个钩子：
* 提供PR hooks，用来储存 PR 请求。
* 如果PR钩子在forked的项目中，CircleCI将会在PRs在分支仓库创建时提交。

详见 [workflows filters](https://circleci.com/docs/2.0/workflows/#using-contexts-and-filtering-in-your-workflows)

CircleCI在每次测试都会清理容器。你的 github 的仪表盘中也会有显示。

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/blog/circleci/status_check.png)

# 使你CircleCI的项目能够检出私有仓库
如果你的测试过程涉及多个存储库，除了部署密钥之外，CircleCI还需要GitHub用户密钥，因为每个部署密钥仅对一个储存库有效，而GitHub用户莫要可以访问所有GitHub储存库。查看[adding ssh keys](https://circleci.com/docs/2.0/add-ssh-key)章节来查看更多。

在 **Project Settings > Checkout SSH keys** 页面提供给 CircleCI GitHub用户私钥。CircleCI创建和关联新的SSH密钥来访问你的所有的仓库。

## 用户密钥安全
CircleCI 绝对不会公开你的SSH密钥。

请记住，SSH密钥只能分享给信任的用户，所有的 GitHub 合作者拥有了SSH密钥都可以访问你的仓库。所以要确保你的SSH密钥的安全。

## 用户密钥访问相关错误信息
下面的错误表明你需要添加用户密钥

**Python**: 在 `pip install`步骤期间

```
ERROR: Repository not found.
```

**Ruby**: 在`bundle install`步骤期间

```
Permission denied (publickey).
```

# 创建机器用户
要对多个存储库进行细粒度访问，请考虑为CircleCI项目创建机器用户。一个机器用户是你为运行自动化任务而创建的GitHub用户。通过使用机器用户的SSH密钥，你可以允许任何人对仓库进行，构造，测试和部署项目。创建机器用户还可以降低丢失与单个用户关联的凭据的风险。

要为机器用户使用SSH密钥，请遵循下列步骤：

**注意**: 要执行下列步骤，机器用户必须拥有管理员权限。完成项目添加后，可以将机器用户权限设为只读。

1. 根据[instructions on GitHub](https://developer.github.com/v3/guides/managing-deploy-keys/#machine-users)，创建机器用户权限。
2. 作为机器用户登陆Github。
3. [登陆CircleCI](https://circleci.com/login)。当GitHub提醒你需要授权给CircleCI的时候，点击 **Authorize application** 按钮。
4. 在[添加项目](https://circleci.com/add-projects), follow你希望使用机器用户访问的项目。
5. 在 **Project Settings > Checkout SSH keys**页面，点击 **Authorize With GitHub** 按钮。这将给CircleCI权限作为机器用户去创造和上传SSH密钥。
6. 点击 **Create and add XXXX user Key** 按钮。

现在Circle将使用机器用户的SSH密钥来使用Git命令行运行你的构建。

# 权限概述
CircleCI要求你的版本控制系统（VCS）提供以下权限，[GitHub permissions model](http://developer.github.com/v3/oauth/#scopes)

## 读权限

* 获取用户的邮件地址

## 写权限

* 给仓库添加部署密钥
* 给仓库添加服务器钩子
* 获取用户所有仓库的列表
* 给用户账户添加SSH密钥

**注意**: CircleCI只有在必须的时候才会请求权限。但是，CircleCI受到VCS提供的权限约束。举例，从GitHub获取所有用户仓库，需要写权限，因为GitHub没有提供只读权限。

如果你强烈的希望减少CircleCI的使用，考虑向你的VCS供应商联系。

## 团队账户权限

本节概述了可用于满足各种业务需求的团队和个人帐户选择：

1. 如果单个用户拥有个人GitHub账户，他会使用它登录CircleCI然后在CircleCI里follow任务。GitHub中该存储库上的每个“协作者”也能够跟踪项目并在推送提交时构建在CircleCI上。由于GitHub储存协作者的方式，CircleCI不会显示完整的列表。有关合作者的完整列表需要参阅GitHub。

2. 如果一个个人用户升级到团队用户，他们将被允许添加用户，甚至给那些运行构建任务的合作者管理者权限。团队用户的拥有者必须到CircleCI的[Add Project](https://circleci.com/add-projects)，点击链接到GitHub应用权限界面，选择授权CircleCI来使组织的成员通过他们的账户follow项目。拥有两名成员的团队帐户每月25美元，而不是个人帐户每月7美元。

3. 对于最多五人的团队，个人Bitbucket账户可免费用于私人回购。个人可以创建Bitbucket团队，添加成员并根据需要为需要构建的人员提供管理员权限。该项目将出现在CircleCI中，供会员遵循，无需额外费用。

## 如何为GitHub组织重新启用CircleCI
这一章节描述了如何在GitHub组织启用了第三方应用程序限制后重新启用CircleCi。参见[GitHub Seeting]。

* "申请进入"，如果你不是组织的管理者会有这种问题。管理者应该批准访问。
* "授予访问权限"如果你是管理者应该给予。

获取访问权限后，CircleCi的行为将会正常。

GitHub最近添加了在每个组织级别[准第三方程序访问权限](third party application access on a per-organization level)。再此之前，组织的成员都能授权给应用（生成 OAhth token 关联他们的GitHub用户账户），而且应用可以使用 OAuth token 来代表用户行事，可以使用OAuth flow中的任何权限。

现在的当第三方组织限制打开的时候，OAuth token默认情况下没有办法访问第三方数据。不管是OAuth进程之前或者之后，你必须准备为组织申请权限，然后组织管理者需要批准权限。

你也可以在GitHub的组织设置界面打开第三方访问权限。并单击“第三方应用程序访问策略”部分中的“设置应用程序访问限制”按钮。

如果您对已运行CircleCI的组织启用这些限制，那么我们将停止从GitHub接收推送事件挂钩（因此不会构建新的推送）。而且API调用将被拒绝（例如，导致重新构建旧版本以使源检出失败。）要使CircleCI再次运行，您必须授予对CircleCI应用程序的访问权限。

# 部署密钥和用户密钥

## 什么是部署密钥？

当你创建新项目的时候，CircleCi将会为你的项目在基于网络的分布式系统上（例如GitHub和Bitbucket）创建密钥。部署密钥是仓库专属的密钥。如果你使用GitHub作为你的分布式系统，而且GitHub有用公钥，CircleCi将会储存私有秘钥。部署密钥给CircleCi访问单个仓库的权限。为了保护CircleCi不能推送内容到你的仓库，部署密钥是只读的。

如果你想要在构建中push内容到你的仓库，你需要一个拥有读写取新鲜的部署密钥，即用户密钥。下面几步展示了为你的VCS创建用户密钥。

## 什么是用户密钥？

一个用户密钥是用户专属的SSH密钥。你的VCS拥有公钥，CircleCi存储私钥。拥有私钥能够给予你模仿用户的能力，像使用`git`访问项目。

## 创建GitHub用户密钥

在下面的例子中，GitHub仓库是`https://github.com/you/test-repo`，你的CircleCi项目是[https://circleci.com/gh/you/test-repo](https://circleci.com/gh/you/test-repo)。

1. 根据[GitHub instructions](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)创建SSH密钥。当提示输入密码的时候，一定不要输入密码。

**警告**：默认情况下，ssh-keygen中的最新更新不会以PEM格式生成密钥。如果你的密钥不是以`-----BEGIN RSA PRIVATE KEY-----`开始，通过使用`ssh-keygen -m PEM -t rsa -C "your_email@example.com"`来强制生成密钥。

1. 到`https://github.com/you/test-repo/settings/keys`，点击"Add deploy key"。在 Title 输入框写下标题，复制粘贴你在第一步创建的密钥，点击"Allow write access"，然后点击"Add key"。

2. 到 [https://circleci.com/gh/you/test-repo/edit#ssh]( https://circleci.com/gh/you/test-repo/edit#ssh)，添加在上面创建的密钥到"Hostname"输入框，输入"github.com"，添加提交按钮

3. 在你的 config.yml ，添加指纹到`add_ssh_keys`。

```yml
version: 2
jobs:
  deploy-job:
    steps:
      - add_ssh_keys:
          fingerprints:
            - "SO:ME:FIN:G:ER:PR:IN:T"
```

## 这些键是怎么用的？

当CircleCi构建你的项目的时候，私有秘钥会被创建到`.ssh`目录，随后将SSH配置为与您的版本控制提供程序进行通信。此后，私有秘钥会给用户提供下列权限：

* 检查主要项目。
* 检查任何 GitHub 托管的子项目。
* 检查任何 GitHub 托管的私有依赖项。
* 自动化 git 的合并和打标签等操作。

由于上述的原因，部署建对于需要添加私有依赖项的项目功能不够强大。

## 关于安全因素？

私钥对于CircleCi生成的密钥绝对不会离开CircleCi系统，只有公钥会被传输给GitHub。私钥会被安全加密的存储。然而，当你将私钥用于你构建的容器，你的CircleCi的任何代码都可以读到他们。

## 部署密钥和用户密钥的区别？

部署密钥和用户密钥是GitHub唯一支持的密钥类型。部署密钥是全局唯一的，举例来说，没有机制能使部署密钥访问多个仓库。而用户密钥没有分割区域，用户可以访问所有与之相关的仓库。

为了实现访问多个不同的仓库，应该考录常见一个GitHub的机器用户。给这个用户你构建项目所需要的权限，然后再CircleCi上关联用户密钥到你的项目。

# 关于GitHub机器用户

如果你的服务需要访问多个仓库，你可以创建一个新的GitHub账户，然后连接的SSH密钥，仅仅用以项目的自动化。这样的Github账户不会被用户使用，这样的Github账户被称作机器账户。你可以添加一个机器用户作为个人账户的合作者（授予读写权限），或者作为公共仓库的外有合作者（授予读写或者管理者权限），或者作为需要自动化的团队仓库。

> GitHub服务条框是:
  不允许机器或者自动化注册账户.
  这表示你不能自动创建账户.但是如果你只是创建一个私有用户来为你的项目或者组织运行自动化脚本,这么做是推荐的.

## Pros
* 任何有权访问仓库或者服务的人都有能力部署项目.
* 真实用户,或者说是人类用户,不需要改变他们的本地SSH密钥.
* 多个密钥是不需要的,一个服务就够了.

## Cons
* 只有组织能够限制机器用户值有读写权限.个人仓库总是授予合作者读写权限.
* 机器用户密钥和部署密钥一样,不受密码保护.

## 创建
1. [运行ssh-keygen过程](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#generating-a-new-ssh-key)在你的服务器上,然后通过机器用户连接公钥.
2. 给及其用户你想自动化的仓库.你可以通过添加账户作为合作者,作为一个外界合作者,或者组织的一个团队.

# gh-pages实践步骤

## 1. 创建机器用户

经过上面的介绍，基本上应该知道。`push` dist 目录到相应的分支，这里需要用到`push`的权限。我们必不可少的就是需要一个用户密钥。使用用户密钥的方法就是利用新创建一个git用户，下面我创建了一个`hddMachine`的用户。然后原仓库的管理者将机器用户添加到合作者中，然后在CircleCI中生成对应的`fingerprints`，并添加到下面的yml文件。这样当我们每次`push`的时候就会调用机器用户权限来使用 git 命令。

## git
```yml
version: 2
jobs:
  build:
    working_directory: ~/repo # directory where steps will run
    docker: # use the docker executor type; machine and macos executors are also supported
      - image: circleci/node:8.10 # the primary container, where your job's commands are run
    filters:
      branches:
        only: master
    steps:
      - add_ssh_keys:
          fingerprints: '41:f1:b3:2d:89:ff:f0:16:a4:eb:49:92:39:27:95:04'
      - checkout # check out the code in the project directory
      - run:
          name: update-npm
          command: 'sudo npm install -g npm@latest'
      - restore_cache: # special step to restore the dependency cache
          # Read about caching dependencies: https://circleci.com/docs/2.0/caching/
          key: dependency-cache-{{ checksum "package.json" }}
      - run:
          name: install-npm
          command: npm install
      - save_cache: # special step to save the dependency cache
          key: dependency-cache-{{ checksum "package.json" }}
          paths:
            - ./node_modules
      - run: # run build
          name: build
          command: npm run build
      - run: # git push
          name: push-github
          command: |
            git config --global user.email "975018306@qq.com"
            git config --global user.name "hddMachine"
            git add -f dist
            git commit -m 'create dist'
            git push origin `git subtree split --prefix dist master`:gh-pages --force

```


# 参考
* https://circleci.com/docs/2.0/configuration-reference/#add_ssh_keys
* https://circleci.com/docs/2.0/language-javascript/
* https://github.com/CircleCI-Public/circleci-demo-javascript-express/blob/master/.circleci/config.yml
* [git-subtree-cant-push](https://stackoverflow.com/questions/13756055/git-subtree-subtree-up-to-date-but-cant-push)
* [GitHub和Bitbucket使用CircleCI](https://circleci.com/docs/2.0/gh-bb-integration/?tdsourcetag=s_pctim_aiomsg)