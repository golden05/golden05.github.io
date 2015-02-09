---
layout: post
title: "iQuery project"
description: "iQuery project"
category: Project
tags: [iQuery]
---
{% include JB/setup %}

#scene
---
##an unlogin employee logon system

		Given I have not login in
		when I open the login page
		and I input empoyee id
		and I input employee password
		and I press login button
		then I login system 
		
##a logined employee as employee
###can query salary

		Given I logon system 
		when I choose the year, default is last year
		and I choose the classify, default is all classify
		and I press the query button
		then I can see the month,any classify,all salary item
		
###can query detail salary

		Given I logon system
		and I can query salary
		when I choose the month 
		and I choose the classify
		and I press the query button
		then I can see the month,the classify detail salary
		
###can change password

		Given I logon system
		when I open the change password page
		and I input the old password
		and I input the new password
		and I input the new confirm_password
		and I press the change password button
		then I can see the password changed successfuly
		
##a logined employee as assistant role

		Given I logon system
		and I have assistant role

###can query total salary

		Given a logined employee as assistant role
		when I open query total salary page
		and I choose the year default is last year
		and I choose the classify default is all classify
		and I press the query button
		then I can see the total salary the month, classify 
		
###can query all employee salary

		Given can query total salary
		when I choose the month 
		and I choose the classify
		and I press the query button
		then I can see the department,month,classify salary
		
###can load data

		Given temporary data is empty
		when I open load data page
		and I press the open button
		and I choose the data file
		and I press the load button
		then I can see the month,classify,department,salary
		when I press the confirm load button
		then I can see load data successfully
		when I press the empty button
		then I can see temporary data is empty
		
##a logined employee as admin

		Given I logon system
		and I have admin role
		
###can admin employee

		when I open admin employee page
		and I press the add button
		and I input the employee id
		and I input the employee name
		and I input the employee password
		and I press the submit button
		then I can see the new employee have added successfully
		
###can admin role

		when I open admin role page
		and I choose the assistant role
		and I can see the assistant role employee
		and I press the add button
		and I choose the employee
		and I press the submit button
		then I can see the employee have added role
		
###can admin foundation data
	
		when I open admin foundation page
		then I can see the department
		and I can see the classify
		and I can see the salary item
		and I can see the date

#require domain conception
##employee

		id,name,CitizenNumber,department_id,pinyin
		
##department

		id,name,belong_department_id
		
##classify

		id,name
		
##date

		id,year
		id,month
		
##salary item 

		id,name
		
##clssify salary

		id,classify_id,salaryItem_id
		
##role

		id,name
		
##ability

		id,name,role_id