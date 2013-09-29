## Handlebars
The index of the current array item has been available for some time now via @index:

    {{#each array}}
    	{{@index}}: {{this}}
	{{/each}}
    
For object iteration, use @key instead:

	{{#each object}}
    	{{@key}}: {{this}}
	{{/each}} 

#Backbone.js
## 概念
### Backbone.js 的最佳应用场景有哪些？
http://www.zhihu.com/question/19720745

**新版的有道笔记 Web 版（http://note.youdao.com ） Backbone**。就像其他答案回答的，**Backbone 最适合的应用场景是单页面应用，并且页面上有大量数据模型，模型之间需要进行复杂的信息沟通。**Backbone 在这种场景下，能很好的实现模块间松耦合和事件驱动。 其他适用产品还有微博，网易微博的前端设计也是和 Backbone 类似的一个结构。

Backbone 的优点和一些经验 Tip：

* view 的划分将页面上的视图元素解耦，粒度细化。View 间通过事件和 Model 通讯，避免了 DOM 事件的滥用。
* model 和 Restful 的通讯方式对于后端人员非常友好。
* MVC 架构清晰， 我有个常年写 Java 没写过 JS 的同事看 Backbone 很快就了解了整体设计，虽然这时候他还是不会写 JS。
* Collection/Model 抽象了以前杂乱的 AJAX 请求，CRUD 请求变得非常非常方便。
* 强烈建议 View -> Model 单向依赖，世界会美好很多。
* 配上一个模块化加载器例如 SeaJS 会很爽。

Backbone 的一些缺点，或者说一些尚未实现的 Feature：

* Model 层比较简单，如果要支持 One-To-One 或者 One-To-Many 等复杂数据关系时有些力不从心。还有 一个 Model 只能属于一个 Collection 这个设计，页面复杂的时候会很受局限。例如这个问题： http://www.zhihu.com/question/19843899 （补充：Backbone.Relations 插件是这个问题的一个解决方案 https://github.com/PaulUithol/Backbone-relational By zjhiphop）
* 同上，Model 只有基本的 CRUD 操作，不能很好的扩展，Backbone.sync 方法写的不太灵活，要想扩展就得重写 sync 方法。
* View 层没有很强的 Page 管理机制，比如通过 URL 切换改变整个页面时，页面上尚存的 View 如何处理？直接销毁的话，是否要销毁关联的 Model、Collection？Cache 住？如何管理 Cache？
* 内存管理需要比较小心，缺乏机制避免创建重复 Model。
* extends override 父类方法的时候得写一串的 SuperClass.prototype.someMethod.apply 什么的，就不能实现个 _super 方法么……
对调试非常不友好。
* 作者有代码洁癖（也是加分项），this.$el 大家呼唤了这么久才加上，估计今生也看不到 this._super。
更新慢。

总体来说 Backbone 还很轻，框架很漂亮但是有些细节还比较粗糙。用之前要做好对 Backbone 进行大量扩展甚至 Hack 的准备。

---------
Backbone.js 的优点：

1. 代码质量比较高，通读一遍还是能学到不少东西的。
2. 只做框架该做的事情，不做高大全的东西。所以很容易和其他的工具或框架整合。比如有人搞了 Bakcbone.js + Knockout.js 的 Knockback.js。
3. 分层的结构很清晰，使得前端工程在扩展性和维护性上都可以进行有效控制。

Backbone.js 缺点:

1. Model 结构比较简单，如 @pw 所述，多对多、但对多的数据模型很难搞，用对象做属性也不行。
2. **内存控制，View 很容易产生 memory leak 的问题**，不过这也和代码的质量有关系，近期的更新有一些是针对这方面的。

**豌豆荚 PC 客户端 2.0 全部是使用 Backbone.js 框架开发的。**
