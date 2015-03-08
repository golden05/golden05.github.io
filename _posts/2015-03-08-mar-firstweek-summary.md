---
layout: post
title: "mar firstweek summary"
description: "Mar firstweek summary"
category: Plan
tags: [Plan summary]

---
{% include JB/setup %}

start iquery project

setup devise and cancancan gem

#devise setup
##first do not use email as authentication_keys

change devise.rb

				config.authentication_keys = [ :employee_number ]
				config.case_insensitive_keys = [ :employee_number ]
				config.strip_whitespace_keys = [ :employee_number ]
				
change user.rb

				def email_required?
				  false
				end
				def	email_changed?
					false
				end
				
rails g devise:view to change email to employee_number

##second   before filter

				before_filter :configure_permitted_parameters, if: :devise_controller?
				
			  protected
			  def configure_permitted_parameters
			    devise_parameter_sanitizer.for(:sign_in) { |u| u.permit(:employee_number, :password )}
			    devise_parameter_sanitizer.for(:account_update) << :employee_number
			  end

insert controller 

				before_filter :authenticate_user!

#cancancan
rails g ability

## setup in the ability

        if user.admin?
          can :manage, :all
        elsif user.human?
          can :read, Payroll
        elsif user.employee?
          can :read, Payroll, :user_id => user.id
        end
				
## user.rb add role

				ROLES = %i[admin human employee]
				
			  def admin?
			    self.role == "admin"
			  end

			  def human?
			    self.role == "human"
			  end

			  def employee?
			    self.role == "employee"
			  end
				
##third in the payrollsController

			load_and_authorize_resource
			
			
#next week payroll 