---
layout: post
title: "Rails Ajax"
description: "Rails Ajax"
category: Rails
tags: [Rails Ajax]
---
{% include JB/setup %}

Don't refresh the page,exchage information and html through javascript

## request html fragment to server , then  JavaScript on browser to replace element on html 
##request javascript fragment to server , then excute on browser
##request JSON or XML to server, then JavaScript on browser to resolve

#Unobtrusive JavaScript (UJS)
---

seperate JavaScript from HTML

##Ajax form

		form_for @user, remote: true
		
##Ajax button

		button_to "ass", remote: true
		button_to "Remove", user_path(@user), :data => { :disable_with => 'Removing' }
		
#replace element on html
---
		
		<%= link_to 'Hello!', welcome_say_hello_path, :id => "ajax-load"  %>	<div id="content">
		</div>

		<script>
		    $(document).ready(function() {
		        $('#ajax-load').click( function(){
		            $('#content').load( $(this).attr("href") );
		            return false;
		        });

		    });
		</script>
		
#request javascript fragment to server  Rails default
---

		<%= link_to 'ajax show', event_path(event), :remote => true %>
		
		respond_to do |format|
		  format.html
		  format.js
		end

show.js.erb
		
		$('#event_area').html("<%= escape_javascript(render :partial => 'event') %>")
		             .css({ backgroundColor: '#ffff99' });
					 
#request JSON to server
---
in the Rails ,every object have to_json method

send ajax request

		<%= link_to 'ajax show', event_path(event), :remote => true, :data => { :type => :json }, :class => "ajax_update" %>
		
how to deal with json

		<script>
		$(document).ready(function() {
		    $('.ajax_update').on("ajax:success", function(event, data) {
		        var event_area = $('#event_area');
		        event_area.html( data.name );
		    });
		});
		</script>		
		
#Turbolinks
---
to make every hyperlink to use Ajax to replace the contents on the body

		 	$(document).on("page:change", function(){ ...})
			
	