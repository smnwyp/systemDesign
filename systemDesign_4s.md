1. 4S
	1. scenario
		1. 系统的volume对于系统的构建有质的影响
		2. KPI
			1. DAU  
			2. given DAU=100m, 注册，登录，信息修改QPS约：
				- 100M*0.1/86400 ~ 100
				- 0.1=平均每个用户每天登陆+注册+信息修改
				- Peak = 100*3 = 300
			3. 查询的QPS约为：
				- 100M*100/86400 ~ 100k
				- 100 = 平均每个用户每天查询与用户信息相关的操作次数（查看好友、发信息，更新消息主页等）
				- Peak = 100k*3 = 300K
		- exmaple:
			- The system should be scalable to 300M active users.
			- Users can have millions of followers.
			- Assume 6k new tweet requests per second (writes) and 600k requests made by users for fetching their timeline each second (read) on average.
			- A user can see their customized News Feed in real time with minimum latency of no more than 2 seconds.
			- User’s posts should be available to millions of followers with low latency of no more than 5 seconds.
			- Up to 800 feeds can appear on the user’s timeline.
	2. service
		1. 拆分拆分拆分
	3. storage
		1. 非常非常重要
			1. 程序= 算法+数据结构
			2. 系统=服务+数据储存
		2. 几类：
			1. file system
			2. sql
			3. nosql
		3. 考虑点：
			1. 不考虑如何少存数据
			2. 只考虑如何improve query效率
	4. scale
		1. newsfeed impl
			1. push vs. pull
				1. 策略
					1. pull
						1. 每次用户看自己feed时 去pull关注者的tweet 然后用kway merge sort
						2. 问题：
							1. 当关注很多人时 db read很费时 用户需要等
						3. 优点：
							1. 只计算需要的人（发起请求的用户）
					2. push
						1. 每次用户发帖 自动插入到follower的feeds里
						2. 问题：
							1. 大v。。。可能不能及时覆盖
							2. 浪费push到僵尸粉
				2. bottleneck在IO
				3. social app一开始是push 慢慢都变成pull （fanout慢）
				4. pull比较难写啦。。
				5. volume决定：
					1. 小volume：push（不用考虑明星效应 没啥事实要求）
					2. 大volume：pull （实时 单向好友）
				6. 可以拓展pull模型
					1. cache是个好方法 read cache >> read db
					2. cache 用户自己的timeline （发帖记录）
						1. 30T的cache对于大的公司。。。毛毛雨
						2. 读db->读cache 快很多好不好
					3. cache用户的newsfeed
						1. 借用push的fanout table 可以优化做k-way merge时加时间戳嘛
						2. “将关注对象中的明星用户的 Timeline 与自己的 News Feed 进行合并后展示 ”
					4. 
				7. 可以拓展push模型
					1. 如何解决大v fanout效率差的问题？
						1. horizontal scale？
						2. 对大v使用pull 小v push push好了呀
				8. push + pull
					1. 
2. refs
	1. thundering herd
		1. 分布式系统的Thundering Herd效应 ://zhuanlan.zhihu.com/p/65843741
		2. 缓存踩踏：Facebook 史上最严重的宕机事件分析 ://1o24bbs.com/t/facebook/26691
		3. Under the hood: Broadcasting live video to millions ://engineering.fb.com/2015/12/03/ios/under-the-hood-broadcasting-live-video-to-millions/
		4. ins Thundering Herds & Promises https://instagram-engineering.com/thundering-herds-promises-82191c8af57d
			1. Instagram处理方法是不cache实际的数据，而是cache promise。这样大量重复的request过来的时候有cache miss，第一个request会创建一个promise放到cache里，这个promise再偷偷地去backend获取数据。请他的大量请求就不会是cache miss，他们都会在同一个promise下等待。这个做法和上面那个直播用queuing的做法有异曲同工之妙，都是尽可能避免后续的重复request对backend造成冲击。
	2. cache design
		1. Redis架构之防雪崩设计：网站不宕机背后的兵法 ://www.jianshu.com/p/5c6f3ec161f1
		2. Scaling Memcache at Facebook https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf
		3. How a Cache Stampede Caused One of Facebook’s Biggest Outages https://betterprogramming.pub/how-a-cache-stampede-caused-one-of-facebooks-biggest-outages-dbb964ffc8ed
	3. system design:
		1. System Design — Twitter Search  https://mecha-mind.medium.com/system-design-twitter-search-ccb29c48d9b6
		2. System Design Interview: Twitter or Facebook Feed ://medium.com/double-pointer/system-design-interview-twitter-or-facebook-news-feed-f960f6d1fd70
		3. Facebook/Twitter News Feed ://systemdesignnotes.com/fb-twtr-news-feed-design
	4. 九章
		1. https://marian5211.github.io/2018/01/30/%E3%80%90%E4%B9%9D%E7%AB%A0%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1%E3%80%91%E4%BB%8E%E7%94%A8%E6%88%B7%E7%B3%BB%E7%BB%9F%E7%90%86%E8%A7%A3%E6%95%B0%E6%8D%AE%E5%BA%93%E5%92%8C%E7%BC%93%E5%AD%98/
	5. 面试
		1. 不瞒你说，关于准备系统设计题，我有一个高效的“作弊”方法  https://posts.careerengine.us/p/61da69702eba422eb4f645b6
			1. 如果你面的是微博，重点掌握：
			2. 新鲜事系统（好友动态）即时通讯系统 （私信）搜索建议系统（搜索）缓存系统评论系统
