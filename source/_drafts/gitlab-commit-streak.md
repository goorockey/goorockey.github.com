title: "Gitlab commit streak实现日志"
date: 2015-04-28
tags: [gitlab, git]
categories: others
---

## 缘起

当时想在所在团队弄个私有的git服务，比较了最常用的Gitolite, Gitlab和Gogs，最后选择了[Gitlab](https://about.gitlab.com/)，主要考虑是Gitlab跟其他服务集成方便一些。其实当时挺想用Gogs，它是国人用golang开发的，风格简单大方，但试了一下觉得还不够成熟，只好放弃了，希望它快点强大起来。

跟许多开源项目一样，Gitlab分为社区版(Gitlab CE)和企业版(Gitlab EE)，社区版一般就够用了。按照[官网的介绍](https://about.gitlab.com/downloads/)安装了Gitlab Omnibus包，很简单就搞定了。

用了一段时间Gitlab之后，总体还是很稳定的，资源占用也可以接收。它的迭代更新也非常快，刚开始用的时候，Gitlab还没有Github的contribute calendar功能，正准备自己实现的，竟然没过多久就发现有人先实现了，开源力量真是强大。

另外，Gitlab还一直没有Github的commit streak功能，网上也有相关的[feature request](http://feedback.gitlab.com/forums/176466-general/suggestions/5863108-implement-github-like-commit-streak)。这个功能可以通过满足个人的虚荣，来提高大家提交的热情，还是挺重要的。为免又被人捷足先登，就动手搞了。

## 准备

Gitlab是用Ruby on Rails开发的，整个系统的运行还利用了redis、nginx、sidekiq、unicorn、postgres，整体的架构可以看[官方文档](https://github.com/gitlabhq/gitlabhq/blob/master/doc/development/architecture.md)。

Gitlab还提供了[Gitlab Development Kit](https://gitlab.com/gitlab-org/gitlab-development-kit)，以方便开发。但我尝试用它的时候，搭建了几次总是遇到问题，就放弃了。留个小tips，就是把每一部分clone下来之后，把Gemfile的源改成[淘宝的镜像](https://ruby.taobao.org/)，快不少。

最后决定把gitlab的仓库clone下来之后，直接在production版的/opt/gitlab/里面改代码，之后把改动同步到仓库，省心省力。

Gitlab的文档相对还比较多，虽然开发方面的我觉得还略显不够。想向Gitlab贡献代码的，要先看看[官方的介绍](https://github.com/gitlabhq/gitlabhq/blob/master/CONTRIBUTING.md)。

## commit streak的实现

终于进入正题。。

Gitlab Omnibus的rails代码默认都在/opt/gitlab/embedded/service/gitlab-rails，主要关注的是app、db、lib、spec目录，分别是主程序代码、数据库迁移升级(migrate)脚本、依赖的库以及单元测试用例。



