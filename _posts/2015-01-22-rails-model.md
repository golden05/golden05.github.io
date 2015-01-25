---
layout: post
title: "Rails Model"
description: "Rails Model"
category: Rails
tags: [Rails Model Rspec]
---
{% include JB/setup %}

#model的 crud 方法
		
		rails g model contact
		Post.create title: "First Post"
		post = Post.new
		post.save
		Post.all
		Post.first
		Post.last
		post = Post.find 2 如果找不到抛出 ActiveRecord::RecordNotFound 例外
		post = Post.where(title: "First Post").first where返回的是relation
		post = Post.find_by title: "First Post"
		post.update title: "Second Post"   更新
		post.destroy   删除
		Post.destroy 2  
		Post.where(title: "First Post").destroy_all
		
##条件查询

		posts = Post.limit(10)
		posts = Post.limit(10).offset(10) 返回的是relation
		posts = Post.limit(10).order "created_at DESC"


##运算

		count = Post.count  计数
		date = Post.maximum :created_at 类似方法sum, average, minimum
		
##迁移
schema_migrations保持迁移跟踪		

		rake db:migrate
		rake db:rollback  回退
		
##Schema模式
db/schema.rb 保持数据库的当前状态
		
		rake db:schema:load  建立新数据库更新
		
##增加一个列
		
		rails g migration add_author_to_posts author:string add_ColumnName_to_TableName
		
#验证Validation
Validation作为类方法实现

		validates :title, :presence => true
		:uniqueness
		validates :title, :length => { :minimum => 5 } 或者使用:is表示一个确切值
		validates :title, :exclusion => { :in => [ "Title" ] } 或者:inclusion
		
##测试数据

		post.valid?  用于判断验证是否有效
		post.errors.full_messages  出错信息
		
#连接associations

		rails g model Comment author:string body:text post:references
		has_many :comments
		belongs_to :post
		post.comments.create :author => "Tony", :body => "Test comment"
		post.comments  返回一个 ActiveRecord::Relation
		post.comments.empty?  是否为空
		post.comments.size
		post.comments.find comment_id 如果找不到抛出ActiveRecord::RecordNotFound
		build_post 创建这个comment的post，但没有存到数据库中
		create_post  创建这个comment的post，并保存到数据库中
		create_post! 创建这个comment的post，存到数据库中,如果无效出错抛出ActiveRecord::RecordInvalid
		
#自连接

		has_many :subordinates, class_name: 'Employee', foreign_key: 'manager_id'
		belongs_to :manager, class_name: 'Employee' class指向同一个类 
		
#多－多连接
两个不同的方法
##has_and_belongs_to_many
		has_and_belongs_to_many :books  要有一个join表authors_books实现这个连接
		rails g migration CreateAuthorsBooks 创建一个空的迁移文件，做以下修改
		def change
		  create_table :authors_books, id: false do |t|
		    t.references :author, null: false, index: true 
			t.references :book, null: false, index: true
	    end
		
		create_join_table :authors, :books do |t|   另一个创建方法
		      t.index :author_id
		      t.index :book_id
		end         
		
##has_many :through      

		class Band < ActiveRecord::Base
		  has_many :performances
		  has_many :venues, through: :performances
		end
		
		class Performance < ActiveRecord::Base
		  belongs_to :band
		  belongs_to :venue
		end
		
#Single-Table单表继承
使用了一个字段type来保持继承，type值为每个对象
		
		class Pet < ActiveRecord::Base
		end
		class Dog < Pet
		end
		class Fish < Pet
		end
		id		type		name		cost
		1 		Dog 		Collie		200
		2 		Fish 		Gold Fish	5
		3 		Dog			Cocker Spaniel	100
		
		Dog.count 返回 2  Fish.count 返回 1
		
#polymorphic多态关联连接
一个模型归属于超过一个以上的模型

		class Post < ActiveRecord::Base
		  has_many :comments, as: :commentable
		end
		class Image < ActiveRecord::Base
		  has_many :comments, as: :commentable
		end
		class Comment < ActiveRecord::Base
		  belongs_to :commentable, polymorphic: true
		end
		
#社交应用
		
		rails generate resource User name email 
		rails g model Subscription leader:references follower:references
		class Subscription < ActiveRecord::Base 
			belongs_to :leader, class_name: 'User' 
			belongs_to :follower, class_name: 'User'
		end
		class User < ActiveRecord::Base
		  has_many :subscriptions, foreign_key: :follower_id,
		                           dependent: :destroy
		  has_many :leaders, through: :subscriptions
		end
		
		rails g resource Post title body:text url type user:references
		rails g resource TextPost --parent=Post --migration=false 继承关系，不单独创建表存在Post表
		class TextPost < Post
		     validates :body, presence: true
		end
		

		

