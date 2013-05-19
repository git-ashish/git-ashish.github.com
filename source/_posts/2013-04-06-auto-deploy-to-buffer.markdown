---
layout: post
title: "Auto deploy to Buffer"
date: 2013-04-06 01:09
comments: true
categories: [octopress, buffer]
---

Octopress is a great simple tool for blogging. A lot of custom stuff can be built upon it. Here is how you can use [BufferApp](https://bufferapp.com) to share you posts to your social networks.

<!-- more -->

## Access Token 
Create an app in your Buffer account and get the app's access token.

## Buffer gem
Install Buffer gem using 

```
gem install buffer 
```

Open the Gemfile and edit below section to make it look like this.

``` ruby
group :development do
  gem 'rake', '~> 0.9'
  gem 'rack', '~> 1.4.1'
  gem 'jekyll', '~> 0.12'
  gem 'rdiscount', '~> 1.6.8'
  gem 'kramdown', '~> 0.13'
  gem 'coderay', '~> 1.0'
  gem 'pygments.rb', '~> 0.3.4'
  gem 'RedCloth', '~> 4.2.9'
  gem 'haml', '~> 3.1.7'
  gem 'compass', '~> 0.12.2'
  gem 'rubypants', '~> 0.2.0'
  gem 'rb-fsevent', '~> 0.9'
  gem 'stringex', '~> 1.4.0'
  gem 'liquid', '~> 2.3.0'
  gem 'twitter'
  gem 'addressable'
  gem 'buffer'
end
```

Note the addition of last gems. You need these for bufferapp to work.

``` ruby
gem 'addressable'
gem 'buffer'

```
## Rakefile
  
Define Access Token

Open Rakefile and add this line after

``` ruby
## -- Misc Configs -- ## ```
buffer_access_token = 'your_buffer_access_token'
```

Adding posts to Buffer Queue

In the Rakefile, find this section

``` ruby
# usage rake new_post[my-new-post] or rake new_post['my new post'] or rake new_post (defaults to "new-post")
desc "Begin a new post in #{source_dir}/#{posts_dir}"
task :new_post, :title do |t, args|
  if args.title
    title = args.title
  else
    title = get_stdin("Enter a title for your post: ")
  end
  raise "### You haven't set anything up yet. First run `rake install` to set up an Octopress theme." unless File.directory?(source_dir)
  mkdir_p "#{source_dir}/#{posts_dir}"
  filename = "#{source_dir}/#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title.gsub(/&/,'&amp;')}\""
    post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
    post.puts "comments: true"
    post.puts "categories: "
    post.puts "---"
  end
end
```

Modify it into this

``` ruby
# usage rake new_post[my-new-post] or rake new_post['my new post'] or rake new_post (defaults to "new-post")
desc "Begin a new post in #{source_dir}/#{posts_dir}"
task :new_post, :title, :buffer do |t, args|
  raise "### You haven't set anything up yet. First run `rake install` to set up an Octopress theme." unless File.directory?(source_dir)
  mkdir_p "#{source_dir}/#{posts_dir}"
  args.with_defaults(:title => 'new-post', :tweet => '')
  title = args.title
  filename = "#{source_dir}/#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title.gsub(/&/,'&')}\""
    post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
    post.puts "comments: true"
    post.puts "categories: "
    post.puts "---"
  end

  buffer = args.buffer
  if not buffer == ''
    # add to bufferapp status queue
    puts 'Adding post to Bufferapp queue, it will be pushed after deploying.'
    open('buffer_queue', 'a') do |file|
      file.puts "#{buffer} - #{blog_url}#{Time.now.strftime('%Y/%m/%d')}/#{title.to_url}/"
    end
  end
end
```

Pushing posts to Bufferapp

In the Rakefile, find this section

``` ruby
##############
# Deploying  #
##############

desc "Default deploy task"
task :deploy do
  # Check if preview posts exist, which should not be published
  if File.exists?(".preview-mode")
    puts "## Found posts in preview mode, regenerating files ..."
    File.delete(".preview-mode")
    Rake::Task[:generate].execute
  end

  Rake::Task[:copydot].invoke(source_dir, public_dir)
  Rake::Task["#{deploy_default}"].execute
end
```

Make it look like this

``` ruby
##############
# Deploying  #
##############

desc "Default deploy task"
task :deploy do
  # Check if preview posts exist, which should not be published
  if File.exists?(".preview-mode")
    puts "## Found posts in preview mode, regenerating files ..."
    File.delete(".preview-mode")
    Rake::Task[:generate].execute
  end

  Rake::Task[:copydot].invoke(source_dir, public_dir)
  Rake::Task["#{deploy_default}"].execute

  # Buffer
  next if not File.exists? 'buffer_queue'
  puts "Buffering..."
  bufferapp = Buffer::Client.new buffer_access_token
  open('buffer_queue', 'r') do |file|
  while (line = file.gets)
    puts "Buffering '#{line.gsub("\n", "")}' ..."
    # ud = bufferapp.api :get, 'profiles'
    # TODO - get profiles automatically
    ud = bufferapp.api :post, 'updates/create', :text => line, :profile_ids => ['5xxxxxxxxxxxxxxxx','5xxxxxxxxxxxxxxx']
    puts ud
  end
end
puts "Deleting queue..."
rm 'buffer_queue'

end
```

profile_ids - you can see two values in the above code as ``5xxxxxxxxxxxxxxxxx``. These are the Buffer profiles or linked social accounts where you would like to share the blog post. You can get this id from the URL of your linked social buffer account when you visit the Bufferapp dashboard. 
``https://bufferapp.com/app/profile/5xxxxxxxxxxxxxxxxx/buffer``

## Sharing

Now, everything necessary to Buffer your blog post is setup. All you need is to create the blog post. To create a blog post you write
``` ruby
rake new_post['my_awesome_blog_post_name']
```
For blog posts which you want to share via Bufferapp, you write
``` ruby
rake new_post['my_awesome_blog_post_name','my_awesome_blog_post_title_to_be_shared_via_buffer']
```
#### _Happy blogging_.