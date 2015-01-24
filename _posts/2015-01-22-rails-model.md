---
layout: post
title: "Rails Model"
description: "Rails Model"
category: Rails
tags: [Rails Model Rspec]
---
{% include JB/setup %}

建立model对象,使用rspec测试驱动开发
		
		rails g model contact
		
编辑生成的contact_spec.rb，判断model对象是否成功创建

		it "is valid a firstname, lastname and email" do
		    contact = Contact.new(
		              firstname: 'Aaron',
		              lastname: 'Sumner',
		              email: 'tester@example.com')
		    expect(contact).to be_valid
		end
		
测试验证

		it "is invalid without a firstname" do
		    expect(Contact.new(firstname:nil)).to have(1).errors_on(:firstname)
		end
		
	    validates :firstname, presence: true
		

		

