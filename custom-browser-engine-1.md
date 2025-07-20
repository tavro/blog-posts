# writing a browser from scratch: step 1 - HTML parsing

this year opera turns 30 and i've been working professionally at opera for the past six months. looking further back, i also contributed to the company as an intern for two additional years. since i'm not actively writing code for the browser itself, i became curious and wanted to understand how browsers actually work. in exploring this, i discovered that web browsers are complex and fascinating pieces of software. they touch on many areas of computer science, such as parsing, graphics, networking, security, performance optimization, etc... in this series, we'll peel back the layers and implement the core building blocks of a browser. starting with the HTML parser! HTML is the language of the web. it's the first thing a browser sees and interprets when you visit a webpage. to render anything on the screen, a browser needs to first parse the raw HTML into a structured representation, commonly known as the DOM. that's exactly what we'll do in this first step.

## minimal HTML parser

we'll implement our HTML parser in C. for now, we'll assume that the input html string contains no malformed content.

here is the full code for our first prototype:

```c
#include <stdbool.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <ctype.h>

typedef enum {
	DOCUMENT,
	ELEMENT,
	TEXT,
	COMMENT
} HTMLNodeType;

typedef struct HTMLAttribute {
	char* name;
	char* value;

	struct HTMLAttribute* next;
} HTMLAttribute;

typedef struct HTMLNode {
	HTMLNodeType type;

	char* name;
	char* content;

	HTMLAttribute* attributes;

	struct HTMLNode** children;
	size_t childCount;
	size_t childCapacity;

	struct HTMLNode* parent;
} HTMLNode;

HTMLNode* createNode(HTMLNodeType type) {
	HTMLNode* node = (HTMLNode*)malloc(sizeof(HTMLNode));

	node->type = type;
	node->name = NULL;
	node->content = NULL;
	node->attributes = NULL;
	node->children = NULL;
	node->childCount = 0;
	node->childCapacity = 0;
	node->parent = NULL;

	return node;
}

HTMLNode* createElement(const char* name) {
	HTMLNode* node = createNode(ELEMENT);

	node->name = strdup(name);
	
	return node;
}

HTMLNode* createText(const char* text) {
	HTMLNode* node = createNode(TEXT);

	node->content = strdup(text);

	return node;
}

HTMLNode* createDocument() {
	return createNode(DOCUMENT);
}

void appendChild(HTMLNode* parent, HTMLNode* child) {
	if (parent->childCapacity == 0) {
		parent->childCapacity = 4;
		parent->children = (HTMLNode**)malloc(sizeof(HTMLNode*) * parent->childCapacity);
	} else if (parent->childCount >= parent->childCapacity) {
		parent->childCapacity *= 2;
		parent->children = (HTMLNode**)realloc(parent->children, sizeof(HTMLNode*) * parent->childCapacity);
	}

	parent->children[parent->childCount++] = child;
	child->parent = parent;
}

void addAttribute(HTMLNode* node, const char* name, const char* value) {
	HTMLAttribute* attribute = (HTMLAttribute*)malloc(sizeof(HTMLAttribute));

	attribute->name = strdup(name);
	attribute->value = strdup(value);
	attribute->next = node->attributes;

	node->attributes = attribute;
}

void freeNodeTree(HTMLNode* node) {
	if (!node) return;

	free(node->name);
	free(node->content);

	HTMLAttribute* attribute = node->attributes;
	while(attribute) {
		HTMLAttribute* next = attribute->next;

		free(attribute->name);
		free(attribute->value);
		free(attribute);

		attribute = next;
	}

	for (size_t i = 0; i < node->childCount; i++) {
		freeNodeTree(node->children[i]);
	}

	free(node->children);
	free(node);
}

char* skipWhitespace(char* s) {
	while (isspace(*s)) s++;
	return s;
}

char* parseTagName(char* s, char* tagName) {
	s = skipWhitespace(s);

	int i = 0;
	while (*s && (isalnum(*s) || *s == '-' || *s == '_')) {
		tagName[i++] = *s++;
	}
	tagName[i] = '\0';

	return s;
}

char* parseAttributes(char* s, HTMLNode* node) {
	s = skipWhitespace(s);

	while (*s && *s != '>' && *s != '/') {
		char attributeName[64], attributeValue[256];

		int i = 0;
		while (*s && *s != '=' && !isspace(*s)) {
			attributeName[i++] = *s++;
		}
		attributeName[i] = '\0';

		s = skipWhitespace(s);
		if (*s == '=') {
			s++;
			s = skipWhitespace(s);
			if (*s == '"' || *s == '\'') {
				char quote = *s++;
				i = 0;
				while (*s && *s != quote) {
					attributeValue[i++] = *s++;
				}
				attributeValue[i] = '\0';

				if (*s == quote) s++;
				addAttribute(node, attributeName, attributeValue);
			}
		}

		s = skipWhitespace(s);
	}

	return s;
}

char* parseNode(char* s, HTMLNode* parent) {
	s = skipWhitespace(s);

	while (*s) {
		if (strncmp(s, "</", 2) == 0) { // end tag
			while (*s && *s != '>') s++;
			if (*s == '>') s++;
			return s;
		} else if (*s == '<') { // start tag
			s++;
			if (*s == '!') { // skip comments
				while (*s && *s != '>') s++;
				if (*s == '>') s++;
				continue;
			}

			char tagName[64];
			s = parseTagName(s, tagName);

			HTMLNode* node = createElement(tagName);
			s = parseAttributes(s, node);

			bool selfClosing = false;
			if (*s == '/') {
				selfClosing = true;
				s++;
			}
			if (*s == '>') s++;

			appendChild(parent, node);

			if (!selfClosing) {
				s = parseNode(s, node);
			}
		} else { // text
			char text[1024];
			int i = 0;
			while (*s && *s != '<') {
				text[i++] = *s++;
			}
			text[i] = '\0';

			char* trimmed = skipWhitespace(text);
			if (*trimmed != '\0') {
				HTMLNode* textNode = createText(trimmed);
				appendChild(parent, textNode);
			}
		}
	}

	return s;
}

HTMLNode* parser(char* inputString) {
	HTMLNode* document = createDocument();
	parseNode(inputString, document);
	return document;
}

void printNode(HTMLNode* node, int indent) {
	for (int i = 0; i < indent; i++) printf("	");

	switch(node->type) {
		case DOCUMENT:
			printf("Document\n");
			break;
		case ELEMENT:
			printf("Element: <%s", node->name);
			HTMLAttribute* attribute = node->attributes;
			while (attribute) {
				printf(" %s=\"%s\"", attribute->name, attribute->value);
				attribute = attribute->next;
			}
			printf(">\n");
			break;
		case TEXT:
			printf("Text: \"%s\"\n", node->content);
			break;
		case COMMENT:
			printf("Text: %s\n", node->content);
			break;
	}

	for (size_t i = 0; i < node->childCount; i++) {
		printNode(node->children[i], indent + 1);
	}
}

int main() {
	char input[] = "<div id=\"main\" class=\"container\"><p>Hello <b>world</b>!</p></div><br/>";

	HTMLNode* DOM = parser(input);
 	printNode(DOM, 0);
	freeNodeTree(DOM);

	return 0;
}
```

if you input:
```
<div id="main" class="container"><p>Hello <b>world</b>!</p></div><br/>
```

the program would output an internal representation structure looking like this:

```mathematica
Document
	Element: <div class="container" id="main">
		Element: <p>
			Text: "Hello "
			Element: <b>
				Text: "world"
			Text: "!"
	Element: <br>
```

## coming up

this is just the beginning. i am building this engine as i write these posts, to learn. but as of right now, this is part of a series on building a web browser engine in C. i've planned future posts on:

- adding css support (CSSOM)
- combining DOM and CSSOM to create rendering tree

*let me know if you'd like me to include diagrams or something to make things easier to follow by sending an email to isakhorvath@gmail.com*
