# 31、版本管理系统 Git 和 GitLab



## DevOps 简介

### 传统开发模型

软件开发生命周期SDLC (Software Development Life Cycle)由计划,分析,设计,实现,测试集成和维护组成

<img src="5day-png/31传统开发模型.png" alt="image-20250219205500408" style="zoom:50%;" />

- 阶段1: 计划和需求分析 (Planning and Requirement Analysis) 

  每个软件开发生命周期模型都从分析开始，过程的利益相关者讨论对最终产品的要求。此阶段的目 标是系统要求的详细定义。此外，还需要确保所有流程参与者都清楚地了解任务以及每个需求将如 何实施。通常，讨论涉及质量保证专家，如果有必要，他们甚至可以在开发阶段干预过程中的添加。

- 阶段2: 设计项目架构 (Project Architecture) 

  在软件开发生命周期的第二阶段，开发人员实际上正在设计架构。所有利益相关者（包括客户）都 会讨论此阶段可能出现的所有不同技术问题。此外，还定义了项目中使用的技术，团队负载，限 制，时间范围和预算。最合适的项目决策是根据定义的要求做出的。

- 阶段3: 开发和编程 (Development and Coding) 

  在批准要求后，该过程进入下一阶段 - 实际开发。程序员从这里开始编写源代码，同时牢记先前定 义的需求。系统管理员调整软件环境，前端程序员开发程序的用户界面以及与服务器交互的逻辑。 编程本身一般会用四个阶段:算法开发,源代码编写,编译,测试和调试

- 阶段4: 测试 (Testing) 

  测试阶段包括调试过程。开发过程中遗漏的所有代码缺陷都会在此处检测到，记录下来并传回给开 发人员进行修复。重复测试过程，直到删除所有关键问题并且软件工作流程稳定。

- 阶段5: 部署 (Deployment) 

  当程序最终确定并且没有关键问题时 - 是时候为最终用户启动它了。新程序版本发布后，技术支持 团队加入。该部门提供用户反馈; 在利用期间咨询和支持用户。此外，此阶段还包括所选组件的更 新，以确保软件是最新的，并且不会受到安全漏洞的影响。

**SDLC 模型 (Software Development Life cycle Model)**

从第一个也是最古老的“瀑布式”SDLC模型演变而来，其种类不断扩大。比较常见的SDLC模型如下 

- 瀑布模型 (Waterfall Model) 
- 迭代模型 (Iterative Model) 
- 螺旋模型 (Spiral Model) 
- V形模型 (V-Shape Model)： 单元测试，集成测试，系统测试，验收测试 
- 敏捷模型 (Agile Model)

#### 瀑布模型

瀑布模型是早期实现的开发模型,是一个级联SDLC模型，其中开发过程看起来像制造业的流水线，一步 一步地进行分析，预测，实现，测试，实施和支持阶段。 

该SDLC模型包括完全逐步执行每个阶段。该过程严格记录并预定义，具有该软件开发生命周期模型的每 个阶段所期望的功能。 

类似工厂中流水线的传送带，加工的产品只能按照一道一道的工序向下不断前进，不能后退。

<img src="5day-png/31瀑布模型.png" alt="image-20250219210025723" style="zoom: 33%;" />

| 优点                                    | 缺点                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| 简单易用和理解                          | 各个阶段的划分完全固定，阶段之间产生大量的文档，极大地增 加了工作量 |
| 当前一阶段完成后，只需要 去关注后续阶段 | 由于开发模型是线性的，用户只有等到整个过程的末期才能见到 开发成果，从而增加了开发风险 |
| 为项目提供了按阶段划分的 检查节点       | 不适应用户需求的变化                                         |



#### V 形模型

<img src="5day-png/31V型模型.png" alt="image-20250219210241072" style="zoom:33%;" />



#### 敏捷开发

Agile Model 敏捷开发的核心是迭代开发(lterative Development)与增量开发 (IncrementalDevelopment) 。



<img src="5day-png/31敏捷开发.png" alt="image-20250219210403182" style="zoom: 50%;" />

**迭代开发:** 

对于大型软件项目，传统的开发方式是采用一个大周期(比如一年或数年)进行开发，整个过程就是一 次"大开发"

迭代开发的方式则不一样，它将开发过程拆分成多个小周期，即一次"大开发"变成多次"小开发"，每次小 开发都是同样的流程，所以看上去就好像重复在做同样的步骤

比如: 某米公司生产手机,第一次生产某米1代(比较简陋),逐年再生产某米2代,目前已经到某米1X代

