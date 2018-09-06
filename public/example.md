---
title:  "Make a Blog with Markdown using NodeJS"
date:   2018-09-03
category: NodeJS
urlName: make-a-blog-with-markdown-using-nodejs
---

This will be an easy-to-follow tutorial to show you how to add a blogging feature to your NodeJS website. This blog itself was generated through methods that you'll see in this tutorial. Your blogs will be able to support: 
* Dynamic rendering of Markdowns to HTML upon server load. (Statically generated HTMLs can be done easily once you understand how it works).
* Metadata for each of the Markdown.
* Syntax highlighting.

You can follow this guide by going section by section but they'll be written in a way so that you can just skip to whichever part you're already on.

### Prerequisite
We'll assume that you have:
- A NodeJS application set up.
- Basic knowledge of NodeJS.

Here is the list of require at the top of the NodeJS file:
``` js
var express = require('express');
var app = express();
var fs = require('fs-extra');
var path = require('path');
```

And here is my dependencies in my package.json that I might use.
``` json
{
 "dependencies": {
    "express": "^4.16.3",
    "fs-extra": "^7.0.0",
  }
}
```
Remember to run `npm install` after updating your dependencies.

### Adding a Markdown Parser
We will use [markdown-it](https://github.com/markdown-it/markdown-it) to generate HTML files from your Markdown pages. It's super straight forward, you can look at the GitHub page itself instead if you wish. Here's what I did.

From terminal/bash:
``` bash
npm install markdown-it
```

Then you can add this code snippet to render the markdown file:
``` js
var md = require('markdown-it')({
    html: true,
    linkify: true,
    typographer: true
});

var filePath = './public/example.md';
var rawMd = fs.readFileSync(filePath, 'utf8');
var html = md.render(rawMd);
```

That's all you really need for the simplest functionality.

### Obtaining Metadata
Metadatas are extremely useful for adding additional information about your markdown page, such as the title and date.
In order to add meta data to our markdown, we can use something called [YAML](http://yaml.org). 
By using YAML, our markdowns can have add metadata at the very top like so:
``` markdown
---
title: "Make a Blog with Markdown using NodeJS"
date: 2018-09-03
---

This will be an easy-to-follow tutorial to show you how to ...
```

The markdown parser, `markdown-it`, supports plugins and we can use something called [markdown-it-meta](https://github.com/CaliStyle/markdown-it-meta).

From terminal/bash:
``` bash
npm install markdown-it-meta
```

Then modify the previous code to use it:
```
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
The current HTML output that `markdown-it` generates doesn't render the code syntax highlighting differently than others so we need to use another add-on to add it.

### (Misc.) My Setup