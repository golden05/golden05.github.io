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
		post.save   pass validate will return true
		post.save!  throw exception while invalidated
		Post.create equal new and save pass validate will return true 
		Post.create! equal new and save throw exception 
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
		update_attribute
		user.update_attribute(:name, "The Dude")   #use a attribute's hash to update
		user.update_attributes(name: "The Dude", email: "dude@abides.org") #update multi attributes
		
		
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

		e = Event.new
		e.valid?      
		validates :title, :presence => true
		validates :email, uniqueness: true
		validates :name, uniqueness: { scope: :year,
		    message: "should happen once per year" }
		validates :title, length: { minimum: 5 } 或者使用:is表示一个确切值
		validates :title, exclusion: { in: [ "Title" ] } 或者:inclusion
		validates :points, numericality: true
		validates :games_played, numericality: { only_integer: true }
		validates :email, format: { with: /\A([\w\.%\+\-]+)@([\w\-]+\.)+([\w]{2,})\z/i }
		validates :size, inclusion: { in: %w(small medium large),
		    message: "%{value} is not a valid size" }     #only value
		validates :subdomain, exclusion: { in: %w(www us ca jp),
		    message: "%{value} is reserved." }            #exclusion value
		
		validates :email, confirmation: true    #confirm  email
		<%= text_field :person, :email %>
		<%= text_field :person, :email_confirmation %>
		
##公用参数 Common Parameters

		allow nil     if value is nil then ignore validate
		allow blank   if value is nil or blank then ignore validate
		message       custom  message
		on			  on: create 
		if,unless     condition
		
##Callback回调   顺序order

		(-) save
		(-) valid
		(1) before_validation  :method
		(-) validate
		(2) after_validation  :method
		(3) before_save  :method
		(4) before_create  :method
		(-) create
		(5) after_create  :method
		(6) after_save	:method
		(7) after_commit  :method
		
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
		build_post 创建这个comment的post，但没有存到数据库中 关联名
		create_post  创建这个comment的post，并保存到数据库中 关联名
		create_post! 创建这个comment的post，存到数据库中,如果无效出错抛出ActiveRecord::RecordInvalid
		
		Post.join(:comment)  #like SQL inner join
		SELECT "Posts".* FROM "posts" INNER JOIN "comments" ON "comments"."id" = "posts"."comment_id"
		Event.joins(:category).where("categories.name is NOT NULL")
		Post.join(:comment,:location) #like inner multi join
		
		Event.includes(:category)  #like SQL left join
		
##自连接

		has_many :subordinates, class_name: 'Employee', foreign_key: 'manager_id'
		belongs_to :manager, class_name: 'Employee' class指向同一个类 
		
##多多连接
两个不同的方法
###has_and_belongs_to_many很少使用
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
		
###has_many :through      主要使用

		class Band < ActiveRecord::Base
		  has_many :performances
		  has_many :venues, through: :performances
		end
		
		class Performance < ActiveRecord::Base
		  belongs_to :band
		  belongs_to :venue
		end
		
##Single-Table单表继承
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
		
##polymorphic多态关联连接
一个模型归属于超过一个以上的模型

		class CreateComments < ActiveRecord::Migration
		  def change
		    create_table :comments do |t|
		      t.text :content
		      t.belongs_to :commentable, polymorphic: true
		      t.timestamps
		    end
		  end
		end
		class Post < ActiveRecord::Base
		  has_many :comments, as: :commentable
		end
		class Image < ActiveRecord::Base
		  has_many :comments, as: :commentable
		end
		class Comment < ActiveRecord::Base
		  belongs_to :commentable, polymorphic: true
		end
		comment = post.comments.create(content: "First comment")
		comment.commentable   #return Post id
		comment.commentable_type 
		
#transaction事务，multi action in one transaction	

		User.transaction do
		  User.create!(:name => 'ihower')
		  Feed.create!
		end
		
#Dirty objects 脏数据

	 	person.changed?   false  no change  true  changed
		person.name_changed?   attribute change? 
		person.name_was       Value before change
		person.name_change     value before and after change 

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
		

		

