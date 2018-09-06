---
title:  "Make a Blog with Markdown using NodeJS"
date:   2018-09-03
updatedDate: 2018-09-06
category: NodeJS
urlName: make-a-blog-with-markdown-using-nodejs
---

This will be a tutorial to show you how to add a blogging feature to your NodeJS application. You basically add your Markdown pages to the project (sorry, we won't add a text editor) and your NodeJS will generate your HTML blogs. This blog itself was generated through methods that you'll see in this tutorial. Your blogs will be able to support: 
* Dynamic rendering of Markdowns to HTML upon server startup.
* Metadata for each of the Markdown.
* Syntax highlighting.

You can follow this guide by going section by section but they'll be written in a way so that you can just skip to whichever part you're already on. If you want to see the code and follow along, it is right [here](https://github.com/logicxd/blog-nodejs).

### Prerequisite
We'll assume that you have:
- A NodeJS application set up.
- Basic knowledge of NodeJS and ExpressJS.

Here is the list of general tools that we'll use that you may want to add to the top of your NodeJS file:
``` js
// ./index.js

var express = require('express');
var app = express();
var fs = require('fs-extra');
var path = require('path');
```

And here is my dependencies in my `package.json`: 
``` json
{
 "dependencies": {
    "ejs": "^2.6.1",
    "express": "^4.16.3",
    "fs-extra": "^7.0.0"
  }
}
```
Remember to run `npm install` after updating your dependencies.

### Adding a Markdown Parser
We will use [markdown-it](https://github.com/markdown-it/markdown-it) to generate HTML files from your Markdown pages. It's super straight forward and you can look at that GitHub page for more details. Here's what I did.

From terminal/bash inside your NodeJS directory:
``` bash
npm install markdown-it
```

Then you can add this code snippet to your NodeJS file to render the Markdown file:
``` js
// ./index.js

var md = require('markdown-it')({
    html: true,
    linkify: true,
    typographer: true
});

var filePath = './public/example.md';
var rawMd = fs.readFileSync(filePath, 'utf8');
var html = md.render(rawMd);
```

That's all you need to start rendering Markdowns to HTML!

### Obtaining Metadata
Metadatas are extremely useful for adding additional information about your Markdown page. 
We can store data such as the title and date.
In order to add meta data to our Markdown, we can use something called [YAML](http://yaml.org). 
By using YAML, our Markdowns can add metadata at the very top like so:
``` markdown
<!-- ./public/example.md -->

---
title: "Make a Blog with Markdown using NodeJS"
date: 2018-09-03

---

This will be a tutorial to show you how to ...
```

The Markdown parser that we used, `markdown-it`, supports plugins and we can use something called [markdown-it-meta](https://github.com/CaliStyle/markdown-it-meta) to enable YAML parsing.

From terminal/bash:
``` bash
npm install markdown-it-meta
```

Now you can modify your javascript to look something like this:
``` javascript
// ./index.js

var mdMeta = require('markdown-it-meta');
var md = require('markdown-it')({
    html: true,
    linkify: true,
    typographer: true
}).use(mdMeta);

var filePath = './public/example.md';
var rawMd = fs.readFileSync(filePath, 'utf8');
var html = md.render(rawMd);
var metaData = md.meta;
var title = metaData.title;
var date = metaData.date;
```

### Code Syntax Highlighting
The current HTML output that `markdown-it` generates doesn't render the code inside your Markdown to support syntax highlighting out of the box. But don't worry, we can easily add syntax highlighting by using [Highlightjs](https://github.com/highlightjs/highlight.js).

From terminal/bash:
``` bash
npm install highlight.js
```

Updating your javascript again to look something like this:
``` javascript
// ./index.js

var mdMeta = require('markdown-it-meta');
var hljs = require('highlight.js');
var md = require('markdown-it')({
    html: true,
    linkify: true,
    typographer: true,
    highlight: function (str, lang) {
        if (lang && hljs.getLanguage(lang)) {
            try {
                return '<pre class="hljs"><code>' +
                    hljs.highlight(lang, str, true).value +
                    '</code></pre>';
            } catch (__) {}
        }

        return '<pre class="hljs"><code>' + md.utils.escapeHtml(str) + '</code></pre>';
    }
}).use(mdMeta);

var filePath = './public/example.md';
var rawMd = fs.readFileSync(filePath, 'utf8');
var html = md.render(rawMd);
var metaData = md.meta;
var title = metaData.title;
var date = metaData.date;
```
The `highlight` function is actually taken directly from `markdown-it`'s [GitHub page](https://github.com/markdown-it/markdown-it#syntax-highlighting). 
Now our HTML output is all ready to go to start highlighting with any CSS. 
If you want to use an existing syntax highlighting, you can download from [Highlight's CSS download page](https://highlightjs.org/download/). They'll be in the folder `styles` right when you open them. Pick any that you like and then use them in your blog page's CSS file and they'll highlight your code!

### Demo/Example
[Here](https://github.com/logicxd/blog-nodejs) is an example NodeJS project for this blog. The major files are also posted below. 

``` javascript
// ./index.js

const PORT = process.env.PORT || 8082;

var express = require('express');
var app = express();
var fs = require('fs-extra');
var path = require('path');
var mdMeta = require('markdown-it-meta');
var hljs = require('highlight.js');
var md = require('markdown-it')({
    html: true,
    linkify: true,
    typographer: true,
    highlight: function (str, lang) {
        if (lang && hljs.getLanguage(lang)) {
            try {
                return '<pre class="hljs"><code>' +
                    hljs.highlight(lang, str, true).value +
                    '</code></pre>';
            } catch (__) { }
        }
        return '<pre class="hljs"><code>' + md.utils.escapeHtml(str) + '</code></pre>';
    }
}).use(mdMeta);

// Start server
var server = app.listen(PORT, function() {
    console.log('Server has started on port ' + PORT);
}); 

app.set('view engine', 'ejs');
app.use(express.static(path.join(__dirname, 'public')));

app.get('/', function(req, res) {
    var filePath = './public/example.md';   // Set this to your markdown file.
    var css = 'atom-one-dark.css';          // Set this to your markdown code syntax highlighting.

    var rawMd = fs.readFileSync(filePath, 'utf8');
    var html = md.render(rawMd);
    var metaData = md.meta;
    var title = metaData.title;
    var date = new Date(metaData.date).toDateString();

    res.render('blog', {
        title: title,
        date: date,
        html: html,
        css: css
    });
});

module.exports = app; 
```

``` html
<!-- ./views/blog.ejs -->

<!DOCTYPE html> 
<html>
<head>
    <link rel="stylesheet" href="<%= css %>"/>
</head>
<body class="container">
    <h1><%= title %></h1>
    <div><b><%= date %></b></div>
    <%- html %>
</body>
</html> 
```

### Problems/Issues

<h5><b>How to add an editor to make these Markdown pages?</b></h5>

This tutorial does not have a text editor for Markdowns - I didn't plan on using an editor so I haven't looked too much into it :(.
You can try looking into [StackEdit](https://stackedit.io/app) and see if that helps!

<h5><b>My Markdown pages are not updating!</b></h5>

The way it is currently set up, the Markdown pages will only be rendered from Markdown to HTML when you first start your NodeJS application. So you will need to restart your NodeJS to see your latest changes.

<h5><b>How can we save the rendered HTML as static pages?</b></h5>

From our example, you can get the HTMl generated from Markdown by calling `md.render(rawMd)`. After that, you can use NodeJS's file output operations from `fs` to store them as you like. Here's a [link](https://nodejs.org/api/fs.html#fs_fs_writefile_file_data_options_callback) to point you to a direction.



