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
Backbone.js 的优点：
1. 代码质量比较高，通读一遍还是能学到不少东西的。
2. 只做框架该做的事情，不做高大全的东西。所以很容易和其他的工具或框架整合。比如有人搞了 Bakcbone.js + Knockout.js 的 Knockback.js。
3. 分层的结构很清晰，使得前端工程在扩展性和维护性上都可以进行有效控制。

Backbone.js 缺点:
1. Model 结构比较简单，如 @pw 所述，多对多、但对多的数据模型很难搞，用对象做属性也不行。
2. 内存控制，View 很容易产生 memory leak 的问题，不过这也和代码的质量有关系，近期的更新有一些是针对这方面的。