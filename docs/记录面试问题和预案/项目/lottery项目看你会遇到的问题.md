# lottery项目看你会遇到的问题

###  

小傅哥，我把抽奖系统包装到实习经历里了。今天面试的那个部门是做广告架构的，可能有广告投放的场景，所以对我简历上的规则引擎量化人群参与活动模块比较感兴趣，让我说细节上的东西，然后我答得不是很好，想请教下您。

1. 问：抽奖的奖品是优惠券，那金额多大呢？候选集有多大？多少人参与？
    
    答：我说几十到几百不等，候选集瞎说的有几千个，参与者上万。（傅哥能给个大概数据吗）

2. 问：你说你用的决策树，那决策树是什么时候创建的呢？是每个用户参与抽奖就创建一次（个性化），还是一开始就创建好？

&nbsp; &nbsp; 答：一开始就创建好，每个用户根据制定好的规则筛选。

3. 问：筛选的标签是什么，根据什么来过滤呢？

&nbsp; &nbsp; 答：根据性别、年龄、首单消费、消费金额、忠实用户等各类身份标签。

4. 问：那有没有可能你制定的这些标签，数据传进来的时候是丢失的，用户没有某个路径上的数据，是不是就到不了叶子节点了，按你说的就没法领取活动了？

&nbsp; &nbsp; 答：这个问题没想过，猜想的解决方案：如果到不了的话就放到待选集，让用户等待处理？（瞎说的）面试官对我的回答不满意，他说你既然这个问题没解决，那我认为你们这个系统实际上是不可用状态......

5. 问：既然你的这些规则都是确定的，为什么要用决策树？决策树和布尔检索有什么区别知道吗？

&nbsp; &nbsp; 答：不知道咋回答了，已经被问懵了

综合下来，面试官应该是想看我有没有针对广告投放场景的量化人群过滤的经验，这种场景可能是要从上亿的候选集种选出真正适合的进行推送，实现变现效率最大的投放。所以我在想我们的这个规则引擎量化人群组件能拓展到别的业务业务场景吗，比如这个？

另外一个问题是小傅哥关于如何设计一个短链系统有什么方案吗，面试被问到了。谢谢小傅哥！


回答：

问题1：参考数据；

- 1.1 参与人群中低高(超过5千非常高、超过1万特别高、超过10万BAT级别)；1000-1500、1500-3000、3000-5000

- 1.2 抽奖的奖品可以说包括；虚拟奖品、实物奖品；虚拟奖品包括优惠券、积分、兑换码等，实物奖品是物流发货，如果内部有商城或者第三方的可以说是对接使用。

- 1.3 奖品梳理不一定，根据运营活动推广策略来设定，比如；几个奖品、几十个奖品、几百个奖品，都有。

问题2；

- 2.1 第一种基于系统启动时，基于初始任务，对存量有效决策树开始分批创建，另外一种是基于新增决策树生效审批前进行内存加载创建。

- 2.2 不需要给每个用户都创建，每个用户的A/BTest都是在一个活动下的统一规则过滤，不同的是每个用户身上的标签。PS：标签来自于对用户行为数据的采集和分析，如点击、搜索、预览、身份、喜好、状态等。

问题3；

性别、年龄、首单、消费、忠诚度、以及点击、搜索、预览、身份、喜好、状态等生成的标签。这些数据主要根据运营活动策略的配置而进行开发和使用。

问题4；

设想是有问题的，任何一款决策树在没有标签的数据的前提下，都会影响最终的决策结果。就像风控模型，我本来判断你是黑产，但标签丢失，我就没法准确判断。所以这种情况通常是进行兜底，风险不可评估，走兜底策略决策结果。

问题5；

规则是确定的还有可能新开发的，但每个玩法活动应对的场景不同，比如我们专门针对拉新的活动，首先看的目标是未注册、大学刚毕业、有某类场景喜好，那么这些则作为决策树的主要节点配置使用。而另外一个活动是促活，那么标签也会随之改变。所以决策树规则引擎的目的是尽可能简化通用场景的开发成本，提高研发能效，快速响应业务需求。PS：哈哈哈，如果是我想怼回去，你们公司啥都是高代码实现？缺少研发思考🤔，做任何需求都是一锤子买卖？嘿嘿，不过这个不要回答，也不要激动。

问题6；

关于规则引擎的拓展，传送门；https://t.zsxq.com/05zRvbUJ2

问题7；

短连接在API网关上其实非常容易实现，设定好映射规则，实现好解密流程，即可把请求的短链转发到对应的URI上。

### 

真实面试场景被问到的抽奖系统问题，请教傅哥。
问:量化规则引擎是一个组件，如果有一个新的业务进来，如何复用? 它的复用性体现在哪?能否支持风控可A/Btest需求?

你可以告诉面试官；
1. 量化规则引擎的组件设计，本身就是一个独立的领域服务。但什么最开始没有给按照一个单独的系统它拆成独立的微服务呢？因为我们目前的实际业务场景体量还没有那么大，不太适合扩大维护成本，但又考虑将来可能会有其他业务也需要同类的模块，所以以独立的领域进行设计，哪怕以后真的有更大的场景需要使用，也能更加方便的拆分出去独立部署。
2. 它的复用性体现在，它是将同类业务场景中的共性需求，凝练成通用的业务组件，而不是把业务逻辑和功能性服务捆绑，所以在量化规则的库表中配置上相应的渠道和要决策的子节点，再通过子节点的决策树配置，就可以被新的业务场景使用。因为它是规则引擎结构设计，所以灵活性很好，复用性高。
3. A/B Test 本身就可以做为决策树中的节点果实来配置，对AB用户发放不同的策略结果。所以结合风控提供的数据，作为一个逻辑节点使用也是可以的。并且换个节点可以配置到不同的决策树中。
4. 另外决策树也可以进行连接，也就是扩展决策树的果实类型，不是发放结果，而是发放下一个决策树的ID，那么这样还可以把风控作为一个单独的共用决策树使用。其他需要使用风控模型的决策树，配置上即可。


### 

1. 实习生这个阶段大多数面试的内容并不会太难，主要以考察在校期间参与的技术学习程度、技术类比赛以及个人的实践经历。如果你的大学学习生活丰富的话，以及参加了很多实践或者自己有技术博客、GitHub、开源项目经历等，都是非常好的考察内容，拿一份实习工作还是挺容易的。
2. 那么自我介绍可以按照一个结构说；面试官您好，我叫Xxx，来自于某某211/985大学，在校期间成绩优异，多次获得奖学金，也担任过学生会某某职位。在岗期间负责沟通协调组织等工作，并2次带领小组，参与过ACM大学生程序设计竞赛，获得过什么成绩，得到某某学校里的老师或者计算机的指导和认可。个人喜欢技术、热爱编程、有着较多的编码经验，也善于沟通协调工作，希望能应聘到贵公司，感谢(各位)面试官。PS：尽量把你最有价值和亮点的内容，有数字的表达出来，哪方面做的优秀就组织哪方面的语言。

扩展：