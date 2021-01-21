---
title: Turn your apps to maintenance mode using environment variables
nav_order: 3
parent: Agile
grand_parent: Engineering
---

# Turn your apps to maintenance mode using environment variables

If there is a planned maintainence, a site must go into maintenance mode and the end user should be made aware of the event.
A notice something like - `Our site is under maintenance, please come back after sometime` should be conveyed. To avoid hardcoding the site routes every time if it goes under maintenance, environment variables can be employed.

##	For rails apps

1. Set an environment variable as `MAINTENANCE = true` in cloud66.
2. Check for this variable in the routes file of your application for redirecting to a `/maintenance` page.

		Note: Always place your maintenance page routes at the top of all other routes

	Examples:
	- For a monolith application

		```
		if ENV["MAINTENANCE"] == true
			get '/', to: redirect('/maintenance_page'), via: :get
			get '/maintenance_page', to: 'application#maintenance_page'
			get '*path', to: redirect('/maintenance_page'), via: :get
		end

		```

	Create an action inside `application_controller.rb` example
		```
		def maintenance_page
		end
		```

	Then create a template under views/application/maintenance_page.html

	- For an API the routes should be

		```
		if ENV["MAINTENANCE"] == true
			get '/', to: 'application#maintenance_page', via: :get
			get '*path', to: 'application#maintenance_page', via: :get
			post '/', to: 'application#maintenance_page', via: :post
			post '*path', to: 'application#maintenance_page', via: :post
		end

		def maintenance_page
			render json: { error: "Under Maintenance Site will be back in few hours"}, status: 503
		end

		```
	 The status code 503 will allow a front-end client to understand the application is under maintainence and they can convey it to the user.

	Example template in a monolith app.

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

	*An industry standard practise is to keep the maintenance page colour scheme and design consistent with your application.
