---
layout: post
title: "Bootstrap"
description: "Bootstrap"
category: Rails Bootstrap
tags: [Bootstrap]
---
{% include JB/setup %}

#setup
---

		gem 'bootstrap-sass'        #in the GEM file
		application.css            # do not change anything
		
		@import "bootstrap-sprockets";    #setup in the custom.css.scss
		@import "bootstrap";             # and any global css setup
		
#css overview
---
##first mobile device

		<meta name="viewport" content="width=device-width, initial-scale=1"> 
		
##Typography and links
set global display , typography and link styles 

###background-color on the body
###@font-family-base, @font-size-base, and @line-height-base 
###global link color @link-color  underline :hover
		
##container  two 
	
		<div class="container">    fixed width and Responsive layout
		<div class="container-fluid"   100% width 
	
#grid system
---
##introduce
###row must in the container or container-fluid  .row 
###column in the row  .col-xs-4
###content in the column
###padding margin in the column 
##media query

		@media (max-width: @screen-xs-max) { ... }    mobile phone
		@media (min-width: @screen-sm-min) and (max-width: @screen-sm-max) { ... } tablet pad
		@media (min-width: @screen-md-min) and (max-width: @screen-md-max) { ... } desktop
		@media (min-width: @screen-lg-min) { ... }  big desktop
		
### mobile phone xs .col-xs-1  .col-xs-2
### tablet sm  .col-sm-1  .col-sm-2
### desktop md  .col-md-1  .col-md-2
### big desktop lg  .col-lg-1   .col-lg-2

		<div class="row">     multi device   mobile phone and tablet 
		  <div class="col-xs-12 col-md-8">.col-xs-12 .col-md-8</div>
		  <div class="col-xs-6 col-md-4">.col-xs-6 .col-md-4</div>
		</div> 

###grid parameter

		<div class="row">
		  <div class="col-md-1">.col-md-1</div>
		  <div class="col-md-1">.col-md-1</div>
		  <div class="col-md-1">.col-md-1</div>
		  <div class="col-md-1">.col-md-1</div>
		  <div class="col-md-1">.col-md-1</div>
		  <div class="col-md-1">.col-md-1</div>
		  <div class="col-md-1">.col-md-1</div>
		  <div class="col-md-1">.col-md-1</div>
		  <div class="col-md-1">.col-md-1</div>
		  <div class="col-md-1">.col-md-1</div>
		  <div class="col-md-1">.col-md-1</div>
		  <div class="col-md-1">.col-md-1</div>
		</div>
		<div class="row">
		  <div class="col-md-8">.col-md-8</div>
		  <div class="col-md-4">.col-md-4</div>
		</div>
		<div class="row">
		  <div class="col-md-4">.col-md-4</div>
		  <div class="col-md-4">.col-md-4</div>
		  <div class="col-md-4">.col-md-4</div>
		</div>
		<div class="row">
		  <div class="col-md-6">.col-md-6</div>
		  <div class="col-md-6">.col-md-6</div>
		</div>
		
###column offset
.col-md-offset-*  use offset turn column right

		 <div class="col-md-4 col-md-offset-4">.col-md-4 .col-md-offset-4</div>
		 
###nesting column
add .row and .col-sm-*  in existing .col-sm-* column

		<div class="row">
		  <div class="col-sm-9">
		    Level 1: .col-sm-9
		    <div class="row">
		      <div class="col-xs-8 col-sm-6">
		        Level 2: .col-xs-8 .col-sm-6
		      </div>
		      <div class="col-xs-4 col-sm-6">
		        Level 2: .col-xs-4 .col-sm-6
		      </div>
		    </div>
		  </div>
		</div>
		
###column order
use .col-md-push-* and .col-md-pull-* change column order
pull first    and  push  last

		<div class="row">
		  <div class="col-md-9 col-md-push-3">.col-md-9 .col-md-push-3</div>
		  <div class="col-md-3 col-md-pull-9">.col-md-3 .col-md-pull-9</div>
		</div>  

		.col-md-3 .col-md-pull-9 .col-md-9 .col-md-push-3    ＃output
		
#Typography 排版
---
##Headings

		<h1> <h2>  <small> sub head

##body
font-size   line-height  @font-size-base  @line-height-base 
###.lead   outstand content

		<mark>xxx </mark>  highlighting xxx
		<del> xxx  </del>  add a delete line
		<s>   xxx   </s>   like <del>
		<ins>  xxx  </ins>  add  down line to present insert
		<small>   xxxx</small>     font size smaller
		<strong>   xxxx</strong>   font size bold
		<em>  xxxx  </em>     font italic
		.text-left .text-center .text-right .text-justify .text-nowrap  # text alignment 
		.text-lowercase .text-uppercase .text-capitalize
		<abbr title="attribute">attr</abbr>   #abbreviation
		<abbr title="HyperText Markup Language" class="initialism">HTML</abbr> 
		<address>
		<blockquote>   reference 
		<footer> <cite>  </cite></footer>

