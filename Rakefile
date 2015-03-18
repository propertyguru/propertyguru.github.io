# -*- encoding : utf-8 -*-

require "rubygems"
require 'rake'

namespace :generate do

  # Generate tag files from all existing posts to tags folder
  #    Options
  #        path="tags"       Tags directory to store generated tag page (default: 'tags')
  #        layout="tags"     Tag page layout (default: 'tags')
  #    Usage
  #        rake generate:tag
  #        rake generate:tag path="tags" layout="tags"
  #
  desc "Generate tag files from all existing posts to tags folder"
  task :tag do
    tag_path = ENV['path'] || 'tags'
    tag_layout = ENV['layout'] || 'tags'

    found_post = false
    tags = []

    Dir.glob('_posts/*') do |file|
      # file = "_posts/2014-05-15-generate-jacoco-multi-project-gradle.markdown"
      found_post = true
      content = File.read(file)
      match_str = /-{3}.*?\ntags:\s*([^\n]+)\n.*/m.match(content)
      if match_str && match_str[1]
        if match_str[1].length > 1
          tags.concat(match_str[1].split)
        end
      end
    end

    if found_post
      if tags.empty?
        puts "No tags found on any post"
      else
        if Dir.exist?(tag_path)
          FileUtils.remove_dir(tag_path)
          puts "Found existing \"#{tag_path}\" folder. Remove folder."
        end

        FileUtils.mkdir(tag_path)

        tags.uniq.each do |tag|
          puts "Creating tag file: #{tag_layout}/#{tag}.html"
          open(tag_path + '/' + tag + '.html', 'w') do |post|
            post.puts "---"
            post.puts "layout: #{tag_layout}"
            post.puts "tag: #{tag}"
            post.puts "permalink: tags/#{tag}/"
            post.puts "---"
          end
        end
      end
    else
      puts "No post found"
    end

    open(tag_path + '/index.html', 'w') do |index|
      index.puts "---"
      index.puts "layout: default"
      index.puts "title: Tags"
      index.puts "---"
      index.puts "{% include tags_cloud.html %}"
    end

  end # task :tag
end
