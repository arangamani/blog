---
layout: post
category : tech
tagline: ""
tags : [chef]
---
{% include JB/setup %}

<link href="/blog/assets/css/syntax.css" rel="stylesheet" type="text/css"/>

## Chef::Taste - What is it?

[Chef Taste](https://github.com/arangamani/chef-taste) is a simple command line utility
to check a cookbook's dependency status. It will list the dependent cookbooks in a tabular
format with the version information, status, and the changelog (if possible) for out-of-date cookbooks.

## Inspiration

The [Gemnasium](https://gemnasium.com) project does a similar job for checking a Ruby
project's dependencies and keep them up-to-date. Chef-Taste is similar to that instead
it provides a simple command line utility to list the dependency status for Chef cookbooks.


## What it does?

When you are inside the cookbooks directory, simply type `taste` to taste the cookbook.
The `metadata.rb` of the cookbook is parsed to obtain the dependencies. It will display
a table that contains the following rows:

* `Name` - The name of the cookbook
* `Requirement` - The version requirement specified in the metadata
* `Used` - The final version used based on the requirement constraint
* `Latest` - The latest version available in the community site
* `Status` - The status of the cookbook: up-to-date (a green tick mark) or out-of-date (a red x mark)
* `Changelog` - The changelog of out-of-date cookbooks if available.

The overall status will also be displayed in the bottom of the table.

### Changelog

Most of the cookbooks are hosted in Github and are tagged for every release.
The changelog is computed by obtaining the source URL provided in the community site and
finding the tags being used and the latest tag and displaying a compare view that
compares these two tags. This URL is then shortened using goo.gl URL shortener to fit the table.

The details are obtained only for cookbooks available in the community site. Other cookbooks are
displayed but will simply have `N/A` in their details.

### Examples

These examples are based on the cookbooks available in the `test/cookbooks` directory
in this repository.

#### 1. fried_rice cookbook

{% highlight bash %}
kannanmanickam@mac fried_rice$ taste
+------------+-------------+--------+--------+--------+----------------------+
| Name       | Requirement | Used   | Latest | Status | Changelog            |
+------------+-------------+--------+--------+--------+----------------------+
| ntp        | ~> 1.4.0    | 1.4.0  | 1.5.0  |   ✖    | http://goo.gl/qsfgwA |
| swap       | = 0.3.5     | 0.3.5  | 0.3.6  |   ✖    | http://goo.gl/vZtUQJ |
| windows    | >= 0.0.0    | 1.11.0 | 1.11.0 |   ✔    |                      |
| awesome_cb | >= 0.0.0    | N/A    | N/A    |  N/A   |                      |
+------------+-------------+--------+--------+--------+----------------------+
Status: out-of-date ( ✖ )
{% endhighlight %}

#### 2. noodles cookbook

{% highlight bash %}
kannanmanickam@mac noodles$ taste
+---------+-------------+--------+--------+--------+----------------------+
| Name    | Requirement | Used   | Latest | Status | Changelog            |
+---------+-------------+--------+--------+--------+----------------------+
| mysql   | ~> 3.0.12   | 3.0.12 | 3.0.12 |   ✔    |                      |
| apache2 | ~> 1.7.0    | 1.7.0  | 1.8.4  |   ✖    | http://goo.gl/9ejcpi |
| windows | >= 0.0.0    | 1.11.0 | 1.11.0 |   ✔    |                      |
+---------+-------------+--------+--------+--------+----------------------+
Status: out-of-date ( ✖ )
{% endhighlight %}

#### 3. curry cookbook

{% highlight bash %}
kannanmanickam@mac curry$ taste
+-----------------+-------------+-------+--------+--------+-----------+
| Name            | Requirement | Used  | Latest | Status | Changelog |
+-----------------+-------------+-------+--------+--------+-----------+
| ntp             | >= 0.0.0    | 1.5.0 | 1.5.0  |   ✔    |           |
| lvm             | >= 0.0.0    | 1.0.0 | 1.0.0  |   ✔    |           |
| application     | >= 0.0.0    | 4.1.0 | 4.1.0  |   ✔    |           |
| application_php | >= 0.0.0    | 2.0.0 | 2.0.0  |   ✔    |           |
+-----------------+-------------+-------+--------+--------+-----------+
Status: up-to-date ( ✔ )
{% endhighlight %}

#### 4. water cookbook

{% highlight bash %}
kannanmanickam@mac water$ taste
No dependent cookbooks
{% endhighlight %}