###list

		<ul><li>     list unorder
		<ol><li>     list  order
		<dl><dt><dd>   .dl-horizontal    description 
		
###code

		<code>   source 
		<kbd>    meaning keyword input
		<pre>     code
		<var>    meaning variable
		<samp>   meaning program output

###table

		<table>  .table   meaning horizontal separate line 
		.table table-striped  meaning horizontal zebra separate line
		.table table-bordered   meaning table with border
		.table table-hover   meaning mouse hover
		.table table-condensed  meaning 紧凑 
		
		<tr class="active">...</tr>   colour
		<tr class="success">...</tr>
		<tr class="warning">...</tr>
		<tr class="danger">...</tr>
		<tr class="info">...</tr> 
		
		.table-responsive  put .table in the .table-responsive create responsive table
		
###form
form element input textarea and select
	
		.form-group any element in .form-group good arrange
		.form-control    will set width 100%
		
####inline form 

		<form class="form-inline">  will set width auto
		
####horizontal arrange form

		<form class="form-horizontal"> put any .form-group in the .form-horizontal to horizontal arrange
			 <div class="form-group">
			 
####supported control
class="form-control"

#####inputs 
text password textarea datetime datetimelocal date month time ...

		<input type="text" class="form-control" placeholder="Text input"> 

#####Textarea

		<textarea class="form-control" rows="3"></textarea>

#####checkbox radio 
.checkbox-inline choose multi  .radio-inline choose one

		<div class="checkbox">
		  <label>
		    <input type="checkbox" value="">
		    Option one is this and that&mdash;be sure to include why it's great
		  </label>
		</div>
		<div class="checkbox disabled">
		  <label>
		    <input type="checkbox" value="" disabled>
		    Option two is disabled
		  </label>
		</div>
		<div class="radio">
		  <label>
		    <input type="radio" name="optionsRadios" id="optionsRadios1" value="option1" checked>
		    Option one is this and that&mdash;be sure to include why it's great
		  </label>
		</div>
		<div class="radio">
		  <label>
		    <input type="radio" name="optionsRadios" id="optionsRadios2" value="option2">
		    Option two can be something else and selecting it will deselect option one
		  </label>
		</div>
		
.checkbox-inline 或 .radio-inline

		<label class="checkbox-inline">
		  <input type="checkbox" id="inlineCheckbox1" value="option1"> 1
		</label>
		<label class="checkbox-inline">
		  <input type="checkbox" id="inlineCheckbox2" value="option2"> 2
		</label>
		<label class="checkbox-inline">
		  <input type="checkbox" id="inlineCheckbox3" value="option3"> 3
		</label>

		<label class="radio-inline">
		  <input type="radio" name="inlineRadioOptions" id="inlineRadio1" value="option1"> 1
		</label>
		<label class="radio-inline">
		  <input type="radio" name="inlineRadioOptions" id="inlineRadio2" value="option2"> 2
		</label>
		<label class="radio-inline">
		  <input type="radio" name="inlineRadioOptions" id="inlineRadio3" value="option3"> 3
		</label>
		
#####select 
		
		<select class="form-control">      #only choose
		  <option>1</option>
		  <option>2</option>
		  <option>3</option>
		  <option>4</option>
		  <option>5</option>
		</select>
		
		<select multiple class="form-control">   #multi choose
		  <option>1</option>
		  <option>2</option>
		  <option>3</option>
		  <option>4</option>
		  <option>5</option>
		</select>
		
#####static control  <p class="form-control-static">

		<p class="form-control-static">email@example.com</p>

#####focus status  :focus box-shadow

#####disable status

		<fieldset disabled>   disable field
	
#####input readonly 

		<input class="form-control" type="text" placeholder="Readonly input here…" readonly>
		
checking status

put these class .has-warning、.has-error 或 .has-success in the parent element

		<div class="form-group has-success">
		  <label class="control-label" for="inputSuccess1">Input with success</label>
		  <input type="text" class="form-control" id="inputSuccess1">
		</div>
		
####control size
height by like .input-lg , width by like .col-lg-*

#####height size

		<input class="form-control input-lg" type="text" placeholder=".input-lg">
		<input class="form-control" type="text" placeholder="Default input">
		<input class="form-control input-sm" type="text" placeholder=".input-sm">
		
