---
title: Rails Anycable
nav_order: 11
parent: Efficient
grand_parent: Engineering
---

## What is Action Cable

- Action Cable seamlessly integrates WebSockets with your Rails application. It allows for real-time features to be written in Ruby and the rest of your Rails application, while still being performant and scalable.

- A single Action Cable server can handle multiple connection instances. It has one connection instance per WebSocket connection. A single user may have multiple WebSockets open to your application if they use multiple browser tabs or devices. The client of a WebSocket connection is called the consumer.

- Each consumer can in turn subscribe to multiple cable channels. Each channel encapsulates a logical unit of work, similar to what a controller does in a regular MVC setup. For example, you could have a ChatChannel and an AppearancesChannel, and a consumer could be subscribed to either or to both of these channels. At the very least, a consumer should be subscribed to one channel.

- When the consumer is subscribed to a channel, they act as a subscriber. The connection between the subscriber and the channel is, surprise-surprise, called a subscription. A consumer can act as a subscriber to a given channel any number of times. For example, a consumer could subscribe to multiple chat rooms at the same time. (And remember that a physical user may have multiple consumers, one per tab/device open to your connection).

- Each channel can then again be streaming zero or more broadcastings. A broadcasting is a pubsub link where anything transmitted by the broadcaster is sent directly to the channel subscribers who are streaming that named broadcasting.

## Why AnyCable instead ActionCable

- Reduce infrastructure costs
	- Scale more efficiently with AnyCable, with its much lower RAM usage and better CPU utilization.
	- When handling 20k connections Action Cable uses 3.5GB RAM usage but AnyCable use only 798MB RAM.

- Better real-time experience
	- Real-time user experience is the battleground today. With AnyCable you win continuously as your app scales.
	- If wait times are more than a second or two, you can no longer call your application “real-time”. With AnyCable, this is no longer a problem.
	- AnyCable optimizes messaging broadcasting to provide very low latency: users no longer have to wait seconds to learn that something has happened.
	- AnyCable will give 10 times fatser response over ActionCable.

## AnyCable Installation

- Requirements
	- Ruby >= 2.5
	- Rails >= 5.0
	- Redis (when using Redis broadcast adapter)
- Installation
Add anycable-rails gem to your Gemfile:

	`gem "anycable-rails", "~> 1.0"`

	when using Redis broadcast adapter

	`gem "redis", ">= 4.0"`

	(and don't forget to run bundle install).

	`bundle exec rails g anycable:setup`

- Web Socket Server Configuration
	# For development it's likely the localhost
	# config/environments/development.rb

		config.action_cable.url = "ws://localhost:3334/cable"
		config.action_cable.mount_path = nil 
		config.action_cable.allowed_request_origins = [/http:\/\/*/, /https:\/\/*/]`

	For production it's likely to have a sub-domain and secure connection

	# config/environments/production.rb 

		config.action_cable.mount_path = nil 
		config.action_cable.url = "wss://#{ENV['API_HOST']}:3334/cable"
		config.action_cable.allowed_request_origins = [/http:\/\/*/, /https:\/\/*/]

- AnyCable Go Installation
	- The easiest way to install AnyCable-Go is to download(https://github.com/anycable/anycable-go/releases) a pre-compiled binary.

	- MacOS users could install it with Homebrew

		brew install anycable-go

	   # or use --HEAD option for edge versions

		brew install anycable-go --HEAD
- AnyCable Go Usuage
	- Change the directoy to the path where anycable go installed `$ anycable-go`

## Sample Web Socket Working Module

- Create a file called `connection.rb` inside app/channels/application_cable/ to autheticate user
- If your application uses api then the code below works

		module ApplicationCable
			class Connection < ActionCable::Connection::Base
				identified_by :current_user
				def connect 
					self.current_user = find_current_user 
					logger.add_tags "ActionCable", "User #{current_user.id}"
				end
				private
				def find_current_user 
					token = request.params['token'] 
					api_key = token.present? ? ApiKey.find_by(access_token: token) : ''
					if api_key.present? && !api_key.expired?
					    return api_key.user 
				    else
				    	reject_unauthorized_connection 
				    end
				end
			end
		end

- Else if you are not using API you can directly call current user method to authenticate.
- Create a file called test_broadcast_channel.rb inside app/channels/
	- Then place the following code inside that file

		class TestBroadcastChannel < ApplicationCable::Channel
			def subscribed
				user = current_user
				stream_from "broadcast_test_messages:#{user.id}"
			end
			def unsubscribed
				# Any cleanup needed when channel is unsubscribed
				stop_all_streams
			end
		end

- create a file called `test_streams.js` inside app/assets/javascripts/channels and place the code below

		App.notifications = App.cable.subscriptions.create("TestBroadcastChannel", {
			connected: function() {},
			disconnected: function() {},
			received: function(data) {
			    return console.log(data);
			}
		});
	- This code will help us to inspect what data was passed when we broadcast.

## Testing in the development environment

- run a anycable server using the command `bundle exec anycable`
- run a anycable go server using the command `$ anycable-go --host=localhost --port=3334`

- Before testing we need to create a file called cable.js inside app/assets/javascripts/ and place the code below

		// Action Cable provides the framework to deal with WebSockets in Rails.
		// You can generate new channels where WebSocket features live using the `rails generate channel` command.
		//
		//= require action_cable
		//= require_self
		//= require_tree ./channels
		(function() {
			this.App || (this.App = {}); 
			App.cable = ActionCable.createConsumer("ws://localhost:3334/anycable?token=<token_here>"); 
		}).call(this);`

- Now you should start rails server and you can go to any page and browser console by inspecting
- Then you should hit the code below to create web-socket connection 

		App.cable = ActionCable.createConsumer("ws://localhost:3334/anycable?token=<token_here>")`
- After the connection we should subscribe to the channel from where we need the messages to be broadcasted 

		App.cable.subscriptions.create("TestBroadcastChannel:<user_id>");

- Now you should login to the rails console to broadcast the message
- From the console we can broadcast anything by using the command called 

		ActionCable.server.broadcast "broadcast_test_messages:#{any_user_id}", { message: "This is the begining message of the broadcast."}

- Afetr hiting this command in the Rails Console you should see the data which we passed here in the browser console. If you see the message then the web-socket is working perfectly.

## Procfile to run anycable server in production environment

- In the production environment to make our anycable servers to work securely, we must create the ssl certification keys using the environment variables $SSL_CERT and $SSL_KEY 
- Then the Procfile should have the following command to start anycable and anycable go server.

		rpc: RAILS_ENV=$RAILS_ENV bundle exec anycable
		ws: sh -c 'cd ~ && exec /var/anycable-go --port 3334 -ssl_cert=/etc/ssl/localcerts/$SSL_CERT -ssl_key=/etc/ssl/localcerts/$SSL_KEY'

## Remove Firewall Restrictions for WebSocket URL

- When the firewall is blocking the websocket won't get connected
- So in this case we need to whitelist our URL from the blockage
- In cloud66 Go to Your-Application/Network Settings/
 	- Inside networking setting click the Firewall tab then click Your rules tab
 	- There change application Firewall rules to From: `anywhere`   