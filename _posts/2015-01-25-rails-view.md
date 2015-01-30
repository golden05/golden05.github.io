---
layout: post
title: "Rails View"
description: "Rails View"
category: Rails
tags: [Rails View Rspec]
---
{% include JB/setup %}

#embedded Ruby
##输出
<%= %>

		<%= @post.title %>
		
##控制流
<% %>
		
		<% @posts.each do |post| %>
		<% end %>

#helpers

		javascript_include_tag
		stylesheet_link_tag
		auto_discovery_link_tag
		favicon_link_tag
		image_tag
		video_tag
		audio_tag
		
##URL Helpers

		link_to 文字超連結  附加HTTP动词如post get
		mail_to E-mail
		button_to 按鈕連結
		current_page?
		
##form helper

		form_for   model
		form_tag   no model

method: :delete 生成data-method="delete"

data: { confirm: 'Are you sure?'} 生成data-confirm="Are you sure?"

数字帮助器

		number_to_currency  100    $100.00
		number_to_human  1000000   1 million
		number_to_human_size 1024  1 KB
		number_to_percentage 12.345, precision: 1  12.3%
		number_with_delimiter
		number_with_precision
		
自己创建助手

		module ApplicationHelper
		  def friendly_date(d)    可以在view和controller中使用
		    d.strftime("%B %e, %Y")
		  end
		end  
		
#layouts
		
		<%= yield %>   嵌入自己的erb文件
		<%= yield :sidebar %>  setup in the layout template file
		<%= content_for :sidebar do %>   use in the erb
		
#Variable 变量
except @variable , cookies,session,flash,params,request,response,controller

		<%= tag(:body, :class => "#{controller.controller_name} #{controller.action_name}") %>
		
		
##Asset Tag Helpers
app/assets 包含 CSS, JavaScript, images

stylesheet_link_tag 指出 默认CSS，app/assets/stylesheets/application.css

		/* 
		 *= require_tree .  包含所有目录
		 *= require_self    包含自身
		 */

javascript_include_tag 指出了js, app/assets/javascript/application.js

		//
		//= require jquery
		//= require jquery_ujs 
		//= require turbolinks 
		//= require_tree .
		
CSRF Meta Tags Helper csrf_meta_tags 生成两行元信息防止CSRF攻击

##Partials
重用局部页面，以下划线带字母开头 文件名作为变量

		<%= render 'form' %>  调用_form.html.erb文件
		<%= render :partial => 'post', :collection => @posts %> 循环@posts局部文件 or tr li 
		
##Form 表单
帮助方法 form_for(@post) do block 必须基于model对象

button_to 和link_to 的区别在如果客户端浏览器JavaScript没开link_to就不能提交除GET外的动作

##simple_form gem

		<%= simple_form_for @user do |f| %>  可以自定义覆盖
		  <%= f.input :username, label: 'Your username please' %>
		  <%= f.input :date_of_birth, as: :date, start_year: Date.today.year - 90,
		                              end_year: Date.today.year - 12, discard_day: true,
		                              order: [:month, :year] %>
		  <%= f.input :password, hint: 'No special characters.' %>
		  <%= f.input :email, placeholder: 'user@domain.com' %>
		  <%= f.input :remember_me, inline_label: 'Yes, remember me' %>
		  <%= f.input :description, as: :text %>
		  <%= f.input :accepts,     as: :radio_buttons %>
		  <%= f.input :username, disabled: true, hint: 'You cannot change your username.' %>
		  <%= f.button :submit %>
		  <%= f.input :age, collection: 18..60, prompt: "Select your age"%>
		  <%= f.association :company %>
		  <%= f.association :roles %>
		  f.association :company, collection: Company.active.all(order: 'name'), prompt: "Choose a Company"		  
		<% end %>

国际化
		
		en:
		  simple_form:
		    labels:
		      user:
		        username: 'User name'
		        password: 'Password'
		        edit:
		          username: 'Change user name'
		          password: 'Change password'




