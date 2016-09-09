---
title: Interacting with C from Python (Part 1)
layout: post
date: 2016-09-08 22:00:00
categories: python C parsing
---

# The Problem

System A is a system that is written in C. It uses CAN or some other byte based protocol to communicate to an interface. System B is a test system written to verify that certain functionality exists in System A.

There are two main cases we want to examine:

  1. C Enums
  2. C Defines

In my case, System A was a control board and System B was a graphics board that displayed the current information and was able to send commands to the control board (i.e. start x, stop y). As is typically the case in a system, there are enums defined in C that are used throughout the project to help readability of the code. For example:

{% highlight C %}
// C-file: Protocol Spec 
typedef enum my_example_s {
  name_1 = 0x0,
  name_2 = 0x1,
  random_name = 0x25,
  final_enum = 0x27
} my_example_t
{% endhighlight %}

Here we have enum `my_example_t`. When System A (we'll call it the control board from now on) tries to send a message to the graphics board, it sends some CAN message with a specified protocol. Normally in Python, to figure out what each item is, we'd just define these same objects in some python file like so:

{% highlight python %}
name_1 = 0x0
name_2 = 0x1,
random_name = 0x25,
final_enum = 0x27
{% endhighlight %}

This is fine until someone changes the spec, either with a new enum, or a new value. 

So what I did was create what I call the `enum_parser`.

You now do something like

{% highlight python %}
my_example_t_dictionary = get_enum_list(filename, search_string)
{% endhighlight %}

Now to be fair, this does assume that you'll do something similar to the style above in C, as it parses the actual C file itself. But now you can do something like:

{% highlight python %}
print(my_example_t_dictionary['name_1'])  # 0x0
assert my_can_message.data[0] == my_example_t_dictionary['final_enum']
{% endhighlight %}

You can find the gist for this here: 

TODO: Turn into small Python library?
