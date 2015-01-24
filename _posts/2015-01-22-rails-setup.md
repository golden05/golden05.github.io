---
layout: post
title: "Rails Setup"
description: "Rails Setup"
category: Rails
tags: [Rails]
---
{% include JB/setup %}

#setup
rails new app -T --database=postgresql

修改gem文件
		
		gem 'slim-rails'
		gem 'unicorn'
		gem 'devise'
		gem 'omniauth'
		gem 'cancan'
		gem 'guard'
		group :development, :test do
		  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
		  gem 'byebug'
		  gem 'guard-rspec'
		  gem 'rspec-collection_matchers'
		  gem 'rspec-rails'
		  gem 'factory_girl_rails'
		  # Access an IRB console on exception pages or by using <%= console %> in views
		  gem 'web-console', '~> 2.0'

		  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
		  gem 'spring'
		end
		group :test do
		  gem 'capybara'
		  gem 'database_cleaner'
		  gem 'launchy'
		end

然后运行 bundle 安装必要的gem

	 	rails g rspec:install
		guard init rspec
		
打开Guardfile,添加以下内容，在tmux输出信息

		notification :tmux,
		  display_message: true,
		  timeout: 5, # in seconds
		  default_message_format: '%s >> %s',
		  # the first %s will show the title, the second the message
		  # Alternately you can also configure *success_message_format*,
		  # *pending_message_format*, *failed_message_format*
		  line_separator: ' > ', # since we are single line we need a separator
		  color_location: 'status-left-bg', # to customize which tmux element will change color

		  # Other options:
		  default_message_color: 'black',
		  success: 'colour150',
		  failure: 'colour174',
		  pending: 'colour179',

		  # Notify on all tmux clients
		  display_on_all_clients: false
		  
打开application.rb,在运行测试生成文件时不产生其他文件
		
		config.generators do |g|
		      g.test_framework :rspec,
		        fixtures: true,
		        view_specs: false,
		        helper_specs: false,
		        routing_specs: false,
		        controller_specs: true,
		        request_specs: false
		      g.fixture_replacement :factory_girl, dir: "spec/factories"
		end

创建所有数据库

		rake db:create:all
		
在rails的4.2版本 数据库已经可以自动clone到测试数据库，不用运行rake:db:clone
		
		
