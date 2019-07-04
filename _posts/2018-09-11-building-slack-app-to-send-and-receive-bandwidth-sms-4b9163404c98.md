---
layout:	"post"
categories:	"blog"
title:	"Building Slack app to send and receive Bandwidth SMS"
date:	2018-09-11
thumbnail:	/img/1*z40MxCRdTn5yJ0eWglTXZQ.png
author:	
---

* * *

![](/img/1*z40MxCRdTn5yJ0eWglTXZQ.png)

Note* this is an Open Source project , and the codebase is available
[here](https://github.com/botsplash/slackSMS)

If you checked out our [previous blog](https://medium.com/botsplash-
engineering/bandwidth-sms-and-voice-integration-d7f053918b64) on Bandwidth,
you might know that we have integrated [Bandwidth](https://www.bandwidth.com/)
cloud communication provide in out conversation platform. APIs are available
in order to enable users to buy phone numbers and send/receive SMS messages
through our platform.

To help us with extensive testing and to do so with ease, we have also
developed Bandwidth [(and Twilio](https://www.twilio.com)) plugin for Slack
app. **Whats the benefit?  -- **think of this as "User Agent/Browser" testing
for UI features. When we buy multiple phone numbers from provider we need a
simple way to run your use cases.

 **The open source code base is located**[
**here**](https://github.com/botsplash/slackSMS) **.**

The add-on makes it extremely easy to test our application from Slack app, and
we maintain a separate channel in Slack where text messages can be sent using
the [slash command](https://api.slack.com/slash-commands). The format of the
message is as below.

    
    
    /sms [bandwidth|twilio] [fromNumber] [toNumber] [message]

![](/img/1*yD2XVsJ-JcTaqMW2T44BsQ.png)

Message sent from slack

And, any SMS received to a number configured to use Slack, is displayed as
below in the channel.

![](/img/1*4E1v75CUZxMjFGxU1aZbUQ.png)

###  **Let 's dive deep and breakdown the implementation!**

In order to bridge the gap between Slack API and Bandwidth/Twilio API, built
an application and exposed endpoints that can be used by Slack and
Bandwidth/Twilio. One endpoint will be used by Slack slash commands and
another by Bandwidth/Twilio.

![](/img/1*6z9aZXP248xY_UgNBxBPuQ.png)

### Prerequisites

  1. Bandwidth/Twilio account
  2. Slack account, a workspace channel ('#messaging' in our case) and a [Slack app](https://api.slack.com/apps)

The project is based on [slackSMS](https://github.com/ammaristotle/slackSMS)
by [ammaristotle](https://github.com/ammaristotle). This project allows users
to send and receive Twilio SMS message. Since we also have Bandwidth, we
decided to modify it in order to incorporate multiple platform as well as keep
room for more integrations.

### Getting Started

First, define the routes which will be used by Slack and Twilio/Bandwidth
respectively. You can clone the repository mentioned above or create new
[Express app](https://expressjs.com/) to start from scratch.

Reference below

    
    
    /** routes/index.js **/
    
    
    router.post('/sms', (req, res) => {});  
    router.post('/slack', (req, res) => {});

The first route "/sms" will be used by Twilio and Bandwidth while "/slack"
will be used by Slack.

* * *

###  **Sending message from Slack slash  commands**

>  **Slash Commands** let users trigger an interaction with your app directly
from the message box in Slack.

![](/img/1*2vIVWLUMBqfei-3ILZo87Q.png)

Example of Slack's Slash Commands taken from <https://api.slack.com/slash-
commands>

Once we have created Slack application from
[here](https://api.slack.com/apps), we need to setup Slash Commands for it. It
can be found under "Features" -> "Slash Commands" -> "Create New" in your app
dashboard.

![](/img/1*k37J2rYCow4vgXqD1P6cxg.png)

In "Request URL", we need to specify the url which we created in order for our
application to listen to Slack.

Once we save this, the "/sms" command can be used in our Slack workspace. The
text specified after "/sms" will be received in our NodeJS application. We
need to handle the payload and break it down into provider, from number, to
number and message.

Below is function that handles the payload.

The payload is split based on blank spaces and tokenized each word as a
parameter. You can apply your own logic and methods to tokenize the words.

Next, a function to send the message to the SMS provider. The message from
Slack has already parsed the payload to identify the **provider** that user is
trying to send the message to in the above "parseMessage" function.

You might recognize the code above if you read our previous
[blog](https://medium.com/botsplash-engineering/bandwidth-sms-and-voice-
integration-d7f053918b64). We are using [node-
bandwidth](https://github.com/Bandwidth/node-bandwidth) and [twilio-
node](https://github.com/twilio/twilio-node) here.

Let's put everything together and we have a route that can receive payload
from Slack slash commands, breakdown each word for parameters and send the SMS
message to the respective provider.

We want to let users know if the message has been sent or if any error has
occurred. We are using [slack-node](https://github.com/clonn/slack-node-sdk)
to send message to slack from our NodeJS application. You can get all the
instructions to set it up from their Github page.

Note: In order for it to work, you need to send message using the number
purchased from Bandwidth or Twilio dashboard and specify the provider
correctly in the slash command.

* * *

### Receiving message and sending it to Slack

Just like we had a URL to handle request from Slack, we also have URL
specified to handle Twilio/Bandwidth requests. Twilio and Bandwidth provide
option to add webhook URL that gets triggered whenever an SMS is received in
the number. We will be making use of that to redirect the message received to
our numbers into Slack. Our webhook URL will be the "/sms" route we defined in
routes/index.js

Steps to set up webhook URL in Twilio as taken from
<https://www.twilio.com/docs/sms/tutorials/how-to-receive-and-reply-python>

  1. Log into twilio.com and go to the [Console's Numbers page](https://www.twilio.com/console/phone-numbers/incoming)
  2. Click on the phone number you'd like to modify
  3. Find the Messaging section and the "A MESSAGE COMES IN" option
  4. Select "Webhook" and paste in the URL you want to use:

![](/img/1*sPVcvT1JU5eW8LERkxQJhA.png)

Steps to setup webhook URL in Bandwidth:

  1. Log into app.bandwidth.com and go to [Applications](https://app.bandwidth.com/applications/) page
  2. Create new application or select an existing application
  3. Select callback method as POST and enter the webhook URL in the Messaging callback URL field.
  4. In the "Associated numbers" section, add the number you want to setup to receive message via Slack.

![](/img/1*nQi1HHF1GSk5HLpGXM3yHA.png)

So, in both the cases above, the callback or webhook URL will be "/sms" route
that we defined. It receives the payload from Bandwidth or Twilio. More
details on the format of payload for Twilio can be found
[here](https://www.twilio.com/docs/chat/webhook-events) and for Bandwidth, it
can be found [here](https://dev.bandwidth.com/ap-docs/apiCallbacks/sms.html).
I won't discussing them as we only require three parameters here: from number,
to number, message.

    
    
    /** routes/index.js **/
    
    
    router.post('/sms', (req, res) => {
    
    
      //twilio req.body.From, req.body.To and req.body.Body  
      //bandwidth req.body.from, req.body.to and req.body.text
    
    
      const from = req.body.From || req.body.from;  
      const to = req.body.To || req.body.to;  
      const body = req.body.Body || req.body.text;  
       
      ...
    
    
    });

Once we have from number, to number and text, all we need to do is post the
message to slack in a nice format.

    
    
    /** routes/index.js **/
    
    
    router.post('/sms', (req, res) => {  
        
      ...
    
    
      slack.webhook({  
        channel: '#messaging',  
        icon_emoji: ':speech_balloon:',  
        username: process.env.SLACK_BOT_NAME,  
        attachments: [  
          {  
            fallback: `from ${from} to ${to}: ${body}`,  
            color: '#3D91FC',  
            author_name: `Recieved message from ${from} to ${to}`,  
            title: body  
          },  
        ],  
      }, () => { });  
        
      ...
    
    
    });

Here, the message received will be displayed in "#messaging" channel so we
need to have that channel in our Slack workspace.

This is what the webhook URL looks like after everything is put together.

 **There you have it**! This enables us to receive and send SMS using Slack.
Get a peek at the code base [here](https://github.com/botsplash/slackSMS).
Hope this article gave you an idea of the integration process. Do share your
thoughts and opinions in the comments sections. I would love to read them and
get back.

* * *

Do you want to read more of Botsplash team contributions? Check out articles
[**here**](https://medium.com/botsplash-engineering).

* * *

For more articles on Live Chat, Automated Bots, SMS Messaging and
Conversational Voice solutions, explore our
[**blog**](https://blogs.botsplash.com/).

Want to learn more about [Botsplash](https://www.botsplash.com) platform?
Write to us [here](https://botsplash.com/support/).

