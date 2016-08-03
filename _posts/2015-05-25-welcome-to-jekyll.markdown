---
layout: post
title:  "Welcome to Jekyll!"
date:   2015-05-25 15:09:23
categories: tech life
---
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Or quickly run the following cmd to auto-generate a new post:

```bash
$ ./jekyll-page "new post example" tech
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	_pages/
	_posts/2016-08-01-new-post-example.md

$ cat _posts/2016-08-01-new-post-example.md 
---
layout: page
title: "new post example"
category: tech
date: 2016-08-01 10:58:10
---
```

Jekyll also offers powerful support for code snippets:

	def print_hi(name)
	  puts "Hi, #{name}"
	end
	print_hi('Tom')
	#=> prints 'Hi, Tom' to STDOUT.

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll’s dedicated Help repository][jekyll-help].

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
