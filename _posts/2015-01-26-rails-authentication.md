---
layout: post
title: "Rails Authentication"
description: "Rails Authentication"
category: Rails
tags: [Rails Authentication Devise]
---
{% include JB/setup %}

#Sign Up注册
密码使用hash版本 用 Bcrypt.gem
##has_secure_password
在model对象实现密码功能,增加了password 和 password_confirmation属性，这两个属性增加了校验功能，如果匹配就把密码自动hash化并存在password_digest文本字段

		rails g migration AddPasswordDigistToUsers password_digest
		has_secure_password
		get 'signup', to: 'users#new', as: 'signup'
		
		def user_params
		  params.require(:user).permit(:name, :email, :password,:password_confirmation)
		end
		
##Log In
用户身份存在session中，有user_id，默认存在于cookie，是加密的

		session[:user_id] = @user.id   在rails中把id存到cookie的session中
		rails g controller Sessions
		resources :sessions   在routes.rb中
		get 'login', to: 'sessions#new', as: 'login'
		get 'logout', to: 'sessions#destroy', as: 'logout'

app/views/sessions/new.html.erb  使用form_tag帮助方法

		<%= form_tag sessions_path do %> <div class="form-group">
		     <%= label_tag :email %>
		     <%= email_field_tag :email, params[:email],
		       class: "form-control" %>
		     </div>
		     <div class="form-group">
		       <%= label_tag :password %>
		       <%= password_field_tag :password, nil,
		         class: "form-control" %>
		     </div>
		    <%= submit_tag "Log In", class: "btn btn-primary" %>
		<% end %>
		
		class SessionsController < ApplicationController
			def new    就是render login 表单
			end
			def create  验证过程
				user = User.find_by(email: params[:email])
				if user && user.authenticate(params[:password])
		         session[:user_id] = user.id   验证通过存储id
		         redirect_to root_url, notice: "Log in successful!"
		       else
		         flash.now.alert = "Invalid email or password"
		         render "new"
		       end
			end
			def destroy session[:user_id] = nil
		       redirect_to root_url, notice: "Log out successful!"
		    end
		end
		
current user在ApplicationController增加current_user方法

		def current_user    返回一个当前登录的current_user，如果nil就是没有用户登录
		    if session[:user_id]
		      @current_user ||= User.find(session[:user_id])
		    end
		end
		  helper_method :current_user    转化为帮助方法可以在控制器和view中使用

##认证用户
使用before_action方法限制网页访问

		before_action :authenticate_user!
		def authenticate_user!     
		       redirect_to login_path unless current_user
		end
		
##判断用户
当前用户的信息

		def edit
		     @text_post = current_user.text_posts.find(params[:id])
		end
		
		<% if text_post.user == current_user %>
		
##防止注入攻击
###sql注入
输入一个sql语句
		
		def self.authenticate
		  username = params[:username]
		  password = params[:password]
		  where(username: username,
		        password: password).first
		end



