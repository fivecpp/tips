## Handlebars
The index of the current array item has been available for some time now via @index:

    {{#each array}}
    	{{@index}}: {{this}}
	{{/each}}
    
For object iteration, use @key instead:

	{{#each object}}
    	{{@key}}: {{this}}
	{{/each}} 
