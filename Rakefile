def jekyll(opts = "")
  sh "rm -rf _site"
  sh "jekyll " + opts
end

desc "Build site using Jekyll"
task :build => :tags do
  jekyll
end

desc "Serve on Localhost with port 4000"
task :default do
  jekyll("--server --auto")
end

desc "Deploy to live site"
task :deploy => :"deploy:live"

namespace :deploy do
  desc "Deploy to Dreamhost"
  task :live => :build do
    rsync "blog.rayapps.com"
  end

  def rsync(domain)
    sh "rsync -rtz --delete _site/ raymonds72@cayman.dreamhost.com:~/#{domain}/"
  end
end

desc 'Generate tags pages'
task :tags  => :tag_cloud do
  puts "Generating tags..."
  require 'rubygems'
  require 'jekyll'
  include Jekyll::Filters

  options = Jekyll.configuration({})
  site = Jekyll::Site.new(options)
  site.read_posts('')

  # Remove tags directory before regenerating
  FileUtils.rm_rf("tags")

  site.tags.sort.each do |tag, posts|
    html = <<-HTML
---
layout: default
title: "Tag archive: #{tag}"
---
<h1>Posts tagged with "#{tag}"</h1>

{% for post in site.tags.#{tag} %}
  {% include show_post.html %}
{% endfor %}
HTML

    FileUtils.mkdir_p("tags/#{tag}")
    File.open("tags/#{tag}/index.html", 'w+') do |file|
      file.puts html
    end
  end
  puts 'Done.'
end

desc 'Generate tags pages'
task :tag_cloud do
  puts 'Generating tag cloud...'
  require 'rubygems'
  require 'jekyll'
  include Jekyll::Filters

  options = Jekyll.configuration({})
  site = Jekyll::Site.new(options)
  site.read_posts('')

  html = ''
  max_count = site.tags.map{|t,p| p.count}.max
  site.tags.sort.each do |tag, posts|
    s = posts.count
    font_size = ((20 - 10.0*(max_count-s)/max_count)*2).to_i/2.0
    html << "<a href=\"/tags/#{tag}\" title=\"Postings tagged #{tag}\" style=\"font-size: #{font_size}px; line-height:#{font_size}px\">#{tag}</a> "
  end
  File.open('_includes/tag_cloud.html', 'w+') do |file|
    file.puts html
  end
  puts 'Done.'
end
