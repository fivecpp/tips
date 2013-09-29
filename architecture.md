## A New Post

Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.

### Basecamp Next
Basecamp Next is running Rails 3.2-stable and we’ve got a good splash of client-side MVC in the few areas where that makes sense through **Backbone.js** and various tailor-made setups.

just over 5,000 lines of **CoffeeScript**

## Handlebars
The index of the current array item has been available for some time now via @index:

    {{#each array}}
    	{{@index}}: {{this}}
	{{/each}}
    
For object iteration, use @key instead:

	{{#each object}}
    	{{@key}}: {{this}}
	{{/each}} 

