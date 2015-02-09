---
layout: post
title: "Rails test"
description: "Rails test"
category: Rails test
tags: [Rails test]
---
{% include JB/setup %}

write unit test process

when  

assert

#debug

		<%= debug(params) if Rails.env.development? %>  # in the application.html.erb
		
		debugger   #in the controller can trigger console , 
				   #show the web console in the action web page,
				   #in the server console to press ctrl+d exit
#assist method
---

		get :action, {'id' => 12},{'user_id' => 1},{'message' => 'boody'} #request,session,flash parameters
		post :create, post: {title: 'Some title'}  #like get

##four hash
		
		assigns(:posts)     #from controller assign to view instance object
		flash[:gordon]   
		session[:shmession]
		cookies[:are_good_for_u]
		
		@user = assigns(:user)
		
##three instance variable in controller test
@controller,@request,@response

		@request.headers["Accept"] = "text/plain, text/html"  #setup HTTP head
		@request.headers["HTTP_REFERER"] = "http://example.com/home" #setup CGI variable
		

#YAML
---
		
		david:                        #users.yml
		  name: David Heinemeier Hansson
		  birthday: 1979-10-15
		  profession: Systems development 
		steve:
		  name: Steve Ross Kellock
		  birthday: 1974-09-27
		  profession: guy with keyboard

##association
belongs_to  has_many
		  
		about:						# In fixtures/categories.yml
		  name: About
 
		one:						# In fixtures/articles.yml
		  title: Welcome to Rails!
		  body: Hello world!
		  category: about            association

##use ERB improve YML

		<% 1000.times do |n| %>
		user_<%= n %>:
		  username: <%= "user#{n}" %>
		  email: <%= "user#{n}@example.com" %>
		<% end %>
		
##fixture is  Active Record  object

		users(:david)           #return user object 
		users(:david).id
		
#model test
---
##assert true   not

		assert @user.valid?     test model validator
		assert_equal expected, actual, [msg]
		assert_same( expected, actual, [msg] )
		assert_nil( obj, [msg] )
		assert_match( regexp, string, [msg] )     #regexp
		assert_in_delta( expecting, actual, [delta], [msg] )  #delta range
		assert_instance_of( class, obj, [msg] )     #direct instance
		assert_kind_of( class, obj, [msg] )      #include all parent
		assert_respond_to( obj, symbol, [msg] )       #symbol is method
		
##rails assert

		assert_difference(expressions, difference = 1, message = nil) {...}
		assert_no_difference(expressions, message = nil, &amp;block)
		assert_response(type, message = nil)
		assert_redirected_to(options = {}, message=nil)
		assert_template(expected = nil, message=nil)
		
#rails controller assert
---
##if response is success

		get :index
		assert_response :success

##if redirect page or render template

		post :create, post: {title: 'Some title'}
		assert_redirected_to post_path(assigns(:post))
		
		get :index
		assert_template :index
		assert_template layout: "layouts/application"
		
		assert_template layout: "layouts/application", partial: "_form"
		
		assert_no_difference 'User.count' do
		  post users_path, user: { name: "", email: "user@invalid",
		    password: "foo", password_confirmation: "bar" }
		end
		assert_difference 'User.count', 1 do
		  post_via_redirect users_path, ...
		end
		
##if pass auth
##if pass @object to template

		get :index
		assert_not_nil assigns(:posts)
		
##if show message on template
		
		test "email validation should accept valid addresses" do
		  valid_addresses = %w[user@example.com USER@foo.COM A_US-ER@foo.bar.org first.last@foo.jp alice+bob@baz.cn]
		  valid_addresses.each do |valid_address|
			@user.email = valid_address
			assert @user.valid?, "#{valid_address.inspect} should be valid"
		  end 
		end

##test route

		assert_routing '/posts/1', {controller: "posts", action: "show", id: "1"}
		
#view test
---
		assert_select 'title', "Welcome to Rails Testing Guide"		
		assert_select 'ul.navigation' do
		  assert_select 'li.menu_item'
		end
		assert_select "a[href=?]", root_path, count: 2    put root_path into ?
		assert_select "div", "foobar"     <div>foobar</div>

#integration_test
---
##help method

		https?  if simulator https request return true
		https!  simulate https start
		https!(false) simulate https stop
		host!  setup next request host name
		redirect?   if redirect return true
		follow_redirect!   track a redirect
		post_via_redirect(path, [parameters], [headers])  
		open_session    create a new session
		
##custom help method

		david = login(:david)
		def login(user)
		   open_session do |sess|
		     sess.extend(CustomDsl)
		     u = users(user)
		     sess.https!
		     sess.post "/login", username: u.username, password: u.password
		     assert_equal '/welcome', sess.path
		     sess.https!(false)
		   end
		end
		
#before and after test
---
		def setup			  #if
		    @post = posts(:one)
		end
		
		def teardown           #clean
		    @post = nil
		end

#assert help method
---
		include UserHelper
		
#setup test_helper.rb Guardfile
---
		group :test do
		  gem 'minitest-reporters'
		  gem 'mini_backtrace'
		  gem 'guard-minitest'
		end
		require "minitest/reporters" 
		Minitest::Reporters.use!
		
		class ActiveSupport::TestCase
		  fixtures :all
		  include ApplicationHelper    #use ApplicationHelper method in test
		end
		
##Guardfile
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
		  
		guard :minitest, spring: true, all_on_start: false do
		  watch(%r{^test/(.*)/?(.*)_test\.rb$}) 
		  watch('test/test_helper.rb') { 'test' } 
		  watch('config/routes.rb') { integration_tests } 
		  watch(%r{^app/models/(.*?)\.rb$}) do |matches|
		    "test/models/#{matches[1]}_test.rb" 
		  end
		  watch(%r{^app/controllers/(.*?)_controller\.rb$}) do |matches| 
		    resource_tests(matches[1])
		  end
		  watch(%r{^app/views/([^/]*?)/.*\.html\.erb$}) do |matches|
		    ["test/controllers/#{matches[1]}_controller_test.rb"] +
		integration_tests(matches[1]) 
		  end
		  watch(%r{^app/helpers/(.*?)_helper\.rb$}) do |matches| 
		    integration_tests(matches[1])
		  end 
		  watch('app/views/layouts/application.html.erb') do
		    'test/integration/site_layout_test.rb' 
		  end
		  watch('app/helpers/sessions_helper.rb') do 
		    integration_tests << 'test/helpers/sessions_helper_test.rb'
		  end 
		  watch('app/controllers/sessions_controller.rb') do
		    ['test/controllers/sessions_controller_test.rb', 'test/integration/users_login_test.rb']
		  end 
		  watch('app/controllers/account_activations_controller.rb') do
		    'test/integration/users_signup_test.rb' 
		  end
		  watch(%r{app/views/users/*}) do 
		    resource_tests('users') + ['test/integration/microposts_interface_test.rb']
		  end 
		end
		def integration_tests(resource = :all)
		  if resource == :all 
		    Dir["test/integration/*"]
		  else 
		    Dir["test/integration/#{resource}_*.rb"]
		  end 
		end
		# Returns the controller tests corresponding to the given resource. 
		def controller_test(resource)
		  "test/controllers/#{resource}_controller_test.rb" 
		end
		# Returns all tests for the given resource. 
		def resource_tests(resource)
		  integration_tests(resource) << controller_test(resource) 
		end
		
		
		




