---
title: A First Serverless Blog Post 
date: 2019-06-01 13:05:43
updated: 2019-06-02 10:41:43 
tags:
---

## First Try 

If this is displayed; the pipeline should be flowing well. 
<!-- more -->
### This blog is : 
* Completely serverless and scalable, it should virtually be able to hold the reddit frontpage trafic 
* Under version control; content included 
* Using CI pipeline deployement  

[Welcome on Delsoir.com](http://www.delsoir.com)

### Next steps : 
* add ssl support 
* use a CDN distribution
* PWA integration
* Add dynamic functions with Lambda
* theming 


### Layout Tests 
	
#### HTML
{% codeblock lang:html %}
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">

<head>
	<title>untitled</title>
	<meta http-equiv="content-type" content="text/html;charset=utf-8" />
	<meta name="generator" content="Geany 1.27" />
</head>

<body>
	
</body>

</html>

{% endcodeblock %}


####  PHP
{% codeblock lang:php %}
<?php 
	echo "coucouju";
        while(true){
		echo "ce code est stupide";
	}
        echo "coucouju";
        while(true){
                echo "ce code est stupide";
        }
        echo "coucouju";
        while(true){
                echo "ce code est stupide";
        }


?>
{% endcodeblock %}

#### Javascript
{% codeblock lang:javascript %}
x = 10;
y = 5;
alert(x  y);
 age = Number(age);
if (isNaN(age)) {
  voteable = "Input is not a number";
} else {
  voteable = (age < 18) ? "Too young" : "Old enough";
}
var voteable = (age < 18) ? "Too young":"Old enough"; 
{% endcodeblock %}


#### Links 
[I'm an inline-style link](https://www.google.com)

[I'm an inline-style link with title](https://www.google.com "Google's Homepage")

[I'm a reference-style link][Arbitrary case-insensitive reference text]

[I'm a relative reference to a repository file](../blob/master/LICENSE)
[You can use numbers for reference-style link definitions][1]

[arbitrary case-insensitive reference text]: https://www.mozilla.org
[1]: http://slashdot.org
[link text itself]: http://www.reddit.com

#### Images 

Here's our logo (hover to see the title text):

Inline-style: 
![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

Reference-style: 
![alt text][logo]

[logo]: https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 2"


{% fancybox https://i.imgur.com/EOA76wD.png https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Un titre pour ce logo CloudJ" %}

#### Embeds

##### Imgur Gallery
{% oembed https://imgur.com/gallery/UXpqVsh %}
##### Youtube 
{% oembed https://youtu.be/aVtO8uQrPXc %}

##### Twitter
{% oembed https://twitter.com/PlayStation/status/1134145873733009409 %}
{% oembed https://twitter.com/hashtag/belgique %}
