---
title: Writing a language parser using LPEG
layout: post
date: 2018-09-06 17:00:00
categories: neovim lua language LPEG
---

# Why I Used Match-Time Captures

When using LPEG, there are times where you want to do more than simply capture the result of a match. Perhaps you want to display a message about the current match during parsing or store them off for later use.

While writing my language parser for `M`, I wanted to be able to determine (within a single function) what variables were the arguments for that function. The reason for this was that I wanted to be able to display those variables differently than other variables that were created in that function only.

# How to use Match-Time Captures

To use a match-time capture in LPEG, you can use the following syntax (take from LPEG):

```
lpeg.Cmt(patt, function)
Creates a match-time capture. Unlike all other captures, this one is evaluated immediately when a match occurs (even if it is part of a larger pattern that fails later). It forces the immediate evaluation of all its nested captures and then calls function.

The given function gets as arguments the entire subject, the current position (after the match of patt), plus any capture values produced by patt.

The first value returned by function defines how the match happens. If the call returns a number, the match succeeds and the returned number becomes the new current position. (Assuming a subject s and current position i, the returned number must be in the range [i, len(s) + 1].) If the call returns true, the match succeeds without consuming any input. (So, to return true is equivalent to return i.) If the call returns false, nil, or no value, the match fails.

Any extra values returned by the function become the values produced by the capture.
```


An simple example of using this might be something like:

{% highlight lua %}

local lpeg = require('lpeg')
local to_parse = 's MYVAR=10'

print(lpeg.Cmt(to_parse, function(

{% endhighlight %}

NOTE: In the following examples, you'll see the `patterns` module used extensively, which is something that I wrote to help me to write intelligible LPEG code. You can find that [here](https://github.com/tjdevries/mparse.nvim/blob/master/lua/mparse/patterns.lua).

One of the things I wanted for my language parser was to have configuration about how it would parse certain items. For example, we have compiler directives that are of the form:

```
  ; It must be in a comment
  ; And it must be surround by two pound (#) signs.
  ;#directive#
```

However, there are only a few limited sets of directives that actually _do_ anything. I prefer to not have my directives highlighted if they won't do anything, but the current behavior for our editor is that they are always highlighted -- regardless of whether they are valid or not.

So I made an option in `parser_options` about whether it should be `strict_compiler_directives`. This resulting in the following code:

```
Here are the following 
patterns.literal = lpeg.P
patterns.matchtime_capture = lpeg.Cmt
```


{% highlight lua %}
  -- Compiler directives are only used within "mComment"
  mCompilerDirective = patterns.capture(
    patterns.concat(
      patterns.literal('#')
      , patterns.matchtime_capture(patterns.any_amount(letter), function(string, index, match)
        if not parser_options.strict_compiler_directives then
          return true
        end

        if verified_compiler_directives[match] then
          return true
        end

        return false
      end)
      , patterns.literal('#')
    )
  )
{% endhighlight %}

This code checks whether or not you have `strict_compiler_directives` on. If you do not, then it always returns `true`, even if the directive does nothing. If it is on, it checks a pre-defined list of directives and sees whether that is included.
