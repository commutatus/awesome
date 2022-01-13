---
title: Slack Integration
nav_order: 1
parent: Efficient
grand_parent: Engineering
---



# Creating a Slackbot for posting messages in channels


## _Register your Slack app_

1. Before registering your slack app create a workspace in Slack which we&#39;ll
   Use to test the app when we develop it
2. Go to [https://api.slack.com/apps](https://api.slack.com/apps) and sign in.
3. On this page click on **Create New App** CTA and select **From Scratch** option. After that you&#39;ll see this form. Input the Name and select the test work space from the dropdown and click **Create App** CTA.

   [![aws lambda console](/assets/images/slack-1.png)](/assets/images/slack-1.png)

4. You&#39;ll land on the Admin Panel of the newly created slack app. On this page you&#39;ll be able to see the App Credentials and other options such as Display Information.
5. Scroll down to **Install your app** section. Here the **Install to Workspace** CTA will be disabled with the note _Please add at least one feature or permission scope to install your app._ We need to give the app some permissions before we can install the app to our workspace.
   
   [![aws lambda console](/assets/images/slack-2.png)](/assets/images/slack-2.png)

6. Click on the **permission scope** hyperlink. And scroll down to **Scopes** section. Here under **Bot Token Scopes** click on **Add an OAuth Scope.** This drop down menu contains all the actions that the app can perform at a bot level. From the dropdown menu select **chat:write** option and that permission will be added to the app.

   [![aws lambda console](/assets/images/slack-3.png)](/assets/images/slack-3.png)

7. After adding the scope, scroll up and you&#39;ll see **Install to Workspace** CTA which now is enabled. Click on it to install to the workspace.
8. After the installation is complete, you can see the app installed on the workspace under the Apps section. Also on the Admin Panel you&#39;ll see **bot token** field. We&#39;ll need this token to configure the app in our code. Copy this code and put it in credentials file or environment variables.
   
   [![aws lambda console](/assets/images/slack-6.png)](/assets/images/slack-6.png)

9. We also have to add this app to the channel where we want to post the messages otherwise when we&#39;ll try to send the message, the Slack Client in the code will throw error.
10. Every slack request need to be verified, something like this
   
     [![aws lambda console](/assets/images/slack-4.png)](/assets/images/slack-4.png)

11. If an OAuth scope was added to a bot token earlier, and you no longer require it, then removing the scope from our App&#39;s OAuth client won&#39;t help, the token(Slack api) itself needs to be revoked, then only the scope will get removed from the token. Because if we submit the app for review from slack it won&#39;t get approved, because there will be mismatch in scopes

     [![aws lambda console](/assets/images/slack-5.png)](/assets/images/slack-5.png)


## _Integrate the slack app in Ruby on Rails_

1. In your GemFile add and Bundle Install.
  ``` ruby
   gem 'slack-ruby-client' 
  ```
2. Create a new concern which will contain the code for integrating the app.
3. In this file write a method where we will

4. Create a Slack Client and configure it with the bot token
5. Do an ``` auth_test ```
6. Post the message in a channel with a payload.

      ``` ruby
         def self.send_message_on_slack_channel(payload, channel)
            client = Slack::Web::Client.new(token:Rails.application.credentials[Rails.env.to_sym][:slack_token])
            client.auth_test
            client.chat_postMessage(channel:channel, text:'New message from Your app', blocks:payload, as_user:true)
         end
      ```

      When this method is executed the Slack client will post a message that is defined in the **payload** with the notification appearing as &quot; **New message from Your app&quot;** in the **channel** that will be passed.

-
## _Design the payload with Block Kit_

1. To create the message we use something called Block Kit, you can read more about Block Kit here - [https://app.slack.com/block-kit](https://app.slack.com/block-kit-builder).
2. Go to [https://app.slack.com/block-kit-builder](https://app.slack.com/block-kit-builder) and sign in if not automatically didn&#39;t. The Block Kit editor will open up where you can design your payload.
3. After designing the message copy the payload from the editor

      [![aws lambda console](/assets/images/slack-7.png)](/assets/images/slack-7.png)

-
## _Write a method to format the payload_

1. The copied message payload from the previous step needs to be formatted with replacing the static placeholder data to dynamic content.
2. One key thing to remember is to remove the &quot; **blocks&quot;** key from the payload.
3. Create a method that will format the payload and call the **send_message_on_slack_channel** which will send the message.

   Below is an example from the project **Gehna**.

   ``` ruby
   def post_order_created_message       
      payload = [
               {
                  "type": "section",
                  "text": {
                        "type": "mrkdwn",
                        "text": ":memo: New order received!  <@#{Rails.application.credentials[Rails.env.to_sym][:vineeth_slack_id]}>, <@#{Rails.application.credentials[Rails.env.to_sym][:padma_slack_id]}>"
                  }
               },
               {
                  "type": "section",
                  "fields": [
                        {
                           "type": "mrkdwn",
                           "text": "*Order ID:*\nGEH-#{self.id}"
                        },
                        {
                           "type": "mrkdwn",
                           "text": "*Date:*\n#{self.created_at.localtime.strftime("%d %B %Y, %A, %I:%M %p")}"
                        },
                        {
                           "type": "mrkdwn",
                           "text": "*Client Name:*\n#{self.client.client_name}"
                        },
                        {
                           "type": "mrkdwn",
                           "text": "*Total Value:*\n ₹#{self.sub_total.to_s}"
                        },
                        {
                           "type": "mrkdwn",
                           "text": "*Payment Type:*\n #{self.payment_modes}"
                        },
                        {
                           "type": "mrkdwn",
                           "text": "*POS:*\n #{self.point_of_sale.name}"
                        }
                  ]
               },
               {
                  "type": "section",
                  "fields": [
                        {
                           "type": "mrkdwn",
                           "text": "*Items:*\n #{self.order_items_list_with_link}"
                        }
                  ]
               },
               {
                  "type": "actions",
                  "elements": [
                        {
                           "type": "button",
                           "text": {
                              "type": "plain_text",
                              "emoji": true,
                              "text": "View Order"
                           },
                           "style": "primary",
                           "url": "#{Rails.application.credentials[Rails.env.to_sym][:admin_panel_url]}/orders/#{self.id}"
                        }
                  ]
               }
            ]
      GehnaSlackApp.send_message_on_slack_channel(payload, "#gehna-website-orders")
   end
   ```

   Note: It&#39;s advisable to call the post\_order\_created\_messageusing a background job.

4. When the above method is run, it&#39;ll pass the formatted payload and the channel name to **send\_message\_on\_slack\_channel** method, which will send the message to the slack channel

-
## _Event Subscriptions and Interactions -_
- **Events** - [Reference](https://api.slack.com/apis/connections/events-api)

   1. Events are basically any event occurring on slack app like clicking home tab, user getting added to the team, user making changes to his details etc.,
   2. For events we should provide a post request URL to slack, in which the slack will send a challenge parameter and in response we should provide the challenge value. IF the URL isn&#39;t able to respond with the challenge parameter, URL won&#39;t get approved by slack.
   3. As per any event occurrence we can handle the actions in our platform with the payload response provided from slack.
   4. There are scopes for Bot and User events, we can subscribe to respective events as per our requirement
   5. [![aws lambda console](/assets/images/slack-8.png)](/assets/images/slack-8.png)

      [![aws lambda console](/assets/images/slack-9.png)](/assets/images/slack-9.png)

- **Interactivity &amp; Shortcuts** - [Reference](https://api.slack.com/interactivity/handling)

   1. Interactions are any user interactions with the slack app like clicking buttons, opening modals or with any other interactive components.
   2. We have to define a Post request URL in slack for these kind of interactions
   3. There are multiple interactions with slack like **Shortcuts, Block actions, View submissions, View closed** etc..,
   4. On any interaction with the application we must provide an acknowledgement response (HTTP Post 200) to slack and this should be provided within 3 seconds.
   5. Some interactions can lead to multiple simultaneous interactions. For these kind of interactions we can use a response\_url which was provided on the response of every slack request. This URL will be unique for every request. We can post a response(Block kit) to this URL.
   6. If response\_url is not present, every interaction response will contain a variable called trigger\_id. We can use this trigger\_id in params while posting back our response to slack. Using this we can replace messages, modals, dropdowns etc.., These responses can be sent upto 5 times within 30 minutes of receiving the payload.
   7. While the interactions if we want to send and get data through block kit, we can use a parameter called private\_metadata in the view section of the block kit.
   8. **Shortcuts -** We can create custom shortcuts with callback\_id for the reference of what kind of shortcut that is. While any shortcut is being triggered, slack will send a request with that callback\_id with the response type as **shortcuts** and we can carry out actions from there.
   9. **Block actions -** This interaction occurs while any interactive element is being triggered Ex., Dropdowns, Buttons. In every block element we can define a custom action\_id from which we can get what kind of the action is being called. And in response we can provide a block kit response to slack which we desire.

## _Slash Commands -_

1. Slash commands are similar like shortcuts. But every command will require a unique request URL.
2. We can define the command, request url and a description in the application.
3. While any command was being triggered slack will hit the respective request URLs. From there we can carry out our actions.


## _Deploying the app_

1. After the tests are done all we have to do is create another app using the actual workspace by repeating the above steps.
2. The **bot token** that&#39;s generated has to be placed inside the credentials or environment variables so appropriate slack channels are used for different environments.
