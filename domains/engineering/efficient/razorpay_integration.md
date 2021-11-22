---
title: Razorpay Integration
nav_order: 17
parent: Efficient
grand_parent: Engineering
---

1. TOC
{:toc}

# Razorpay Ruby on Rails Integration


## _Prerequisites_

1. [Create a Razorpay Account](https://dashboard.razorpay.com/#/access/signup)
2. After creating the account switch to **test mode**; and go to **settings** in the left navigation panel. Click on **API Keys** and click on **Generate Keys**	. Safely store the Id and Secret.

&nbsp; &nbsp;  &nbsp;  &nbsp;
[![Razorpay admin panel](/assets/images/razorpay_admin_panel.gif)](/assets/images/razorpay_admin_panel.gif)

## _Rails Integration_

1. Open the rails credentials and put the **key\_id** and **key\_secret** obtained in the previous step under test, development and staging environments.
2. In the Gemfile add the gem **razorpay** and do **bundle install**.
3. Create a file inside the ```config/initializers``` folder called **razorpay.rb**. Here require the razorpay library and set it up like so

	``` ruby
	require 'razorpay'

	Razorpay.setup(Rails.application.credentials.dig(Rails.env.to_sym, :razorpay_key_id),Rails.application.credentials.dig(Rails.env.to_sym,:razorpay_secret_token)
	```

4. Next, create a module called **razorpay\_payment.rb** inside the concerns folder, here generate the Razorpay Order like so

	``` ruby
	order = Razorpay::Order.create amount: order_amount, currency: cart.total_amount.currency.iso_code, receipt: cart.uuid, payment_capture: 1
	cart.update!(razorpay_order_id:order.id) 
	```


The options `payment_capture: 1` tells Razorpay that the payment that will be made against this order will be captured automatically. The `receipt` argument is an optional one.


The response that will be stored in the order variable is structured like this

``` json
{
	"id": "order_DBJOWzybf0sJbb",
	"entity": "order",
	"amount": 50000,
	"amount_paid": 0,
	"amount_due": 50000,
	"currency": "INR",
	"receipt": "rcptid_11",
	"status": "created",
	"attempts": 0,
	"notes": [],
	"created_at": 1566986570
}
```

**Make sure to store the id  that&#39;s returned by razorpay in this step in the database as this will be used to verify and fetch Cart or whatever you&#39;re making the order for.**


1. Write a mutation that the FE will hit to create a Razorpay Order.

	Example -

	``` ruby
	field:createRazorpayOrder, Types::Objects::RazorpayOrderResponseType do
		description "Create a Razorpay order for checkout"
		argument:cart_id, !types.String

		resolve -> (_, args, _) {
			order_response = []
			cart = Cart.find_by(uuid:args[:cart_id])
			raise Exceptions::InvalidCartId.new("Cart not found") if cart.nil?
			order_response = cart.find_or_create_razorpay_order()
			order_response
		}
	end
	```

2. Return back the order response to the client-side application with the needed values. Order&#39;s id and amount\_due is mandatory as that will be needed to show the Razorpay pop-up.
3. The client side will make the payment against the given order\_id and Razorpay will generate **razorpay\_payment\_id** and **razorpay\_signature** for the  **razorpay\_order\_id** created in the previous step and send it to the client side callback or handler function. You can request the payment\_response from the client side to verify the payment using the method provided by Razorpay to verify the payment like so

	``` ruby 
	Razorpay::Utility.verify_payment_signature(payment_response)
	```

4. Create a mutation that the client side will hit after the payment is made to verify and then process the cart order, subscription or whatever the payment was done for. Fetch the order from Razorpay using the order\_id that was stored in Step 4 and using the order\_id given to the server by client side, **use the stored order\_id ONLY.**

	``` ruby
	field:verifyRazorpayPayment, Types::Objects::OrderType do
		description "Verify a Razorpay payment"
		argument:cart_id, !types.String

		resolve -> (_, args, _) {
			cart = Cart.find_by(uuid:args[:cart_id])
			fetched_order = Razorpay::Order.fetch(cart.razorpay_order_id)
			order = cart.create_platform_order(fetched_order)
			order
		}
	end
	```
**Note:** Make sure to have only one method for creating an order or subscription which also will contain the verification of the payment. If ```the fetched_order``` status is **paid** then proceed. This single method could be reused when webhooks will be implemented.

	``` ruby
	def create_platform_order(fetched_order)
		return self.order if self.order.present?
		order = nil
		order_creation_status = {}

		if fetched_order.status == 'paid'
			Order.transaction do
				# Create your order
			end
		else
			raise Exceptions::PaymentFailed
		end
	end

	```



## _Webhooks_

Webhooks ([https://razorpay.com/docs/webhooks/](https://razorpay.com/docs/webhooks/)) are automated messages sent from apps when something happens. In this case suppose the communication between the bank and Razorpay or between you and Razorpay did not take place. This could be because of a slow network connection or your customer closing the window while the payment is being processed. This could lead to a payment being marked as **Failed** on the Razorpay Dashboard, but changed to **Authorized** at a later time.

You can use webhooks to get notified about payments that get authorized and analyze this data to decide whether or not to capture the payment.

1. **Set up webhooks on the Dashboard**
  1. Log into your [Razorpay Dashboard](https://dashboard.razorpay.com/#/access/signin) and navigate to **Settings** → **Webhooks**.
  2. Click **+ Add New Webhook**.
  3. In the **Webhook Setup** modal:
    1. Enter the **URL** where you want to receive the webhook payload when an event is triggered. We recommended using an HTTPS URL.
    2. **Note** : Webhooks can only be delivered to public URLs. If you attempt to save a localhost endpoint as part of a webhook set-up, you will notice an error. Please refer to the [test webhooks](https://razorpay.com/docs/webhooks/test/#on-an-application-running-on-localhost) section for alternatives to localhost.
    3. Enter a **Secret** for the webhook endpoint. The secret is used to validate that the webhook is from Razorpay. Do not expose the secret publicly. [Learn more about Webhook Secret.](https://razorpay.com/docs/webhooks/#validation)
    4. In the **Alert Email** field, enter the email address to which notifications must be sent in case of webhook failure.
    5. Select the required events from the list of **Active Events**. [Sample payloads for all events are available](https://razorpay.com/docs/webhooks/webhook-payloads/). 
    [![Webhook](/assets/images/webhook-setup.png)](/assets/images/webhook-setup.png)
  4. Click **Create Webhook**.

Once created, it appears on the list of webhooks:

[![Webhook list](/assets/images/webhook-list.png)](/assets/images/webhook-list.png)

Watch the short video below for more details. [Razorpay webhook setup](https://youtu.be/qojkh8Vbnek)


**_Add a listener for the webhook in the code_**

1. Add a post route in **routes.rb** that will listen for the webhook events.

	``` ruby
	post "/webhooks/process/:webhook_source",  controller: :webhooks, action: :process 
	```

2. Create a controller for this route which will handle the webhook event and also manage in case the particular webhook fires multiple times to avoid multiple order creation for the same webhook event. We&#39;ll need a model to store the webhook events which is discussed in later steps.

	``` ruby
	class WebhooksController < ApplicationController

		def process(webhook_source)
			case params[:webhook_source]
			when 'razorpay'
				Razorpay::Utility.verify_webhook_signature(request.raw_post, request.headers["X-Razorpay-Signature"], Rails.application.credentials.dig(Rails.env.to_sym,:razorpay_webhook_signature)
				webhook_event = WebhookEvent.find_by(webhook_event_id:  request.headers['x-razorpay-event-id'])
				process_razorpay(params, request) if webhook_event.nil?
			end
		head:ok
		end

		private
		def process_razorpay(params, request)
			WebhookEvent.create!(source: params[:webhook_source], payload: params.as_json, event_type: params[:event].gsub('.', '_'), webhook_event_id: request.headers['x-razorpay-event-id'])
		end
	end

	```

3. Generate a **WebhookEvent** model with the following table columns **source:integer**, **payload:json**, **event\_type:string**, **status:integer**, **webhook\_event\_id:string**.

4. Create an _after\_commit_ method

	``` ruby
	def perform_webhook_job
		ProcessWebhookJob.perform_later(self)
	end
	```

	The above **ProcessWebhookJob** will be discussed in later steps.

5. Define an instance method which will be called in the _ProcessWebhookJob_ to handle the webhooks. The method is generalized so it&#39;s able to handle webhooks other than Razorpay as well.

	``` ruby
	def process_webhook
		case source
			when 'razorpay'			
				send("handle_razorpay_#{event_type}")
			end
	end
	```

6. Create a new file called _process\_webhook\_job.rb._ Here write a perform method which will call the process\_webhook model instance method.

	``` ruby
	def perform(webhook_event)
		webhook_event.process_webhook
	end
	```

7. Create a new concern in your model concerns folder called _razorpay\_webhook.rb._ Here the actual handler method will be written.

8. Fetch the razorpay order id from the webhook payload

	``` ruby
	razorpay_order_id = payload['payload']['order']['entity']['id']
	```

9. Using this order id find the cart and then go ahead with the creation of the product order or subscription.

	``` ruby
	def handle_razorpay_order_paid
		razorpay_order_id = payload["payload"]["order"]["entity"]["id"]

		cart = Cart.find_by(razorpay_order_id:razorpay_order_id) 
		if cart
			fetched_order = Razorpay::Order.fetch(cart.razorpay_order_id)
			cart.create_gehna_order(fetched_order)
		end
	end
	```

## _Razorpay Dashboard_

1. Login to the Razorpay dashboard ([http://dashboard.razorpay.com/](http://dashboard.razorpay.com/)) and sign in.
2. Go to &#39;Transactions&#39; in the left navigation panel.
3. Here the &#39;Payments&#39; and &#39;Orders&#39; tab will be present and you can check the payments and orders to verify them yourself.

	[![Dashboard](/assets/images/rzp-dashboard.png)](/assets/images/rzp-dashboard.png)

4. If you want to refund a particular payment that option will also be available, click on the payment\_id in the &#39;Payments&#39; tab and click on &#39;Issue Refund&#39;

	[![Refund](/assets/images/issue-refund.png)](/assets/images/issue-refund.png)

5. To generate live keys switch to &#39;live mode&#39; and follow the instructions given in Prerequisites step 2.

### Note:

- For more information you can check out the official Razorpay documentation in this link

[https://razorpay.com/docs/payment-gateway/web-integration/standard/](https://razorpay.com/docs/payment-gateway/web-integration/standard/)