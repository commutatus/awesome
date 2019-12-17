---
title: Turn your apps to maintenance mode using environment variables
nav_order: 2
parent: Agile
grand_parent: Engineering
---

# Turn your apps to maintenance mode using environment variables

If the site needs an migration for ruby versions or any other major update, the site shut go in a maintenance mode by informing the user that 
	`Our site is under maintenance, please come back after sometime`
For this we need to hardcode our site routes every time if we go for maintenance

Making this kind of things easier we can follow the following steps to reduce the difficulty.

##	For rails apps


1. Set Environment variable as `MAINTENANCE -> true` in cloud66.
2. Set the condition in the routes of your application for redirecting to the `/maintenance` url.
		
		Note: Always place your maintenance page routes at the top of all other routes

	Example: 
		`if ENV["MAINTENANCE"] == "true"
			get '/', to: redirect('/maintenance_page'), via: :get
			get '/maintenance_page', to: 'application#maintenance_page'
			get '*path', to: redirect('/maintenance_page'), via: :get
		end`

	Create a action inside application_controller.rb called 
		`def maintenance_page
		end`

	Then create a template under views -> application -> maintenance_page.html.slim

	For API 
		The routes should be
			`if ENV["MAINTENANCE"] == "true"
			get '/', to: 'application#maintenance_page', via: :get
			get '*path', to: 'application#maintenance_page', via: :get
			post '/', to: 'application#maintenance_page', via: :post
			post '*path', to: 'application#maintenance_page', via: :post
		end`

	         `def maintenance_page
		       render json: { error: "Under Maintenance Site will be back in few hours"}, status: 503
	         end`
	         
3. Create a default maintenance template in the project
	
	Create a template called `maintenance_page.html.slim` under views -> application -> maintenance_page.html.slim

	Example Template

		<div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%,-50%); text-align: center;">
			<div>
				<img src="/w3images/forestbridge.jpg" />
			</div>
			<p style="text-transform: none; color: #31383D; font-weight: bold; font-size: 36px; padding: 20px 20px 0px 20px; line-height: 49px;">
				<br />
				Sorry, our site is under maintenance.
			</p>
			<p style="color: #31383d; font-family: &#39;Source Sans Pro&#39;; font-size: 18px; padding: 20px; line-height: 28px;">
				Thank you for being patient. You can get back to our site in a few hours.
				<br />
				Contact us on
				<a href="mailto:info@address.in" style="color:blue"> info@address.in </a> for any queries.
			</p>
		</div>
