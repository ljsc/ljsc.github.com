---
layout: post
title: "Ruby Methods &amp; Multiple Blocks"
date: 2011-08-06 19:35
comments: true
categories: ruby
---

# Higher order methods?

One of the really nice things about working with Ruby has always been how easy
it is to get your methods to accept a functional parameter via the block syntax.
It's no wonder that folks new to Ruby often enjoy the use of the `Enumerable`
mixin so much: most all of the methods contained therein leverage blocks to make
the code concise and readable. For instance,

{% codeblock Simple Block lang:ruby %}
fruit = [:apple, :orange, :pineapple]
fruit.each do |f|
  puts "I like #{f}!"
end
{% endcodeblock %}

is so much nicer than the equivalent that lower level constructs like C's `for`
loops. Taking a block parameter let's us easily configure and generalize the
behavior of our methods. What happens though when we have more than one bit of
functionality that we want to parameterize?

# Stand alone `Proc`s

Unfortunately, we won't get any syntactic support from the language here. If we
want to accept more than one block, our method needs to accept the additional
blocks in the same way as any other value. So we need to do something like this:

{% codeblock Higher Order Method lang:ruby %}
def higher_order(f2)
  yield :x
  f2.call(:y)
end

higher_order(lambda { |x| puts "Invoke Proc 2 -- #{x}" }) do |x|
  puts "Invoke Proc 1 -- #{x}"
end
{% endcodeblock %}

Needless to say, this code looks like it's been beaten with the ugly stick. The
`Proc` created with `lambda` could be assigned to a local variable, but it ends
up still looking pretty nasty compared to methods that take only a single block.

Last week on the [Thoughtbot][giant-robots] blog, there was a post about [dealing
with `nil` objects][nils]. That article discusses some of the problems with
having methods return `nil` as a value since it requires the caller to remember
to explicitly check for nil when it expects a domain object.

  [giant-robots]: http://robots.thoughtbot.com/ "Giant Robots Smashing into other Giant Robots"
  [nils]: http://robots.thoughtbot.com/post/8181879506/if-you-gaze-into-nil-nil-gazes-also-into-you "If you gaze into nil, nil gazes also into you"

One way to solve the problem is to have a method that requires two callbacks.
One is called when there is an actual value to return, the other when an object
can't be found. Obviously the problem with this is that it would be ugly as
discussed above. Probably the best we could do is a method as follows. It
accepts a block which is passed an object which records the two callbacks by
responding to methods by storing the blocks passed individually to each.

{% codeblock Multiple Blocks! Sorta lang:ruby%}
may_return_nil do |when|
  when.one {|obj| puts "Got an object, yeay!: #{obj}" }
  when.none { puts "Got nadda!" }
end
{% endcodeblock %}

What do you think? What's the best way to handle a method that needs to accept
many blocks? Should ruby have a built in way to handle this?
