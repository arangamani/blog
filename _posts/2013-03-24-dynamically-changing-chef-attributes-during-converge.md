---
layout: post
category : tech
tagline: "Hack the resource collection"
tags : [chef]
---
{% include JB/setup %}

<link href="/blog/assets/css/syntax.css" rel="stylesheet" type="text/css"/>

## Background

In Chef, when a resource is defined all its attributes are evaluated during
compile time and the execution of the resource takes place in converge phase.
So if the value of a particular attribute is changed in converge
(and not in compile) the resource will be executed with the old value.

_Note:_ In Chef 11, there is a feature coming up called
[lazy](http://tickets.opscode.com/browse/CHEF-1814) evaluator which will allow
you to evaluate an attribute during converge instead of compile phase.


_Update:_ This feature is implemented and the documentation can be found
[here](http://docs.opscode.com/resource_common.html#lazy-attribute-evaluation).


## Example problem:
Lets consider this situation where there are two steps involved in a recipe.
Step 1 is a Ruby block that changes a node attribute. Rubyblocks get executed
in converge phase. Step 2 is a Chef resource that makes use of the node
attribute that was changed in Step 1

### Without any modification to the code

{% highlight ruby %}
node[:test][:content] = "old content"

# Step 1
ruby_block "step1" do
  block do
    node[:test][:content] = "new content"
  end
end

# Step 2
file "/tmp/some_file" do
  owner "root"
  group "root"
  content node[:test][:content]
end

{% endhighlight %}

The file resource will still have the old content as it is set in the compile
phase.

### With hacked code

{% highlight ruby %}
node[:test][:content] = "old content"

# Step 1
ruby_block "step1" do
  block do
    node[:test][:content] = "new_content"

    # Dynamically set the file resource attribute
    # Obtain the desired resource from resource_collection
    file_r = run_context.resource_collection.find(:file => "/tmp/some_file")
    # Update the content attribute
    file_r.content node[:test][:content]
  end
end

# Step 2
file "/tmp/some_file" do
  owner "root"
  group "root"
  content node[:test][:content]
end

{% endhighlight %}

The file resource will now have the updated content. This may not be a cleaner
way to do things. There might be other workarounds which will avoid hacking the
resource collection like this. I had a situation where I had to dig in and find
a way to dynamically change the resource attribute. The lazy evaluator will
make things simpler. For people who is not ready to adopt Chef 11 yet, this
method of hacking will provide a workaround.