**增量开发:** 

软件的每个版本，都会新增一个可以被用户感知到的完整功能。也就是说，按照新增功能来划分迭代。 

比如:  

房地产公司开发一个小区。如果采用增量开发的模式，该公司第一个迭代就是交付第一期楼盘，第二个 迭代交付第二期楼盘..….每个迭代都是只完成一期完整的楼盘。而不是第一次迭代就挖好所有楼盘的地 基，第二个迭代建好所有楼盘的骨架，第三个迭代架设屋顶.…..... 

虽然敏捷开发将软件开发分成多个迭代，但是也要求，每次迭代都是一个完整的软件开发周期，必须按 照软件工程的方法论，进行正规的流程管理。

**优势**

- 尽早交付,加快资金回笼 
- 降级风险,及时了解市场需求，及时获取问题反馈 
- 提高效率,阶段性功能拆分及快速质量反馈

敏捷开发这种小步快跑的方式，大幅提高了开发团队的工作效率，让版本的更新速度变得更快。 

更新版本的速度快了，风险不是更大了吗？其实，事实并非如此。 

敏捷开发可以帮助更快地发现问题，产品被更快地交付到用户手中，团队可以更快地得到用户的反馈， 从而进行更快地响应。而且，DevOps小步快跑的形式带来的版本变化是比较小的，风险会更小（如下 图所示）。即使出现问题，修复起来也会相对容易一些。

![image-20250219210652170](5day-png/31敏捷模式和瀑布模式区别.png)

### DevOps介绍

<img src="5day-png/31DevOps介绍.png" alt="image-20250219210915920" style="zoom:25%;" />

DevOps 即开发 Development 和 Operations运维的缩写。

**DevOps是一组过程、方法与系统的统称，用于促进开发、技术运营和质量保障（QA）部门之间的沟通、协作与整合。**

传统的模式是开发人员只关心开发程序，追求功能的变化和实现  

运维只负责基础环境管理和代码部署及监控等，更看重应用的稳定运行 

双方缺少一个共同的目标

**DevOps组织优先推动的目标** 

- 一组标准化的环境 
- 降低新版本的失败率 
- 缩短版本之间的交付时间 
- 更快的平均版本恢复时间 

**DevOps的关键指标** 

- 平均恢复时间: Mean-time to recovery (MTTR) 
- 平均投产时间: Mean-time to production 
- 平均提前时间:Average lead-time，比计划时间提前 
- 部署速度:Deployment speed 
- 部署频率:Deployment frequency 
- 生产失败率:Production failure rate

**DevOps 涉及的四大相关平台**

- 项目管理：如：Jira,禅道 
- 代码托管：如：Gitlab,SVN 
- 持续交付：如：Jenkins,Gitlab 
- 运维平台：如：腾讯蓝鲸,Spug等

### DevOps 流水线模型

- 第 1 步：需求和用户故事 

  该过程从产品所有者根据业务需求创建用户情景开始。这些用户故事是开发团队工作的基础。 

- 第 2 步：冲刺计划 

  开发团队从待办事项中获取用户故事，并将它们组织到冲刺中，通常遵循敏捷方法。每个冲刺可能跨越 一个固定的时间段，例如两周的开发周期。 

- 第 3 步：源代码提交 

  开发人员将他们的源代码提交到 Git 等版本控制系统中。这可确保跟踪更改，并且可以管理不同版本的 代码。

- 步骤 4：持续集成 （CI） 

  生成过程是使用 CI 工具（如 Jenkins）启动的。源代码经过严格的测试，包括单元测试、代码覆盖率检 查和代码质量门。 SonarQube 等工具有助于维护代码质量标准。 

- 第 5 步：工件管理 

  成功的生成存储在项目存储库中，例如 Artifactory。此存储库允许轻松检索以前的构建和项目。 

- 步骤 6：部署到开发环境 

  新代码将部署到开发环境中进行初始测试和集成。这种环境有助于发现不同开发分支之间的早期问题和 冲突。 

- 第 7 步：隔离测试 

  在多个开发团队处理不同功能的情况下，单独测试这些功能至关重要。因此，代码被部署到单独的 QA1  和 QA2 环境中以避免干扰。 

- 第 8 步：QA 测试 

  质量保证 （QA） 团队接管新的 QA 环境并执行各种测试，包括功能测试、回归测试和性能测试。目标 是在继续操作之前识别并解决任何问题。 

- 步骤 9：用户验收测试 （UAT） 

  QA 测试成功通过后，代码将继续进入用户验收测试 （UAT） 环境。利益干系人验证它是否满足他们的 要求和期望。 

