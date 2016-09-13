# encoding: utf-8

require 'rubygems'
require 'rake'
require 'tempfile'
require 'rake/clean'
require 'scss_lint/rake_task'
require 'w3c_validators'
require 'nokogiri'
require 'spellchecker'

task default: [:clean, :build, :scss_lint, :pages]

desc "Lint SASS sources"
SCSSLint::RakeTask.new do |t|
  f = Tempfile.new(['bloghacks-', '.scss'])
  f << File.open('css/main.scss').drop(2).join("\n")
  f.flush
  f.close
  t.files = Dir.glob([f.path])
end

desc "Build Jekyll site"
task :build do
  system('jekyll build')
  raise 'Jekyll failed' unless $?.exitstatus == 0
end

desc "Check the existence of all critical pages"
task :pages => [:build] do
  [
    'robots.txt',
    'css/main.css',
    'CNAME',
    'about.html',
    'favicon.ico',
    'images/tomato.svg',
    'rss.xml',
    'sitemap.xml',
    '2016/09/12/first-post.html'
  ].each do |p|
    file = "_site/#{p}"
    raise "Page #{file} is not found" unless File.exists? file
    puts "Page #{file} is in place"
  end
end

desc "Validate a few pages for W3C compliance"
# It doesn't work now, because of: https://github.com/alexdunae/w3c_validators/issues/16
task :w3c => [:build] do
  include W3CValidators
  validator = MarkupValidator.new
  [
    'index.html',
    '2016/09/12/first-post.html'
  ].each do |p|
    file = "_site/#{p}"
    results = validator.validate_file(file)
    if results.errors.length > 0
      results.errors.each do |err|
        puts err.to_s
      end
      raise "Page #{file} is not W3C compliant"
    end
    puts "Page #{p} is W3C compliant"
  end
end

desc "Check spelling in all HTML pages"
task :spell => [:build] do
  Dir["_site/**/*.html"].each do |f|
    html = Nokogiri::HTML(File.read(f))
    html.search('//code').remove
    html.search('//script').remove
    html.search('//pre').remove
    html.search('//header').remove
    html.search('//footer').remove
    tmp = Tempfile.new(['bloghacks-', '.txt'])
    tmp << html.xpath('/html/body/section').text
    tmp.flush
    tmp.close
    stdout = `cat "#{tmp.path}" | aspell -a --lang=en_US --ignore=2 --ignore-case -p ./_rake/aspell.en.pws | grep ^\\&`
    raise "Typos at #{f}:\n#{stdout}" if !stdout.empty?
    puts "Spell check passed, no typos at #{f}"
  end
end
