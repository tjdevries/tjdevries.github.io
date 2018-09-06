---
title: Writing a language parser using LPEG
layout: post
date: 2018-09-06 17:00:00
categories: neovim lua language LPEG
---

# The Problem

At work, we use a language that not many other places use -- M. An example of the language can be seen below (it is unlikely to be highlighted correctly).

{% highlight MUMPS %}

  ; This is a comment
myFunction(param1,param2) ;
  n result
  s result=param1+param2
  q resut

{% endhighlight %}

I like to use Neovim for my editing, but there were no existing syntax files for the language. I considered using some of the built-in highlighting that (Neo)Vim offers, but I wanted to try something different first. The language syntax is actually quite easy to parse (there is essentially no whitespace, everything must be on the same line, only simple arrays and functions, etc.), so I thought maybe I could write something to parse the language and then output highlighting into Neovim for me -- smarter than I could have done with simple pattern matching.

At just this time, Neovim integrated Lua into the editor as a first class configuration citizen. I was interested in learning more about Lua, so I researched available libraries for language parsing and found LPEG.


