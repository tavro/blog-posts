# writing a browser from scratch: step 1 - HTML parsing

this year opera turns 30 and i've been working professionally at opera for the past six months. looking further back, i also contributed to the company as an intern for two additional years. since i'm not actively writing code for the browser itself, i became curious and wanted to understand how browsers actually work. in exploring this, i discovered that web browsers are complex and fascinating pieces of software. they touch on many areas of computer science, such as parsing, graphics, networking, security, performance optimization, etc... in this series, we'll peel back the layers and implement the core building blocks of a browser. starting with the HTML parser! HTML is the language of the web. it's the first thing a browser sees and interprets when you visit a webpage. to render anything on the screen, a browser needs to first parse the raw HTML into a structured representation, commonly known as the DOM. that's exactly what we'll do in this first step.

## minimal HTML parser

to keep things simple, we'll implement our HTML parser in C. we'll make a lot of assumptions to get started:

- the input is a single HTML element with text content
- no nested tags
- no attributes
- no malformed input

here is the [full code for our first prototype](https://github.com/tavro/bem-src/blob/main/parser/html-parser.c)

this is a simple parser that can read HTML strings like:

```
<p>This is a test string</p>
```

if you input this, the program would output an internal representation structure:

```mathematica
Document
	Element: <p>
		Text: "This is a test string"
```

## coming up

this is just the beginning. our parser has many limitations, and i am building this engine as i write these posts, to learn. but as of right now, this is part of a series on building a web browser engine in C. i've planned future posts on:

- improving the parser
- adding css support
- rendering to a window

*let me know if you'd like me to include diagrams or something to make things easier to follow by sending an email to isakhorvath@gmail.com*
