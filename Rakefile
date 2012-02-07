require 'rubygems'
require 'rake'
require 'fileutils'

desc "Draft a new post"
task :new do
  puts "What should we call this post for now?"
  name = STDIN.gets.chomp

  date  = Time.now()
  year  = date.strftime("%Y")
  month = date.strftime("%m")
  day   = date.strftime("%d")

  filename = "_posts/#{year}-#{month}-#{day}-#{name}.markdown"

  FileUtils.touch(filename)

  open(filename,'a') do |f|
    f.puts "---"
    f.puts "layout: post"
    f.puts "title: \"DRAFT: #{name}\""
    f.puts "---"
  end
end

desc "Startup Jekyll"
task :start do
  sh "jekyll --server"
end

desc "Deploy..."
task :deploy do
  sh "cp .htaccess _site/"
  sh "rsync -ave ssh --exclude=.DS_Store --exclude=*~ --exclude=stats --delete _site/ admin@hocuspokus.net:/srv/hocuspokus.net/public/htdocs/"
end

task :default => :start

