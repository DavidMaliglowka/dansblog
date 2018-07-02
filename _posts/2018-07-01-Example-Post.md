---
layout: post
title: 'This Post Shows off all of the blog features'
date: 2018-07-01
author: David Maliglowka
cover: 'https://weeblytutorials.com/wp-content/uploads/2017/04/Weebly-blogs-example.png'
tags: example tutorial
---

This blog should allow you to do everything you need. Everything will be hosted on github and it all uses jekyll: a static site generator.

## Features
* Tags
* Pagination
* Dark/Light mode
* Search
* Commenting
* Social Share Buttons
* Syntax Highlight

Plus it's all responsive!


Easy to write posts just add a markdown file to the `_posts` folder with the following markup at the top of each page:

```javascript
---
layout: post
title: 'Title of Blog'
date: 2018-07-01 //Date created
author: Dan Larimer //Author of post
cover: '' //Link for cover image
tags: example tutorial //Tags you want to use
---
```


## You can easily add inline images:
![Example Picture](https://steemitimages.com/DQmUSLXN9TnhANEUJCCYu3Z4fGemUhLbJ4jBQ5o8vAqVssp/image.png)

```javascript
![Example Picture](https://steemitimages.com/DQmUSLXN9TnhANEUJCCYu3Z4fGemUhLbJ4jBQ5o8vAqVssp/image.png)
```

## Quote people you agree with
>The problem with these "race condition" ICOs is that it doesn't matter how good the network is--that just becomes the new race condition. Say the network can handle double the transactions--then people just cram all of their transaction attempts into half the time. Even if the network could handle a billion transactions in a single block there would still be an incentive to transaction flood. -- emanslpater


## Add Tables
```html
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
```

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

## Headers
___
```html
# H1
## H2
### H3
#### H4
##### H5
###### H6

```

# H1
## H2
### H3
#### H4
##### H5
###### H6

## Emphasis
___
```html

Emphasis, aka italics, with *asterisks* or _underscores_.

Strong emphasis, aka bold, with **asterisks** or __underscores__.

Combined emphasis with **asterisks and _underscores_**.

Strikethrough uses two tildes. ~~Scratch this.~~

```

Emphasis, aka italics, with *asterisks* or _underscores_.

Strong emphasis, aka bold, with **asterisks** or __underscores__.

Combined emphasis with **asterisks and _underscores_**.

Strikethrough uses two tildes. ~~Scratch this.~~

## Lists
___
```html

1. First ordered list item
2. Another item
⋅⋅* Unordered sub-list.
1. Actual numbers don't matter, just that it's a number
⋅⋅1. Ordered sub-list
4. And another item.

⋅⋅⋅You can have properly indented paragraphs within list items. Notice the blank line above, and the leading spaces (at least one, but we'll use three here to also align the raw Markdown).

⋅⋅⋅To have a line break without a paragraph, you will need to use two trailing spaces.⋅⋅
⋅⋅⋅Note that this line is separate, but within the same paragraph.⋅⋅
⋅⋅⋅(This is contrary to the typical GFM line break behaviour, where trailing spaces are not required.)

* Unordered list can use asterisks
- Or minuses
+ Or pluses

```

1. First ordered list item
2. Another item
  * Unordered sub-list.
1. Actual numbers don't matter, just that it's a number
  1. Ordered sub-list
4. And another item.

You can have properly indented paragraphs within list items. Notice the blank line above, and the leading spaces (at least one, but we'll use three here to also align the raw Markdown).

⋅⋅⋅To have a line break without a paragraph, you will need to use two trailing spaces.⋅⋅
⋅⋅⋅Note that this line is separate, but within the same paragraph.⋅⋅
⋅⋅⋅(This is contrary to the typical GFM line break behaviour, where trailing spaces are not required.)

* Unordered list can use asterisks
- Or minuses
+ Or pluses

## Links
___
```html

[I'm an inline-style link](https://www.google.com)

[I'm an inline-style link with title](https://www.google.com "Google's Homepage")

[I'm a reference-style link][Arbitrary case-insensitive reference text]

[I'm a relative reference to a repository file](../blob/master/LICENSE)

[You can use numbers for reference-style link definitions][1]

Or leave it empty and use the [link text itself].

URLs and URLs in angle brackets will automatically get turned into links.
http://www.example.com or <http://www.example.com> and sometimes
example.com (but not on Github, for example).

Some text to show that the reference links can follow later.

[arbitrary case-insensitive reference text]: https://www.mozilla.org
[1]: http://slashdot.org
[link text itself]: http://www.reddit.com

```

[I'm an inline-style link](https://www.google.com)

[I'm an inline-style link with title](https://www.google.com "Google's Homepage")

[I'm a reference-style link][Arbitrary case-insensitive reference text]

[I'm a relative reference to a repository file](../blob/master/LICENSE)

[You can use numbers for reference-style link definitions][1]

Or leave it empty and use the [link text itself].

URLs and URLs in angle brackets will automatically get turned into links.
http://www.example.com or <http://www.example.com> and sometimes
example.com (but not on Github, for example).

Some text to show that the reference links can follow later.

[arbitrary case-insensitive reference text]: https://www.mozilla.org
[1]: http://slashdot.org
[link text itself]: http://www.reddit.com
