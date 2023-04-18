---
layout: post
title: My First Ten Minutes with Sourcegraph Cody
date: 2023-04-18 18:36 +0000
---
> Give me the Jekyll command to create a draft blog post called 'My First Ten Minutes with Sourcegraph Cody'

[Sourcegraph Cody](https://about.sourcegraph.com/cody) then responded with the exact command I needed to run *as well as additional context*.

```
The Jekyll command to create a draft blog post called "My First Ten Minutes with Sourcegraph Cody" would be:

jekyll draft "My First Ten Minutes with Sourcegraph Cody"
This will create a file named:

_drafts/my-first-ten-minutes-with-sourcegraph-cody.md

The contents of the file will be:

---
layout: post
title: "My First Ten Minutes with Sourcegraph Cody" 
---

Draft blog post content here
You can then edit this draft blog post and when ready, move it out of the _drafts folder to publish it.
```

My initial surprise is just how much more productive Cody is making me at solving problems, especially ones that
I don't know where to start. For example, this blog is written using [Jekyll](https://jekyllrb.com/), a framework
that I absolutely do not understand. I came to write this blog post and immediately hit a dependency issue:

```
Your bundle is locked to nokogiri (1.13.6-x86_64-linux) from rubygems repository https://rubygems.org/ or installed locally, but that version can no longer be found in that source. That means the author of nokogiri (1.13.6-x86_64-linux) has removed it. You'll need to update your bundle to a version other than nokogiri (1.13.6-x86_64-linux) that hasn't been removed in order to install.
```

Uhhhhhhhhhhhhh, great? I don't know anything about this language or its toolchain, and to be honest I'd probably
just have given up nearly straight away with this one. Let's ask Cody!

> How do I fix this Ruby dependency issue? Your bundle is locked to nokogiri (1.13.6-x86_64-linux) from rubygems repository https://rubygems.org/ or installed locally, but that version can no longer be found in that source. That means the author of nokogiri (1.13.6-x86_64-linux) has removed it. You'll need to update your bundle to a version other than nokogiri (1.13.6-x86_64-linux) that hasn't been removed in order to install.

```
This error is occurring because the nokogiri gem version 1.13.6-x86_64-linux has been removed from RubyGems.org, but your Gemfile.lock is locked to that specific version.

To fix this, you'll need to:

Update the nokogiri version in your Gemfile to allow newer versions:
gem 'nokogiri', '~> 1.13'
The ~> operator allows patch-level updates to nokogiri.

Run bundle update nokogiri to update nokogiri to the latest version that satisfies the Gemfile requirement.

Commit the changes to your Gemfile and Gemfile.lock.

Run bundle install to install the updated dependencies.

This will update nokogiri to a version still available on RubyGems.org and resolve the dependency issue.

The steps in summary are:

# Update Gemfile
gem 'nokogiri', '~> 1.13'

# Update nokogiri
bundle update nokogiri 

# Commit changes
git add Gemfile Gemfile.lock
git commit -m "Update nokogiri dependency"

# Re-install dependencies
bundle install
```

(Cody also renders the text in nice markdown with syntax highlighting, I just can't show that here properly!).

This is *unbelievably* helpful and solved the issue first try. The comments and explanations are a nice little touch I really appreciate to.

To summarise, in ten minutes, Cody has helped me:
- Understand, in great detail, a CloudFormation template.
- Identify potential flaws in that template (potentially wide IAM permissions on an S3 bucket).
- Explain an ASCII diagram from [my last post]({% post_url 2023-04-01-cloudformation-cloudfront %}) flawlessly (????).
- Generate better CSS than I can (no, I'm not learning frontend).
- Fix a bunch of Ruby issues that I had no hope fixing myself.

Looking forward to seeing what else I can do with it :).

> How do I publish a Jekyll draft?
