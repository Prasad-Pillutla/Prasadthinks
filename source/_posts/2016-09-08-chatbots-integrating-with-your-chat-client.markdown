---
layout: post
title: "Chatbots - Integrating with your chat client"
date: 2016-09-08 15:56:22 +0000
comments: true
categories: Machine Learning, Chatbots, Web Services, Microsoft Azure
---
Chatbot, software world’s new celebrity. What is it? As per Wikipedia
    
_A **Chatbot** (also known as a **Talkbot**, **Chatterbot**, **Bot**, **Chatterbox**, **Artificial Conversational Entity**) is a computer program designed to simulate an intelligent conversation with one or more human users via auditory or textual methods_.

The way I see it (in over simplified statement), it is a **Web Service** powered by Natural Language Processing and, driven by BigData and Machine/Deep Learning.

There are many frameworks available to build a Chatbot and of all, I chose [Microsoft Bot Framework](https://dev.botframework.com/) for obvious reasons. There are good number of articles on how to build Chatbot using [Microsoft Bot Framework](https://dev.botframework.com/) and deploy it on Azure, which I’ll not repeat.

Once Chatbot is built and deployed, a client application is required using which users can communicate with it. Microsoft, provides Out-of-box integration with Skype and allows Chatbot developers to integrate their Chatbots with Facebook Messenger, WeChat and [few others](https://docs.botframework.com/en-us/csharp/builder/sdkreference/gettingstarted.html#channels). 

There are situations when developers want their custom chat clients to interact with Chatbots and integration with custom chat clients is made possible by [DirectLine API](https://docs.botframework.com/en-us/restapi/directline/). Chatbot integration with custom chat clients is a challenge as there is little or no documentation and no samples available for reference. In this article I would using [fiddler](http://www.telerik.com/fiddler) to demonstrate Chatbot integration with custom chat clients. Let’s get started

**Step 1: Enable DirectLine API** 
Login into your [bot dashboard](https://dev.botframework.com/) using Microsoft Live ID. Once you register your bot you’ll supported channels.In the list of channels ***Add Direct Line*** channel.

![](/images/enableDirectLine.jpg)

When you select Add, a new browser tab will open up where you should **Check** _“Enable this bot on Direct Line”_ option. You should also generate Direct Line Secret by clicking _“Generate Direct Line secret button”_. Save generated secret for later use.

![](/images/AddDL.jpg)

**Step 2: Use Fiddler to communicate with the Chatbot hosted on Azure**

***Step 2.1: Initiate communication with Chatbot***
Any custom chat client should start its interaction with Chatbot by posting message to [](https://directline.botframework.com/api/conversations). Header of the POST message should have Authorization parameter, which take Direct Line secret generated in Step 1  
Authorization: BotConnector ***<Direct Line secret>***
Above request returns with **return code 200** and conversation id, like below

![](/images/Post 1.jpg)

**Note** conversationId. This token/conversationId is ***valid only for 30 minutes***

***Step 2.2: POST message to Chatbot***
Using the conversationId we can POST any number of messages to Chatbot. All messages to Chatbot will be posted to a new URL, which is [](https://directline.botframework.com/api/conversations/<ConversationId>/messages), in this example the POST request will be sent to 

[](https://directline.botframework.com/api/conversations/Jk21ZkYRQ6a/messages)

    Request Body will be in the format below
    {
    "text": “<Your Message>”
    }

![](/images/Post 2.jpg)

Above request doesn’t have a response body and **return Code will be 204**. 

![](/images/Post 1 response.jpg)

***Step 2.3: Retrieve Chatbot's response for the message sent***

Above request doesn’t return response from Bot. To view responses from Bot we need to perform a GET request on same URI to which we posted in _Step 2.2: POST message to Chatbot_

GET [](https://directline.botframework.com/api/conversations/Jk21ZkYRQ6a/messages)


Response from bot is present in “text” field

![](/images/Get Message.jpg)

