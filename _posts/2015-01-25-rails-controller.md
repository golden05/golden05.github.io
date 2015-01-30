---
layout: post
title: "Rails Controller"
description: "Rails Controller"
category: Rails 
tags: [Rails Controller Rspec]
---
{% include JB/setup %}

##默认的RESTful动作 index show new edit create update destroy
四个动作 GET POST PATCH DELETE,其中PATCH DELETE是rails在POST加上一个隐含参数_method=PATCH或_method=DELETE
		
		<%= button_to 'Delete', event_path(event), :method => :delete %>
		
#路由
config/routes.rb
##Resources 资源

		rake routes  查看现有路由
		root 'posts#index'    根路由
		Rails.application.routes.draw do
		  resources :posts
		end

		resources :posts do 
			resources :comments   嵌套路由 edit_post_comment_path
		end
		
		resources :posts do
			resources :comments, only: :create   受限的Resources
		end
		
		resource :events, :except => [:index, :show] 
		
		resources :events do
		  resources :attendees, :controller => 'event_attendees'  使用其他的controller
		end
		
		resources :events do
		    collection do
		        get :search    这个controller多了一个search方法也是类似index一样是多数据
		    end				   search_events_path
		end
		
		get  :sold, :on => :collection 同上
		
		resources :events do
		    member do
		        get :dashboard  类似于show方法 dashboard_event_path(event)
		    end
		end
		
		resource :post  注意是resource 没有index Action
		
		resources :events do
		    resource :dashboard  注意是resource 只有一个Action叫做show
		end
		
		namespace :admin do
		  resources :events  这个controller是Admin::EventsController admin_events_path  
		end
		
		get 'login'  => 'user_sessions#new'  自定义路由
		post 'login' => 'user_session#create'
		delete 'logout' => 'user_sessions#destroy'
		
		get '/meetings' => 'events#index', as: "meetings"  命名路由meetings_path和meetings_url
		
		Path Helpers		urL Helpers   url 绝对路径
		posts_path 			posts_url 
		new_post_path 		new_post_url
		edit_post_path(id)	edit_post_url(id) 
		post_path(id)		post_url(id)
		
		app.posts_path     可以在rails console 中查询posts的路由
		app.root_path      查根路由
		
		before_action :set_post, only: [:show, :edit, :update, :destroy] 调用set_post方法

##protect_from_forgery
启动CSRF安全性功能，所有非GET的HTTP request都必須帶有一个Token参数才能存取，Rails会自动在所有表单中帮你插入Token参数，预设的Layout中也有一行<%= csrf_meta_tag %>标签可以让JavaScript读取到这个Token。

		skip_before_action :verify_authenticity_token 在特定controller取消
		
#request处理流程
做三件事情: 
## 收集request的资讯，例如使用者传进來的参数 
action名称、cookie、http headers、params、request、response、session

##参数

		get 'meetings/:id' => 'events#show'  会把:id部分转成params[:id]传入Controller
		
		@post = Post.find(params[:id])  是指在路由后面的 http://localhost:3000/posts/1 
		def post_params				Strong Parameters
		  params.require(:post).permit(:title, :body) 通过require和permit把params里提取出{"title"=>"dd1", "body"=>"eee"}
		end
		
## 操作Model来做资料的处理 
## 返回response结果，这个动作称为render或redirect_to

		render "/events/index.html.erb" 
		render :action  到另一个action的页面文件不执行另一个action的方法
		render :text => "Hello" 
		render :xml => @event.to_xml
		render :json => @event.to_json
		
redirect_to 要使用者转向其他页面

		redirect_to events_url
				
##Response Formats
respond_to可以在一个Action支持多种格式 : HTML JSON  XML  PDF

		respond_to do |format|   respond_to 方法 传递一个含format参数的block
		 format.html { redirect_to posts_url }
		 format.json { head :no_content }
		end
		
##session
默认用Cookies session storage存储Session
		session[:cart] = Cart.new
		
		
#Flash 
在下一个action中显示
Flash的消息存储在user的session在cookie中，在controller中设置在view中读取显示，使用过自动清除

		redirect_to @post, notice: 'Post was successfully created.' 设置
		flash[:notice] = "event was successfully created"    设置
		
		<p style="color: green"><%= flash[:notice] %></p>    显示
		flash.now[:notice] = "foobar"   在当前action中显示
		
#Filter
Three kinds of filters:before_action、after_action and around_action
accept a block,a symbol action method , with :only :except

if need to cancel inherited from the parent class ,can use 

		skip_before_action :filter_method_name
		skip_after_action
		skip_around_action
		
#rescue_from
exceptions can be declared in th Controller rescued

		rescue_from ActiveRecord::RecordNotFound, :with => :show_not_found method
		rescue_from ActiveRecord::RecordInvalid, :with => :show_error method
		
#detective variant client  device
setup request.variant 
		
		case request.user_agent
		    when /iPad/i
		      request.variant = :tablet
		    when /iPhone/i
		      request.variant = :phone
		    when /Android/i && /mobile/i
		      request.variant = :phone
		    when /Android/i
		      request.variant = :tablet
		    when /Windows Phone/i
		      request.variant = :phone
		    else
		      request.variant = :desktop
		    end
		  end

in the action read  request.variant 
  
		  respond_to do |format|
		  	format.html
		  	format.html.phone       index.html+phone.erb
			format.html.desktop     index.html+desktop.erb
		  end
		  
#layout 
	
	layout "special"   in the controller app/views/layouts/special.html.erb
	layout "special", :only => :index
	layout "special", :except => [:show, :edit, :new]
	
	render :layout => "foobar"   in the controller action
		
		
		
				
