---
layout:	"post"
categories:	"blog"
title:	"Bandwidth SMS and Voice Integration"
date:	2018-08-22
thumbnail:	/img/1*SNzHbMzC-kFiO0garrZVsw.png
author:	
---

* * *

![](/img/1*SNzHbMzC-kFiO0garrZVsw.png)

[Bandwidth](https://www.bandwidth.com/) is a cloud communication platform that
provides API based integration of voice and sms features into business
applications. Here at [Botsplash](https://botsplash.com), we integrated
Bandwidth for our conversation platform, to allow agents and visitors to
send/receive messages and make voice calls. This blog is a deep dive to share
with you how we implemented Bandwidth integration.

### Getting Started

Before we can begin, the first step is creating a [Bandwidth
account](https://catapult.inetwork.com/portal/signup). Next, create new
application by going to the Bandwidth [dashboard](https://app.bandwidth.com/)
and navigate to the Applications tab. To continue, note the API credentials
namely userId, apiToken and apiSecret.

Our technology is NodeJS backend and hence have used their NodeJS
[SDK](https://github.com/Bandwidth/node-bandwidth). The business use case for
our platform is that Botsplash clients buy phone numbers and use the same for
sending and receiving messages. The APIs we implemented are

  1. Search phone number based on area code
  2. Buy phone number
  3. Send SMS/MMS message
  4. Receive SMS/MMS message

 **Note** : Depending of your specific use case, you might require any, all or
more integrations.

### Bandwidth client and methods

    
    
    //instantiating bandwidth-nodejs client
    
    
    var Bandwidth = require('node-bandwidth');
    
    
    var client = new Bandwidth({  
        userId    : "YOUR_USER_ID",  
        apiToken  : "YOUR_API_TOKEN",  
        apiSecret : "YOUR_API_SECRET"  
    }); 

  1. **Search phone number based on area code**

The first API we need is to allow users to search for available phone numbers.

    
    
    router.get('/search-phone-number/:areaCode', (req, res) => {  
      var areaCode = req.params.areaCode;  
        
      bandwidthClient.AvailableNumber.search('local', {  
        areaCode,  
        quantity : 5  
      }).then((numbers) => {   
        //return available numbers  
        res.send(numbers);  
      });  
    });

This API will return 5 phone numbers in the given area code. We display the
returned numbers to the user in the UI which they will be able to select in
order to buy.

 **2\. Buy phone number**

This API will allow user to buy a number which the user selects from the
previous search phone number API. It will take on the application Id and the
phone number as payload body.

    
    
    POST /send-message HTTP/1.1  
    Content-Type: application/json; charset=utf-8  
      
    {  
        "phoneNumber"     : "+13233326955",  
        "applicationId"   : "{appId}"  
    }

* * *
    
    
    router.post('/buy-phone-number', (req, res) => {  
      var number = req.body.phoneNumber;  
      var applicationId = req.body.applicationId;
    
    
      bandwidthClient.PhoneNumber.create(  
        Object.assign(  
          { number },  
          applicationId ? { applicationId } : null  
        );  
      ).then((response) => {  
        //send user success response  
        res.send({  
          success: true,  
          number     
        });  
      });  
    });

 **3\. Send SMS/MMS message**

This API will allow sending SMS or MMS message from the Bandwidth number that
we purchased. For MMS, we simply need to send list of media urls in the
payload body.

    
    
    POST /send-message HTTP/1.1  
    Content-Type: application/json; charset=utf-8  
      
    {  
        "from"   : "+13233326955",  
        "to"     : "+13865245000",  
        "text"   : "Example message",  
        "media"  : ["http://example.com/1.jpg"]  
    }

* * *
    
    
    router.post('/send-message', (req, res, next) => {
    
    
      var text  = req.body.message;  
      var from  = req.body.from; //number purchased from Bandwidth  
      var to    = req.body.to;  
      var media = req.body.media; //media url for MMS
    
    
      bandwidthClient.Message.send(  
        Object.assign(  
          { text, from, to },  
          media  
        )  
      ).then((message) => {  
        console.log("Message sent with ID " + message.id);  
          
        // save message sent record to db  
        handleMessageSent(message).then((res) => {  
          res.send(res);  
        });  
      }).catch(function(err) {  
        console.log(err.message);  
        return next(err); //error handler  
      });
    
    
    });

In the above code, bandwidthClient.Message.send will send the message to the
provided phone number. Once the message is sent, we need to keep this record
in our database which is what "handleMesssageSent" function does.

 **4\. Receive SMS/MMS message**

Bandwidth provides option to add a webhook url in our Bandwidth application
dashboard that will get triggered if an SMS/MMS is received to any number
attached to the application. We will need to build this webhook url that can
listen to messages received.

Response data received from Bandwidth to our webhook url:

    
    
    POST /receive-message HTTP/1.1  
    Content-Type: application/json; charset=utf-8  
    User-Agent: BandwidthAPI/v1  
      
    {  
        "eventType"     : "sms",  
        "direction"     : "in",  
        "messageId"     : "{messageId}",  
        "messageUri"    : "https://api.catapult.inetwork.com/v1/users/{userId}/messages/{messageId}",  
        "from"          : "+13233326955",  
        "to"            : "+13865245000",  
        "text"          : "Example",  
        "applicationId" : "{appId}",  
        "time"          : "2012-11-14T16:13:06.076Z",  
        "state"         : "received"  
    }

* * *
    
    
    router.post('/receive-message', (req, res, next) => {  
      var { text, from, to } = req.body;  
      var attachments = getAttachments(req.body);  
        
      handleMessage(  
        text,  
        attachments,  
        from,  
        to  
      ).then((response) => {  
        return { success: true };  
      });
    
    
    });

The above endpoint will receive the message payload from Bandwidth. We can
have a function to parse the attachments to store the media in different
server as required which is what "getAttachments" function does. We then save
the message received to the database.

Similar to the messages, voice call engagement and forwarding could be enabled
using the bandwidth SDK. For complete set of capabilities and integration
details, refer to the [Calls API](https://dev.bandwidth.com/ap-
docs/methods/calls/calls.html).

Hope you have enjoyed this post and found it useful. It is very encouraging to
read your comments and feedback, so please share in comments sections any
thoughts you may have.

* * *

Do you want to read more of Botsplash team contributions? Check out articles
[**here**](https://medium.com/botsplash-engineering).

* * *

For more articles on Live Chat, Automated Bots, SMS Messaging and
Conversational Voice solutions, explore our
[**blog**](https://blogs.botsplash.com/).

Want to learn more about [Botsplash](https://www.botsplash.com) platform?
Write to us [here](https://botsplash.com/support/).