- 第 10 步：准备生产 

  如果 UAT 测试成功，则内部版本将成为候选版本。根据发布计划，计划将它们部署到生产环境。 

- 第 11 步：站点可靠性工程 （SRE） 

  站点可靠性工程 （SRE） 团队承担着监控生产环境的关键责任。他们确保其稳定性和可靠性，并及时响 应任何问题。

总的来说，以上这个流程展示了一个高度自动化、协调一致、质量保障的 DevOps 实践，符合一般的大 厂在软件开发和部署方面的要求。 

这种结构化的发布过程可确保代码更改在到达生产环境之前经过全面测试并满足业务要求。 

虽然具体的工具和方法可能因公司而异，具体的流程可能因组织和项目的不同而有所调整，但在许多软 件开发组织中，发布过程的核心结构保持一致。

### DevOps 和 NoOps

<img src="5day-png/31devops&noops.png" alt="image-20250219211753993" style="zoom: 80%;" />



### 持续集成、持续交付和持续部署 CICD

最初是瀑布模型，后来是敏捷开发，现在是DevOps，这是现代开发人员构建出色的产品的技术路线。 随着DevOps的兴起，出现了**持续集成（Continuous Integration）、持续交付（Continuous  Delivery） 、持续部署（Continuous Deployment） 的新方法。** 

传统的软件开发和交付方法正在迅速变得过时。从以前的敏捷时代，大多数公司会每月，每季度，每两 年甚至每年发布部署/发布软件。然而现在在DevOps时代，每周，每天，甚至每天多次是常态。 

当SaaS正在占领世界时，尤其如此，您可以轻松地动态更新应用程序，而无需强迫客户下载新组件。很 多时候，他们甚至都不会意识到正在发生变化。开发团队通过软件交付流水线（Pipeline）实现自动 化，以缩短交付周期，大多数团队都有自动化流程来检查代码并部署到新环境。

CI/CD 是一种通过在应用开发阶段引入自动化来频繁向客户交付应用的方法。CI/CD 的核心概念是持续 集成、持续交付和持续部署。作为一个面向开发和运营团队的解决方案，CI/CD 主要针对在集成新代码时所引发的问题 

具体而言，CI/CD 可让持续自动化和持续监控贯穿于应用的整个生命周期（从集成和测试阶段，到交付 和部署）。这些关联的事务通常被统称为“CI/CD 管道”，由开发和运维团队以敏捷方式协同支持。















































## Git

















## Gitliab安装

```bash
Thank you for installing GitLab!
GitLab was unable to detect a valid hostname for your instance.
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
  sudo gitlab-ctl reconfigure

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

Help us improve the installation experience, let us know how we did with a 1 minute survey:
https://gitlab.fra1.qualtrics.com/jfe/form/SV_6kVqZANThUQ1bZb?installation=omnibus&release=17-8

Scanning processes...                                                                                                                                                 
Scanning linux images...                                                                                                                                              

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
N: Download is performed unsandboxed as root as file '/root/gitlab-ce_17.8.2-ce.0_amd64.deb' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)
```

```bash
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
  sudo gitlab-ctl reconfigure
```

#### Git 本地设置

本地配置您的 Git 身份，仅在此项目中使用：

```
git config --local user.name "kang"
git config --local user.email "kang@qq.com"
```



### 添加文件

使用 SSH 或 HTTPS 将文件推送到此存储库。如果不确定，建议使用 SSH。

- [SSH](http://github.hzk.com/devops/myapp#)
- [HTTPS](http://github.hzk.com/devops/myapp#)

[如何使用 SSH 密钥](http://github.hzk.com/help/user/ssh.md)？

#### 创建一个新仓库

```
git clone git@gitlab.hzk.com:devops/myapp.git
cd myapp
git switch --create main
touch README.md
git add README.md
git commit -m "add README"
git push --set-upstream origin main
```

#### 推送现有文件夹

##### Go to your folder

```
cd existing_folder
```

##### Configure the Git repository

```
git init --initial-branch=main
git remote add origin git@gitlab.hzk.com:devops/myapp.git
git add .
git commit -m "Initial commit"
git push --set-upstream origin main
```

#### 推送现有的 Git 仓库

##### Go to your repository

```
cd existing_repo
```

##### Configure the Git repository

```
git remote rename origin old-origin
git remote add origin git@gitlab.hzk.com:devops/myapp.git
git push --set-upstream origin --all
git push --set-upstream origin --tags
```