---
layout: post
title: "Rails security"
description: "Rails security"
category: Rails 
tags: [Rails Security]
---
{% include JB/setup %}

#secure password

		has_secure_password   #add password_digest column to store hashed password
		                      #add password password_confirmation virtual column
							  #add authenticate method   if password is valid return true
		rails generate migration add_password_digest_to_users password_digest:string 
		gem 'bcrypt'
		def setup
		    @user = User.new(name: "Example User", email: "user@example.com",
		                     password: "foobar", password_confirmation: "foobar") #add
		end
		user.authenticate("foobaz")   #first compute the user password 'foobar' to hashed password ,
			 						  #then match database column password_digest value  