require 'fileutils'

def new_post_file(name,linkblog=nil)
  date  = Time.now()
  year  = date.strftime("%Y")
  month = date.strftime("%m")
  day   = date.strftime("%d")

  filename = "_posts/#{year}-#{month}-#{day}-#{name}.markdown"

  FileUtils.touch(filename)

  open(filename,'a') do |f|
    f.puts "---"
    f.puts "layout: post"
    f.puts "title: #{name}"
    f.puts "permalink: /#{year}/#{month}/#{name}"
    f.puts "tags: []"
    f.puts "linkblog: #{linkblog}" unless linkblog.nil?
    f.puts "---"
  end
end

desc "Write a new post"
task :new_post do
  puts "What should we call this post for now?"
  name = STDIN.gets.chomp
  new_post_file(name)
end

desc "Write a new linkblog post"
task :new_linkblog do
  puts "What should we call this post for now?"
  name = STDIN.gets.chomp
  puts "What are we linking to?"
  link = STDIN.gets.chomp
  new_post_file(name,link)
end

desc "Startup Jekyll"
task :start do
  sh "bundle exec jekyll serve"
end

desc "Deploy..."
task :deploy do
  sh "cp .htaccess _site/"
  sh "rsync -ave ssh --exclude=.DS_Store --exclude=*~ --exclude=stats --exclude=fever --delete _site/ admin@hocuspokus.net:/srv/hocuspokus.net/public/htdocs/"
end

task :default => :start
