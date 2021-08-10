title: "Gitlab commit streak实现日志"
date: 2015-04-28 00:00:00
tags: [Gitlab, Github, programming, 开源]
categories: programming
---

# 缘起

当时想在所在团队弄个私有的git服务, 比较了最常用的Gitolite, Gitlab和Gogs, 最后选择了[Gitlab](https://about.gitlab.com/), 主要考虑是Gitlab跟其他服务集成方便一些。其实当时挺想用Gogs, 它是国人用golang开发的, 风格简单大方, 但试了一下觉得还不够成熟, 只好放弃了, 希望它快点强大起来。

跟许多开源项目一样, Gitlab分为社区版(Gitlab CE)和企业版(Gitlab EE), 社区版一般就够用了。按照[官网的介绍](https://about.gitlab.com/downloads/)安装了Gitlab Omnibus包, 很简单就搞定了。

用了一段时间Gitlab之后, 总体还是很稳定的, 资源占用也可以接收。它的迭代更新也非常快, 刚开始用的时候, Gitlab还没有Github的contribute calendar功能, 正准备自己实现的, 竟然没过多久就发现有人先实现了[这个功能](https://github.com/gitlabhq/gitlabhq/pull/6958), 开源力量真是强大。

另外, Gitlab还一直没有Github的commit streak功能, 网上也有相关的[feature request](http://feedback.gitlab.com/forums/176466-general/suggestions/5863108-implement-github-like-commit-streak)。这个功能可以通过满足个人的虚荣, 来提高大家提交的热情, 还是挺重要的。为免又被人捷足先登, 就动手搞了。

<!--more-->

# 准备

Gitlab是用Ruby on Rails开发的, 整个系统的运行还利用了redis, nginx, sidekiq, unicorn, postgres, 整体的架构可以看[官方文档](https://github.com/gitlabhq/gitlabhq/blob/master/doc/development/architecture.md)。

Gitlab还提供了开发环境[Gitlab Development Kit](https://gitlab.com/gitlab-org/gitlab-development-kit), 还是比较方便的。留个小tips, 就是把每一部分clone下来之后, 把Gemfile的源改成[淘宝的镜像](https://ruby.taobao.org/), 快不少。

Gitlab的文档相对还比较多, 虽然开发方面的我觉得还略显不够。想向Gitlab贡献代码的, 最好先看看[官方的介绍](https://github.com/gitlabhq/gitlabhq/blob/master/CONTRIBUTING.md)。

# commit streak的实现

终于进入正题, 目标是在Gitlab的用户页面, 加上commit streak信息, 包括Contributions in last year(去年总的贡献数), Longest streak(最长连续提交数), Current streak(当前的连续提交数)。

Gitlab Omnibus的rails代码默认都在`/opt/gitlab/embedded/service/gitlab-rails`, 主要关注的是app、db、lib、spec目录, 分别是主程序代码、数据库迁移升级(migrate)脚本、依赖的库以及单元测试用例。

## 新增数据库字段

首先是对数据库的用户表新增以下字段

    longest_streak_start_at       :date
    longest_streak_end_at         :date
    current_streak                :integer          default(0)
    last_contributed_at           :date

新增的办法是利用Rails的Migrate机制，通过在db/migrate/添加一个20150430101032_add_streak_to_users.rb文件，对users表新增字段。

{% codeblock 20150430101032_add_streak_to_users.rb lang:ruby %}
class AddStreakToUsers < ActiveRecord::Migration
  def change
    add_column :users, :longest_streak_start_at, :date
    add_column :users, :longest_streak_end_at, :date
    add_column :users, :current_streak, :integer, default: 0
    add_column :users, :last_contributed_at, :date
  end
end
{% endcodeblock %}

然后在命令行执行以下命令即可

    sudo gitlab-rake db:migrate RAILS_ENV=production

为了证明字段已经添加成功，可以执行以下命令进入postgres数据库查看。

    sudo su - gitlab-psql # 切换到gitlab-psql用户
    /opt/gitlab/embedded/bin/psql gitlabhq_production #登录postgres，访问gitlabhq_production数据库

    \d+ users # 查看users表的结构

## 修改MVC代码

通过grep查找关键字, 定位到用户页的view是在app/views/users, 里面都是[HAML格式](http://haml.info/)的文件, 之前用过Jade差不多。在里面添加了一个_streak.html.haml, 并在show.html.haml中包含了对它的渲染。

{% codeblock app/views/_streak.html.haml lang:haml %}
.user-streak.row.prepend-top-20.text-center
  .col-md-4
    %p.light
      Contributions in last year
    %h3
      #{@streak_info[:contribution_year]} total
    %p.light
      #{@streak_info[:contribution_year_period]}

  .col-md-4
    %p.light
      Longest streak
    %h3
      #{@streak_info[:longest_streak]} days
    %p.light
      #{@streak_info[:longest_streak_period]}

  .col-md-4
    %p.light
      Current streak
    %h3
      #{@streak_info[:current_streak]} days
    %p.light
      #{@streak_info[:current_streak_period]}
{% endcodeblock %}

然后在app/controllers/user_controller.rb加上streak_info函数，以及在show函数调用它。

{% codeblock app/controllers/user_controller.rb lang:ruby %}
def streak_info
  @streak_info = {
    contribution_year:
      contributions_calendar.events_by_period((Time.now - 1.year), Time.now).count,
    contribution_year_period: @user.contribution_year_period,

    longest_streak: @user.longest_streak,
    longest_streak_period: @user.longest_streak_period,

    current_streak: @user.current_streak,
    current_streak_period: @user.current_streak_period,
  }
end
{% endcodeblock %}

接着就是改model部分，在app/models/user.rb添加新增的model函数。

{% codeblock app/models/user.rb lang:ruby %}
def contribution_year_period
  streak_date_format(1.year.ago) + ' - ' + streak_date_format(Date.today)
end

def longest_streak
  last_contributed_at ? (longest_streak_end_at - longest_streak_start_at + 1).to_i : 0
end

def longest_streak_period
  return 'No contribution yet' if longest_streak.zero?

  if longest_streak_start_at == longest_streak_end_at
    streak_date_format(longest_streak_start_at)
  else
    streak_date_format(longest_streak_start_at) + ' - ' + streak_date_format(longest_streak_end_at)
  end
end

def current_streak_period
  return 'No contribution yet' if longest_streak.zero?

  if last_contributed_at + 1 < Date.today
    return 'Last contributed at ' + streak_date_format(last_contributed_at)
  end

  if current_streak == 1
    streak_date_format(last_contributed_at)
  else
    streak_date_format(last_contributed_at - current_streak + 1) + ' - ' + streak_date_format(last_contributed_at)
  end
end

def streak
  return if last_contributed_at == Date.today

  self.current_streak ||= 1
  self.current_streak += 1 if last_contributed_at && (last_contributed_at + 1 == Date.today)

  if current_streak > longest_streak
    self.longest_streak_start_at = Date.today - current_streak + 1
    self.longest_streak_end_at = Date.today
  end

  self.last_contributed_at = Date.today
  self.save
end

private

def streak_date_format(date)
  date.strftime("%b %d,%Y")
end
{% endcodeblock %}

由于需要对每个contribution响应，更新计数，还要在app/service/event_create_service.rb中添加相应代码。

{% codeblock services/event_create_service.rb lang:ruby %}
def create_event(project, current_user, status, attributes = {})
  attributes.reverse_merge!(
    project: project,
    action: status,
    author_id: current_user.id
  )

  event = Event.create(attributes)
  if event && event.contribute? # 响应contribution
    current_user.streak
  end

  event
end
{% endcodeblock %}

最后在app/assets/stylesheets添加相应的css文件。

{% codeblock generic/streak.css lang:css %}
.user-streak {
  p.light {
    margin: 0px;
  }

  h3 {
    margin: 10px 0px;
  }
}
{% endcodeblock %}

修改js、css等assets资源，需要执行以下命令重新编译资源：

    sudo gitlab-rake assets:clean assets:precompile cache:clear RAILS_ENV=production
    sudo gitlab-ctl restart

所有改动可以查看我的提交[d69031e](https://github.com/goorockey/gitlabhq/commit/d69031e9add519ece1ebd278d8180a8628ace6d7)。

# 提交pull request

参考[官方文档](https://github.com/gitlabhq/gitlabhq/blob/master/CONTRIBUTING.md#merge-request-guidelines)提交PR。

值得注意的是提交之前，参考[这里](http://git-scm.com/book/en/Git-Tools-Rewriting-History#Squashing-Commits)，把多个commit合并成一个。

修改了界面的话，在PR中要附上修改前后的截图。

最后修改截图如下

![gitlab-commit-streak](http://goorockey.github.io/uploads/2015/gitlab-commit-streak.png)

Github上提交的PR在[这里](https://github.com/gitlabhq/gitlabhq/pull/9233)。

# 最后

提交了没多久，就有人回复表示赞，心里有些小激动。负责项目管理的人也很快review了代码，提出了修改意见，以及希望我补上相关的测试用例。

以前对别人项目的contribution，都是提一下issue，这次是第一次PR，而且是对新增功能的提交，终于有种通过Github参与开源项目的感觉，再接再厉 :D