#####horizontal arrange form group size
put .form-group-lg or  .form-group-sm in the .form-horizontal parent form element

		<form class="form-horizontal">
		  <div class="form-group form-group-lg">
		    <label class="col-sm-2 control-label" for="formGroupInputLarge">Large label</label>
		    <div class="col-sm-10">
		      <input class="form-control" type="text" id="formGroupInputLarge" placeholder="Large input">
		    </div>
		  </div>
		  <div class="form-group form-group-sm">
		    <label class="col-sm-2 control-label" for="formGroupInputSmall">Small label</label>
		    <div class="col-sm-10">
		      <input class="form-control" type="text" id="formGroupInputSmall" placeholder="Small input">
		    </div>
		  </div>
		</form>
		
#####adjust column  width size

		<div class="col-xs-2">
		    <input type="text" class="form-control" placeholder=".col-xs-2">
		</div>
		
#Buttons
##Button tags

		<a class="btn btn-default" href="#" role="button">Link</a>
		<button class="btn btn-default" type="submit">Button</button>
		<input class="btn btn-default" type="button" value="Input">

##Options

		<!-- Standard button -->
		<button type="button" class="btn btn-default">Default</button>

		<!-- Provides extra visual weight and identifies the primary action in a set of buttons -->
		<button type="button" class="btn btn-primary">Primary</button>

		<!-- Indicates a successful or positive action -->
		<button type="button" class="btn btn-success">Success</button>

		<!-- Contextual button for informational alert messages -->
		<button type="button" class="btn btn-info">Info</button>

		<!-- Indicates caution should be taken with this action -->
		<button type="button" class="btn btn-warning">Warning</button>

		<!-- Indicates a dangerous or potentially negative action -->
		<button type="button" class="btn btn-danger">Danger</button>

		<!-- Deemphasize a button by making it look like a link while maintaining button behavior -->
		<button type="button" class="btn btn-link">Link</button>
		
##Sizes
.btn-lg, .btn-sm, or .btn-xs

		<button type="button" class="btn btn-primary btn-lg">（大按钮）Large button</button>

.btn-block  become to block element and the width extend parent width
 		
		<button type="button" class="btn btn-primary btn-lg btn-block">（块级元素）Block level button</button>

##Active state
###button 

		<button type="button" class="btn btn-default btn-lg active">Button</button>
		
###link element

		<a href="#" class="btn btn-default btn-lg active" role="button">Link</a>
		
##Disabled state
###Button element

		<button type="button" class="btn btn-default btn-lg" disabled="disabled">Button</button>
		
###link element

		<a href="#" class="btn btn-default btn-lg disabled" role="button">Link</a>
		
#Img
---
##responsive img
.img-responsive

##image shapes
.img-rounded  .img-circle  .img-thumbnail

		<img src="..." alt="..." class="img-rounded">
		
#Helper class
##Contextual colors
can change color when mouse hover on it

		<p class="text-muted">...</p>
		<p class="text-primary">...</p>
		<p class="text-success">...</p>
		<p class="text-info">...</p>
		<p class="text-warning">...</p>
		<p class="text-danger">...</p>
		
##Contextual backgroundColors
can deepen color when mouse hover on it

		<p class="bg-primary">...</p>
		<p class="bg-success">...</p>
		<p class="bg-info">...</p>
		<p class="bg-warning">...</p>
		<p class="bg-danger">...</p>

##close icon
				
		<button type="button" class="close" aria-label="Close"><span aria-hidden="true">&times;</span></button>
		
##Carets 三角符号
have dropdown menu

		<span class="caret"></span>
		
##quick floats 快速浮动
make element turn to left or right 

		<div class="pull-left">...</div>
		<div class="pull-right">...</div>
		// Classes
		.pull-left {
		  float: left !important;
		}
		.pull-right {
		  float: right !important;
		}
		
###navbar

		.navbar-left or .navbar-right
		
##Center content block 内容块居中

		<div class="center-block">...</div>
		// Classes
		.center-block {
		  display: block;
		  margin-left: auto;
		  margin-right: auto;
		}

		// Usage as mixins
		.element {
		  .center-block();
		}

##clear float 
put .clearfix in parent element to clear float

		<div class="clearfix">...</div>
		
##show or hide content
.show 和 .hidden  to force  any element 

##screen reader and keybord nav

		<a class="sr-only sr-only-focusable" href="#content">Skip to main content</a>
		
##img instead text

		<h1 class="text-hide">Custom heading</h1>
		
#responsive utilites
##ability class
.visible-*-block  .visible-*-inline  .visible-*-inline-block

##printer class
.visable-print-block .visable-print-inline .visable-print-block

##test class
.visible .hidden

