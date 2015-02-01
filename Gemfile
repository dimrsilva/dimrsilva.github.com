source 'https://rubygems.org'

require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'jekyll', '2.4.0'
gem 'jekyll-minibundle'
gem 'coderay'
gem 'rake'

gem 'github-pages', versions['github-pages']
