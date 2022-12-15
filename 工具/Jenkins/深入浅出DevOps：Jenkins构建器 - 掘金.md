# 深入浅出DevOps：Jenkins构建器 - 掘金
我报名参加金石计划1期挑战——瓜分10万奖池，这是我的第4篇文章，[点击查看活动详情](https://s.juejin.cn/ds/jooSN7t "https://s.juejin.cn/ds/jooSN7t")

> 💯 作者： **俗世游子**【谢先生】。 8年开发3年架构。专注于Java、云原生、大数据等领域技术。  
> 💥 成就： 从CRUD入行，负责过亿级流量架构的设计和落地，解决了千万级数据治理问题。  
> 📖 同名社区：[掘金](https://juejin.cn/user/3359725700263694 "https://juejin.cn/user/3359725700263694")​、​[​github​](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fxiezhyan "https://github.com/xiezhyan")​​、​[​51CTO​](https://link.juejin.cn/?target=https%3A%2F%2Fblog.51cto.com%2Fu_14948012 "https://blog.51cto.com/u_14948012")​、​[​gitee​](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fmr_sanq "https://gitee.com/mr_sanq")​​  
> 📂 清单： ​​[​goku-framework​](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fxiezhyan%2Fgoku-framework "https://github.com/xiezhyan/goku-framework")​​、​[​【更新中】享阅读II​](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fxiezhyan%2Fenjoy-read-ii "https://github.com/xiezhyan/enjoy-read-ii")​

[深入浅出DevOps：DevOps核心思想](https://juejin.cn/post/7134696448616038414 "https://juejin.cn/post/7134696448616038414")

[深入浅出DevOps：版本控制Git&Gitlab](https://juejin.cn/post/7135949075387514894 "https://juejin.cn/post/7135949075387514894")

[深入浅出DevOps：持续集成工具Jenkins](https://juejin.cn/post/7137300017152262151 "https://juejin.cn/post/7137300017152262151")

[深入浅出DevOps：简易Docker入门](https://juejin.cn/post/7137666687872008205 "https://juejin.cn/post/7137666687872008205")

[深入浅出DevOps：Jenkins实战之CI](https://juejin.cn/post/7139406386911248414 "https://juejin.cn/post/7139406386911248414")

[深入浅出DevOps：SonarQube提升代码质量](https://juejin.cn/post/7140608660006240270 "https://juejin.cn/post/7140608660006240270")

[深入浅出DevOps：SonarQube提升代码质量【下】](https://juejin.cn/post/7142440715421745165 "https://juejin.cn/post/7142440715421745165")

前言
--

我们从开始学习Jenkins到现在，大家肯定已经尝试了很多次构建项目的过程

但其实我想说的是，大家在构建的时候是不是觉得目前的构建方式比较繁琐：

*   打开浏览器，进入到Jenkins
*   在Jenkins中找到对应的任务
*   然后点击立即构建

这样才能开始构建当前任务，但是如果有这样一种方式，不需要上述繁琐的过程，是不是会比较舒服呢：

*   Jenkins能够自己去GitLab上拉取代码，拉取到代码之后自动执行后续脚本进行构建；
*   或者说当我们将代码提交到GitLab上之后，Jenkins通过某种方式得到通知然后就能执行构建。

别说，这样想想都挺开心的对不对

别急，功能强大的Jenkins早就为我们准备好了一切。这不就要跟大家介绍关于Jenkins的另一大便捷的功能，容我卖个关子，大家继续往下看

构建触发器
-----

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02912b29b6894fbfbfc39fadb57c8806~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

触发器是任务配置中的一个核心模块，它实现了我们在上面的一切设想。接下来我们就来好好聊一聊这个触发器

> 我们挑几种比较常用的触发器来介绍

### 其他工程构建后触发

如果我们的项目需要依赖其他项目才能正常执行，此时在构建的时候就有一个先后顺序，我们基于该触发器下的配置项进行构建项目，根据我们所设置的关注项目，最终在构建的时候会形成类似责任链模式的长链

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1cf82bf31fdb4df083fd12dafd8d5215~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

> 这种方式非常简单，就不多介绍

### 定时构建

定时构建，如其名就是说Jenkins提供类似cron特性的方式来定期执行构建项目。

这种方式我们需要结合公司实际业务思考，如果公司是习惯于像每晚/每周这样定期安排构建的，那么非常适合采用定时构建的触发器，否则就需要慎重

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a769979015864370829ac3d3a70fa6f5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

在显示出来的日程表的写入Jenkins支持的定时表达式，Jenkins会根据表达式自动给个提示，显示出下次运行时间，而关于定时表达式的相关内容，我们继续往下看

#### 定时表达式

##### 基本信息

```sql
MINUTE HOUR DOM MONTH DOW

```

具体来说，每一行由 5 个字段组成，由 TAB 或空格分隔，其中表示：

*   **MINUTE**

一小时内的分钟数 (0–59)

*   **HOUR**

一天中的小时 (0–23)

*   **DOM**

月份中的某天 (1–31)

*   **MONTH**

月份 (1–12)

*   **DOW**

星期(0–7)，其中 0 和 7 是星期日

而这里还需要注意的是：**H**

这是一个负载参数，可以任务是作业名称的散列，理论上可以放到任意位置上，一般在**MINUTE**或者**HOUR**的前面加上`​H​`​这个参数。保证定时调度任务在系统上产生负载，这样能够使任务不会出现在同一时刻，最大限度的使用服务资源。

```markdown
# 每隔15分钟执行一次
H/15 * * * *

```

##### 进阶

除了这种固定的时间之外，Jenkins提供的定时表达式还提供了范围区间：

1.  ​`​*​`​指定所有有效值，这就不多说了，在cron中经常见到
2.  ​​`​M-N​`​指定值的范围

```markdown
# 每天1~2点，每隔15分钟执行一次
H/15 H(1-2) * * * 

```

3.  ​`​M-N/X​`​或​`​*/X​`​在指定范围或整个有效范围内按 X 的间隔步进
4.  ​`​A,B,...,Z​`​枚举多个值

> 找几个例子让大家找找感觉，也可以直接在Jenkins中进行测试

```markdown
# 在5，10,15分的时候执行
5,10,15 * * * * 

# 工作日的0点，3点，6点，每隔10分钟执行一次
H/10 0,3,6 * * 1-5 

```

#### 任务构建

接下来我们就让我们的任务定时构建一次就每隔2分钟执行一次

> Q：为啥不每隔一分钟执行一次？  
> A：大概Jenkins觉得你们太调皮了，不支持1分钟的调用

```markdown
H/2 * * * *

```

准时构建，那么在实际下根据自身需要，合理选择表达式进行构建操作

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21e9d4bf8a7a4845aa9a3d37ee040804~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

> 我这里留了一个配置，让它在零点执行：​`​H(0-1)/1 0 * * *​`​接下来拭目以待

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1a8f1e9226847049932ac4a7e7a19e5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### GitLab webhook

#### 额外说明

这里先插一句，在Jenkins构建器中关于GitLab webhook的方式

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2617cf37ae2345eb804cc4b13760e338~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

默认情况下是不会出现这一选项的，需要额外安装一个插件，虽然在之前也提到过，但是这里还是要多说一句：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bbf9329818741109230793c397158b9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

在插件管理中需要先将**GitLab Plugin**插件安装上，成功之后才能使用webhook的方式来触发构建

#### Webhooks

穿插了一个关键点之后，我们继续介绍Webhooks

> 维基百科：  
> 在web开发过程中的webhook，是一种通过通常的callback，去增加或者改变web page或者web app行为的方法。  
> 这些callback可以由第三方用户和开发者维持当前，修改，管理，而这些使用者与网站或者应用的原始开发没有关联

结合Jenkins来介绍：

*   当我们对GitLab仓库进行操作（提交代码，合并分支）之后，GitLab会向对应配置的接口中发送http请求，Jenkins接收到请求之后会对当前项目作出构建的操作。这样我们就能够实现自动化CI操作，减少部署成本

\\

#### 实操：基于GitLab的Webhooks实现自动化构建

那么接下来我们就开始配置Webhooks，实现自动化构建任务吧

##### GitLab配置：本地出站请求

首先，由于GitLab版本的升级，会限制本地的出站请求，由于本人在搭建环境的时候将Jenkins和GitLab放到了同一台服务器上，所以需要先在GitLab上对出站请求进行配置

> 如果环境在多台服务器上，这一步可以省略

当前配置在GitLab位于 ​`​Menu>Admin>Settings>Network>​`​​`​Outbound requests​`​下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8b1724ae149438dac8872d399f17565~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

勾选图中的两个选项框或者在下方的文本框中输入本地IP和域名都是可以的

> 为了方便我就勾选了

##### Jenkins 503配置

由于Jenkins开启了GitLab的身份验证，此时如果直接在GitLab上添加Webhooks的话，无法正确被处理。

这里就需要进入到Jenkins的 ​`​系统管理>系统配置​`​，找到**Gitlab**的位置，并且去掉​`​Enable authentication for '/project' end-point​`​的勾选

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e70be4a703543f0ad67f5144bb568a9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

##### Jenkins项目配置触发器

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d5fa550cdc148d6a167e7f4c5a933b4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

进入到项目配置页面中，勾选图中框住的选项，并复制最后的URL地址

##### GitLab设置Webhooks

进入到 GitLab上该项目页面，从​`​settings>Webhooks​`​找到对应的配置页面，将复制到的URL添加到指定的位置中就可以了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbcbdcebd8ea4323ad3e5ebdb2bfd51c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee19135f6dcd44fcb32b4a33c7deeec0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

此时最下方会出现一条Webhooks的记录，我们可以通过​`​Test​`​按钮进行测试，看看是否可以正常运行

##### 测试

直接推送一个事件，看看Jenkins能不能接收到

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ff7d886a423413fbe0e480c4d3a042b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

开始构建的话就说明Webhooks的方式已经构建成功。测试完成，收工。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1282b898126342bdafd67569aea45461~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

当前测试通了最后，那么通过git提交代码之后，Jenkins对项目进行构建那也是必然可以正常执行的。

最后
--

到这里关于Jenkins触发器相关的内容就介绍的差不多了。我们只是列举出几个比较常用的触发器，如果有涉及到需要使用其他触发器的可以自行查看帮助文档。