#Using Less
---
##variables
###colors
two color schemes : grayscale 灰度 and semantic 语义
		
		@gray-darker:  lighten(#000, 13.5%); // #222
		@gray-dark:    lighten(#000, 20%);   // #333
		
		@brand-primary: darken(#428bca, 6.5%); // #337ab7
		@brand-success: #5cb85c;
		
		// Use as-is
		.masthead {
		  background-color: @brand-primary;
		}

###Scaffolding
quick custom set site skeleton

		// Scaffolding
		@body-bg:    #fff;
		@text-color: @black-50;
		
###Links
color 
		// Variables
		@link-color:       @brand-primary;
		@link-hover-color: darken(@link-color, 15%);

		// Usage
		a {
		  color: @link-color;
		  text-decoration: none;

		  &:hover {
		    color: @link-hover-color;
		    text-decoration: underline;
		  }
		}
		
###Typography

		@font-family-sans-serif:  "Helvetica Neue", Helvetica, Arial, sans-serif;
		@font-family-serif:       Georgia, "Times New Roman", Times, serif;
		@font-family-monospace:   Menlo, Monaco, Consolas, "Courier New", monospace;
		@font-family-base:        @font-family-sans-serif;

		@font-size-base:          14px;
		@font-size-large:         ceil((@font-size-base * 1.25)); // ~18px
		@font-size-small:         ceil((@font-size-base * 0.85)); // ~12px

		@font-size-h1:            floor((@font-size-base * 2.6)); // ~36px
		@font-size-h2:            floor((@font-size-base * 2.15)); // ~30px
		@font-size-h3:            ceil((@font-size-base * 1.7)); // ~24px
		@font-size-h4:            ceil((@font-size-base * 1.25)); // ~18px
		@font-size-h5:            @font-size-base;
		@font-size-h6:            ceil((@font-size-base * 0.85)); // ~12px

		@line-height-base:        1.428571429; // 20/14
		@line-height-computed:    floor((@font-size-base * @line-height-base)); // ~20px

		@headings-font-family:    inherit;
		@headings-font-weight:    500;
		@headings-line-height:    1.1;
		@headings-color:          inherit;
		
###Icons 
two quick vaiiables  customizing the location and filename of your icons

		@icon-font-path:          "../fonts/";
		@icon-font-name:          "glyphicons-halflings-regular";
		
###Components

		@padding-base-vertical:          6px;
		@padding-base-horizontal:        12px;

		@padding-large-vertical:         10px;
		@padding-large-horizontal:       16px;
		
##Vendor mixins
###Box-sizing 

		.box-sizing(@box-model) {
		  -webkit-box-sizing: @box-model; // Safari <= 5
		     -moz-box-sizing: @box-model; // Firefox <= 19
		          box-sizing: @box-model;
		}
		
###Rounded corners

		.border-top-radius(@radius) {
		  border-top-right-radius: @radius;
		   border-top-left-radius: @radius;
		}
		.border-right-radius(@radius) {
		  border-bottom-right-radius: @radius;
		     border-top-right-radius: @radius;
		}
		.border-bottom-radius(@radius) {
		  border-bottom-right-radius: @radius;
		   border-bottom-left-radius: @radius;
		}
		.border-left-radius(@radius) {
		  border-bottom-left-radius: @radius;
		     border-top-left-radius: @radius;
		}
		
###Box (Drop) shadows

		.box-shadow(@shadow: 0 1px 3px rgba(0,0,0,.25)) {
		  -webkit-box-shadow: @shadow; // iOS <4.3 & Android <4.1
		          box-shadow: @shadow;
		}
		
###Transitions

		.transition(@transition) {
		  -webkit-transition: @transition;
		          transition: @transition;
		}
		.transition-property(@transition-property) {
		  -webkit-transition-property: @transition-property;
		          transition-property: @transition-property;
		}
		.transition-delay(@transition-delay) {
		  -webkit-transition-delay: @transition-delay;
		          transition-delay: @transition-delay;
		}
		.transition-duration(@transition-duration) {
		  -webkit-transition-duration: @transition-duration;
		          transition-duration: @transition-duration;
		}
		.transition-timing-function(@timing-function) {
		  -webkit-transition-timing-function: @timing-function;
		          transition-timing-function: @timing-function;
		}
		.transition-transform(@transition) {
		  -webkit-transition: -webkit-transform @transition;
		     -moz-transition: -moz-transform @transition;
		       -o-transition: -o-transform @transition;
		          transition: transform @transition;
		}
		
###Transformations
Rotate, scale, translate (move), or skew any object.

		.rotate(@degrees) {
		  -webkit-transform: rotate(@degrees);
		      -ms-transform: rotate(@degrees); // IE9 only
		          transform: rotate(@degrees);
		}
		.scale(@ratio; @ratio-y...) {
		  -webkit-transform: scale(@ratio, @ratio-y);
		      -ms-transform: scale(@ratio, @ratio-y); // IE9 only
		          transform: scale(@ratio, @ratio-y);
		}
		.translate(@x; @y) {
		  -webkit-transform: translate(@x, @y);
		      -ms-transform: translate(@x, @y); // IE9 only
		          transform: translate(@x, @y);
		}
		.skew(@x; @y) {
		  -webkit-transform: skew(@x, @y);
		      -ms-transform: skewX(@x) skewY(@y); 

###Animations

		.animation(@animation) {
		  -webkit-animation: @animation;
		          animation: @animation;
		}
		.animation-name(@name) {
		  -webkit-animation-name: @name;
		          animation-name: @name;
		}
		
###Opacity

		.opacity(@opacity) {
		  opacity: @opacity;
		  // IE8 filter
		  @opacity-ie: (@opacity * 100);
		  filter: ~"alpha(opacity=@{opacity-ie})";
		}
		
###Placeholder text

		.placeholder(@color: @input-color-placeholder) {
		  &::-moz-placeholder           { color: @color; } // Firefox
		  &::-webkit-input-placeholder  { color: @color; } // Safari and Chrome
		}

###Columns

		.content-columns(@width; @count; @gap) {
		  -webkit-column-width: @width;
		     -moz-column-width: @width;
		          column-width: @width;
		  -webkit-column-count: @count;
		     -moz-column-count: @count;
		          column-count: @count;
		  -webkit-column-gap: @gap;
		     -moz-column-gap: @gap;
		          column-gap: @gap;
		}
		
###Gradients
turn any two colors into a background gradient

		#gradient > .vertical(#333; #000);
		#gradient > .horizontal(#333; #000);
		#gradient > .radial(#333; #000);

##utitlty mixins
###Clearfix

		// Mixin
		.clearfix() {
		  &:before,
		  &:after {
		    content: " ";
		    display: table;
		  }
		  &:after {
		    clear: both;
		  }
		}

		// Usage
		.container {
		  .clearfix();
		}
		
###Horizontal centering

		// Mixin
		.center-block() {
		  display: block;
		  margin-left: auto;
		  margin-right: auto;
		}

		// Usage
		.container {
		  width: 940px;
		  .center-block();
		}
		
###Sizing helpers

		// Mixins
		.size(@width; @height) {
		  width: @width;
		  height: @height;
		}
		.square(@size) {
		  .size(@size; @size);
		}

		// Usage
		.image { .size(400px; 300px); }
		.avatar { .square(48px); }
		
###Resizable textareas

		.resizable(@direction: both) {
		  // Options: horizontal, vertical, both
		  resize: @direction;
		  // Safari fix
		  overflow: auto;
		}
		
###Truncating text

		// Mixin
		.text-overflow() {
		  overflow: hidden;
		  text-overflow: ellipsis;
		  white-space: nowrap;
		}

		// Usage
		.branch-name {
		  display: inline-block;
		  max-width: 200px;
		  .text-overflow();
		}
		
###Retina images

		.img-retina(@file-1x; @file-2x; @width-1x; @height-1x) {
		  background-image: url("@{file-1x}");

		  @media
		  only screen and (-webkit-min-device-pixel-ratio: 2),
		  only screen and (   min--moz-device-pixel-ratio: 2),
		  only screen and (     -o-min-device-pixel-ratio: 2/1),
		  only screen and (        min-device-pixel-ratio: 2),
		  only screen and (                min-resolution: 192dpi),
		  only screen and (                min-resolution: 2dppx) {
		    background-image: url("@{file-2x}");
		    background-size: @width-1x @height-1x;
		  }
		}

		// Usage
		.jumbotron {
		  .img-retina("/img/bg-1x.png", "/img/bg-2x.png", 100px, 100px);
		}
		
#Glyphicons 字体图标
---
http://v3.bootcss.com/components/  allow bootstrape free use

##how to use
need a base class with individal icon class

Don't mix with other components  in <span>
	
Only for use on empty elements

Changing the icon font location

		<span class="glyphicon glyphicon-search" aria-hidden="true"></span>
		
##examples

		<button type="button" class="btn btn-default" aria-label="Left Align">
		  <span class="glyphicon glyphicon-align-left" aria-hidden="true"></span>
		</button>

		<button type="button" class="btn btn-default btn-lg">
		  <span class="glyphicon glyphicon-star" aria-hidden="true"></span> Star
		</button>
		
#dropdown menu
---
##examples
put dropdown trigger and dropdown menu include in .dropdown or positive: relative

		<div class="dropdown">
		  <button class="btn btn-default dropdown-toggle" type="button" id="dropdownMenu1" data-toggle="dropdown" aria-expanded="true">
		    Dropdown
		    <span class="caret"></span>
		  </button>
		  <ul class="dropdown-menu" role="menu" aria-labelledby="dropdownMenu1">
		    <li role="presentation"><a role="menuitem" tabindex="-1" href="#">Action</a></li>
		    <li role="presentation"><a role="menuitem" tabindex="-1" href="#">Another action</a></li>
		    <li role="presentation"><a role="menuitem" tabindex="-1" href="#">Something else here</a></li>
		    <li role="presentation"><a role="menuitem" tabindex="-1" href="#">Separated link</a></li>
		  </ul>
		</div>
		
##Alignment
default along with parent element
	
		.dropdown-menu-left
		
		<ul class="dropdown-menu dropdown-menu-right" role="menu" aria-labelledby="dLabel">
		  ...
		</ul>
		
##Headers
class="dropdown-header"

##Divider
in the dropdown-menu
		
		<ul class="dropdown-menu" role="menu" aria-labelledby="dropdownMenuDivider">
		  ...
		  <li role="presentation" class="divider"></li>
		  ...
		</ul>
		
##Disabled menu item

		<li role="presentation" class="disabled"><a role="menuitem" tabindex="-1" href="#">Disabled link</a></li>
		
#Button Group
---
##Basic Examples
wrap a series of buttons with .btn and .btn-group

		<div class="btn-group" role="group" aria-label="...">
		  <button type="button" class="btn btn-default">Left</button>
		  <button type="button" class="btn btn-default">Middle</button>
		  <button type="button" class="btn btn-default">Right</button>
		</div>
		
##Button toolbars

		<div class="btn-toolbar" role="toolbar" aria-label="...">
		  <div class="btn-group" role="group" aria-label="...">...</div>
		  <div class="btn-group" role="group" aria-label="...">...</div>
		  <div class="btn-group" role="group" aria-label="...">...</div>
		</div>
		
##Sizes
put .btn-group-* in .btn-group

		<div class="btn-group btn-group-lg" role="group" aria-label="...">...</div>
		<div class="btn-group" role="group" aria-label="...">...</div>
		<div class="btn-group btn-group-sm" role="group" aria-label="...">...</div>
		<div class="btn-group btn-group-xs" role="group" aria-label="...">...</div>
		
##Nestings
put dropdown menu in button, just nesting .btn-group

##Vertical variation

		<div class="btn-group-vertical" role="group" aria-label="...">
		  ...
		</div>
		
##Justified button groups  两端对齐
###About <a>
put .btn element into .btn-group .btn-group-justified

		<div class="btn-group btn-group-justified" role="group" aria-label="...">
		  ...
		</div>
		
###About <button>
put each button in a button group 

		<div class="btn-group btn-group-justified" role="group" aria-label="...">
		  <div class="btn-group" role="group">
		    <button type="button" class="btn btn-default">Left</button>
		  </div>
		  <div class="btn-group" role="group">
		    <button type="button" class="btn btn-default">Middle</button>
		  </div>
		  <div class="btn-group" role="group">
		    <button type="button" class="btn btn-default">Right</button>
		  </div>
		</div>
		
#Dropdown menu button style
put any button in .btn-group 
##single button dropdown menu

		<!-- Single button -->
		<div class="btn-group">
		  <button type="button" class="btn btn-default dropdown-toggle" data-toggle="dropdown" aria-expanded="false">
		    Action <span class="caret"></span>
		  </button>
		  <ul class="dropdown-menu" role="menu">
		    <li><a href="#">Action</a></li>
		    <li><a href="#">Another action</a></li>
		    <li><a href="#">Something else here</a></li>
		    <li class="divider"></li>
		    <li><a href="#">Separated link</a></li>
		  </ul>
		</div>
		
##Split button dropdown menu
##sizes
		
		btn btn-default btn-lg
		
##Dropup menu

		<div class="btn-group dropup">
		
#Input
##Basic examples
anyside or bothside add button or element

		<div class="input-group">
		  <span class="input-group-addon" id="basic-addon1">@</span>
		  <input type="text" class="form-control" placeholder="Username" aria-describedby="basic-addon1">
		</div>
		
##size

		<div class="input-group input-group-lg">
		  <span class="input-group-addon" id="sizing-addon1">@</span>
		  <input type="text" class="form-control" placeholder="Username" aria-describedby="sizing-addon1">
		</div>
		
##checkbox and radio addons

		<div class="input-group">
		      <span class="input-group-addon">
		        <input type="checkbox" aria-label="...">
		      </span>
		      <input type="text" class="form-control" aria-label="...">
		</div><!-- /input-group -->
		
##Button addons

		<div class="input-group">
		      <span class="input-group-btn">
		        <button class="btn btn-default" type="button">Go!</button>
		      </span>
		      <input type="text" class="form-control" placeholder="Search for...">
		</div>
			
##dropdown menu addons

		<div class="input-group">
		      <div class="input-group-btn">
		        <button type="button" class="btn btn-default dropdown-toggle" data-toggle="dropdown" aria-expanded="false">Action <span class="caret"></span></button>
		        <ul class="dropdown-menu" role="menu">
		          <li><a href="#">Action</a></li>
		          <li><a href="#">Another action</a></li>
		          <li><a href="#">Something else here</a></li>
		          <li class="divider"></li>
		          <li><a href="#">Separated link</a></li>
		        </ul>
		      </div><!-- /btn-group -->
		      <input type="text" class="form-control" aria-label="...">
		 </div><!-- /input-group -->
		 
##split button dropdown menu addons

		<div class="input-group">
		  <div class="input-group-btn">
		    <!-- Button and dropdown menu -->
		  </div>
		  <input type="text" class="form-control" aria-label="...">
		</div>
		
#Navs
---
base class .nav

##tab page
		
		<ul class="nav nav-tabs">
		  <li role="presentation" class="active"><a href="#">Home</a></li>
		  <li role="presentation"><a href="#">Profile</a></li>
		  <li role="presentation"><a href="#">Messages</a></li>
		</ul>
		
##Pill page 胶囊导航

		<ul class="nav nav-pills">
		  <li role="presentation" class="active"><a href="#">Home</a></li>
		  <li role="presentation"><a href="#">Profile</a></li>
		  <li role="presentation"><a href="#">Messages</a></li>
		</ul>
		
##Justified 两边对齐

		<ul class="nav nav-tabs nav-justified">
		  ...
		</ul>
		
##Disabled link

		<ul class="nav nav-pills">
		  ...
		  <li role="presentation" class="disabled"><a href="#">Disabled link</a></li>
		  ...
		</ul>
		
##add dropdown menu
###tab page with dropdown menu

		<ul class="nav nav-tabs">
		  <li role="presentation" class="dropdown">
		    <a class="dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-expanded="false">
		      Dropdown <span class="caret"></span>
		    </a>
		    <ul class="dropdown-menu" role="menu">
		      ...
		    </ul>
		  </li>
		</ul>
		
###pills page with dropdown menu

		<ul class="nav nav-pills">
		  <li role="presentation" class="dropdown">
		    <a class="dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-expanded="false">
		      Dropdown <span class="caret"></span>
		    </a>
		    <ul class="dropdown-menu" role="menu">
		      ...
		    </ul>
		  </li>
		</ul>

#Navbar
---
##default style 
		
		<nav class="navbar navbar-default">
		
##Brand image  品牌图标
		
		<nav class="navbar navbar-default">
		  <div class="container-fluid">
		    <div class="navbar-header">
		      <a class="navbar-brand" href="#">
		        <img alt="Brand" src="...">
		      </a>
		    </div>
		  </div>
		</nav>

##Forms

		<form class="navbar-form navbar-left" role="search">
		  <div class="form-group">
		    <input type="text" class="form-control" placeholder="Search">
		  </div>
		  <button type="submit" class="btn btn-default">Submit</button>
		</form>
		
##Buttons

		<button type="button" class="btn btn-default navbar-btn">Sign in</button>
	
##Text 

		<p class="navbar-text">Signed in as Mark Otto</p>
		
##No-nav links

		<p class="navbar-text navbar-right">Signed in as <a href="#" class="navbar-link">Mark Otto</a></p>
		
##Component alignment
.navbar-left or .navbar-right 
###Fixed top   don't disappear
.navbar-fixed-top have subelement .container 或 .container-fluid 


###Fixed bottom  don't disappear
.navbar-fixed-bottom have subelement .container 或 .container-fluid

###Static top  disappear with nav bottom
.navbar-static-top

###Inverted navbar 反色
.navbar-inverse

#Breadcrumbs 
---
		
		<ol class="breadcrumb">
		  <li><a href="#">Home</a></li>
		  <li><a href="#">Library</a></li>
		  <li class="active">Data</li>
		</ol>
		
#Pagination
---
##default Pagination

		<nav>
		  <ul class="pagination">
		    <li>
		      <a href="#" aria-label="Previous">
		        <span aria-hidden="true">&laquo;</span>
		      </a>
		    </li>
		    <li><a href="#">1</a></li>
		    <li><a href="#">2</a></li>
		    <li><a href="#">3</a></li>
		    <li><a href="#">4</a></li>
		    <li><a href="#">5</a></li>
		    <li>
		      <a href="#" aria-label="Next">
		        <span aria-hidden="true">&raquo;</span>
		      </a>
		    </li>
		  </ul>
		</nav>
		
###disable and active status

		<nav>
		  <ul class="pagination">
		    <li class="disabled"><a href="#" aria-label="Previous"><span aria-hidden="true">&laquo;</span></a></li>
		    <li class="active"><a href="#">1 <span class="sr-only">(current)</span></a></li>
		    ...
		  </ul>
		</nav>
		
###sizes
.pagination-lg 或 .pagination-sm 

##Pager 翻页
###Default example

		<nav>
		  <ul class="pager">
		    <li><a href="#">Previous</a></li>
		    <li><a href="#">Next</a></li>
		  </ul>
		</nav>
		
###Aligned links

		<nav>
		  <ul class="pager">
		    <li class="previous"><a href="#"><span aria-hidden="true">&larr;</span> Older</a></li>
		    <li class="next"><a href="#">Newer <span aria-hidden="true">&rarr;</span></a></li>
		  </ul>
		</nav>
		
###Optional disabled state

		<nav>
		  <ul class="pager">
		    <li class="previous disabled"><a href="#"><span aria-hidden="true">&larr;</span> Older</a></li>
		    <li class="next"><a href="#">Newer <span aria-hidden="true">&rarr;</span></a></li>
		  </ul>
		</nav>
		
#Labels
---
##Examples

		<h3>Example heading <span class="label label-default">New</span></h3>
		
##Available variations

		<span class="label label-default">Default</span>
		<span class="label label-primary">Primary</span>
		<span class="label label-success">Success</span>
		<span class="label label-info">Info</span>
		<span class="label label-warning">Warning</span>
		<span class="label label-danger">Danger</span>
		
#Badges 徽章
---
link or button or nav element nesting badges

		<a href="#">Inbox <span class="badge">42</span></a>

		<button class="btn btn-primary" type="button">
		  Messages <span class="badge">4</span>
		</button>
		
##Self collapsing 
if empty can hidden CSS :empty

##adapt nav element active status

		<ul class="nav nav-pills" role="tablist">
		  <li role="presentation" class="active"><a href="#">Home <span class="badge">42</span></a></li>
		  <li role="presentation"><a href="#">Profile</a></li>
		  <li role="presentation"><a href="#">Messages <span class="badge">3</span></a></li>
		</ul>

#Jumbotron 巨幕
---

		<div class="jumbotron">
		  <h1>Hello, world!</h1>
		  <p>...</p>
		  <p><a class="btn btn-primary btn-lg" href="#" role="button">Learn more</a></p>
		</div>
		
#Page header
---
to H1 add appropriately space 

		<div class="page-header">
		  <h1>Example page header <small>Subtext for header</small></h1>
		</div>
	
#Thumbnails
---
			
		 <a href="#" class="thumbnail">
		 
#Alerts
---

		<div class="alert alert-success" role="alert">...</div>
		<div class="alert alert-info" role="alert">...</div>
		<div class="alert alert-warning" role="alert">...</div>
		<div class="alert alert-danger" role="alert">...</div>
		
		<div class="alert alert-danger" role="alert">
		  <a href="#" class="alert-link">...</a>
		</div>
		
#Progress bars
---

		<div class="progress">
		  <div class="progress-bar" role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100" style="width: 60%;">
		    <span class="sr-only">60% Complete</span>
		  </div>
		</div>
		
		<div class="progress">
		  <div class="progress-bar" role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100" style="width: 60%;">
		    60%
		  </div>
		</div>
		
#media object
---

		<div class="media">
		  <div class="media-left">
		    <a href="#">
		      <img class="media-object" src="..." alt="...">
		    </a>
		  </div>
		  <div class="media-body">
		    <h4 class="media-heading">Media heading</h4>
		    ...
		  </div>
		</div>
		
##Media alignment

		<div class="media">
		  <div class="media-left media-middle">
		    <a href="#">
		      <img class="media-object" src="..." alt="...">
		    </a>
		  </div>
		  
#List group
---
		
		<ul class="list-group">
		  <li class="list-group-item">Cras justo odio</li>
		  <li class="list-group-item">Dapibus ac facilisis in</li>
		  <li class="list-group-item">Morbi leo risus</li>
		  <li class="list-group-item">Porta ac consectetur ac</li>
		  <li class="list-group-item">Vestibulum at eros</li>
		</ul>
		
##badges
		
		<ul class="list-group">
		  <li class="list-group-item">
		    <span class="badge">14</span>
		    Cras justo odio
		  </li>
		</ul>
		
#Panel
.panel .panel-default .panel-heading .panel-title .panel-body







