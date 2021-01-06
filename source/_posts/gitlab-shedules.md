---
title: GitLab CI/CD Pipelene schedules
date: 2020-10-15 23:00:48
tags: [Git]
categories: [学习笔记]
---

https://docs.gitlab.com/ee/ci/pipelines/schedules.html

<!--more-->



### Pipeline Scheules

Pipeline通常是在满足某些条件的情况下运行的。例如，将分支推送到存储库时。

pipeline schedules可以用于以特定间隔运行pipeline 。例如：

- 每个月的22号都执行特定的分支。
- 每天一次。

除了使用GitLab UI外，还可以使用[Pipeline schedules API](https://docs.gitlab.com/ee/api/pipeline_schedules.html)维护pipeline schedule

计划时间由cron表示法配置，由 fugit 解析。



## 先决条件

为了成功创建计划的pipeline：

- 计划所有者必须具有合并到目标分支的权限。
- pipeline配置必须有效。

否则，不会创建pipeline。



## 配置pipeline schdeules

计划项目的pipeline：

1. 导航到项目的**CI / CD>计划**页面。
2. 单击**新建计划**按钮。
3. 在“**计划”中**填写**新的pipeline**表单。
4. 单击**保存pipeline计划**按钮。

[![New Schedule Form](https://docs.gitlab.com/ee/ci/pipelines/img/pipeline_schedules_new_form.png)](https://docs.gitlab.com/ee/ci/pipelines/img/pipeline_schedules_new_form.png)

**注意：** pipeline执行[时间取决于](https://docs.gitlab.com/ee/ci/pipelines/schedules.html#advanced-configuration)Sidekiq自己的时间表。

在“**计划**索引”页面中，您可以看到计划运行的pipeline的列表。下次运行由安装了GitLab的服务器自动计算。

[![Schedules list](https://docs.gitlab.com/ee/ci/pipelines/img/pipeline_schedules_list.png)](https://docs.gitlab.com/ee/ci/pipelines/img/pipeline_schedules_list.png)

### 

### 使用变量

在GitLab 9.4中[引入](https://gitlab.com/gitlab-org/gitlab-foss/-/merge_requests/12328)。

您可以传递任意数量的任意变量，它们将在GitLab CI / CD中可用，以便可以在您的[`.gitlab-ci.yml`文件中使用](https://docs.gitlab.com/ee/ci/yaml/README.html)。

[
  ](https://docs.gitlab.com/ee/ci/pipelines/img/pipeline_schedule_variables.png)

### 使用only and except

要配置仅在计划 pipeline（或相反）时才可以执行作业，您可以使用only 和 except 配置关键字。

例如：

```
job:on-schedule:
  only:
    - schedules
  script:
    - make world

job:
  except:
    - schedules
  script:
    - make build
```



### 进阶设定

pipeline不会完全按计划执行，因为计划由Sidekiq处理，Sidekiq根据其间隔运行。

例如，在以下情况下，每天将仅创建+两个pipeline：

- 您设置了时间表以每分钟（`* * * * *`）创建一条pipeline。
- Sidekiq工作人员每天（`0 */12 * * *`）在00:00和12:00运行。

更改Sidekiq工作人员的频率：

1. 编辑`gitlab_rails['pipeline_schedule_worker_cron']`实例`gitlab.rb`文件中的值。
2. [重新配置GitLab，](https://docs.gitlab.com/ee/administration/restart_gitlab.html#omnibus-gitlab-reconfigure)以使更改生效。

对于GitLab.com，请参阅专用设置画面



## 使用计划的pipelines

配置完成后，GitLab将支持许多用于计划pipelines的功能。



### 手动运行

在GitLab 10.4中[引入](https://gitlab.com/gitlab-org/gitlab-foss/-/merge_requests/15700)。

要手动触发pipeline计划，请单击“播放”按钮：

[![播放pipeline时间表](https://docs.gitlab.com/ee/ci/pipelines/img/pipeline_schedule_play.png)](https://docs.gitlab.com/ee/ci/pipelines/img/pipeline_schedule_play.png)

这将安排一个后台作业来运行pipeline计划。一条简短消息将提供指向CI / CD pipeline索引页面的链接。

**注意：** 为避免滥用，限制了用户每分钟触发一次pipeline的速率。



### 取得所有权

pipeline以拥有日程表的用户身份执行。这会影响pipeline可以访问的项目和其他资源。

如果用户不拥有pipeline，则可以通过单击“**获取所有权”**按钮**获取所有权**。下次计划pipeline时，将使用您的凭据。

[![时间表清单](https://docs.gitlab.com/ee/ci/pipelines/img/pipeline_schedules_ownership.png)](https://docs.gitlab.com/ee/ci/pipelines/img/pipeline_schedules_ownership.png)

如果pipeline计划的所有者没有能力在目标分支上创建pipeline，则该计划将停止创建新pipeline。

例如，如果发生这种情况：

- 所有者被阻止或从项目中删除。
- 目标分支或标签受保护。

在这种情况下，具有足够特权的人必须拥有日程表的所有权。