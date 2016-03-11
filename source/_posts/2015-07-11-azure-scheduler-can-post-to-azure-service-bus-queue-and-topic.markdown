---
layout: post
title: "Azure Scheduler can post to Azure Service Bus Queue and Topic"
date: 2015-07-11 00:23:09 +0530
comments: true
categories: Azure, Scheduler, Service Bus, Topic, Queues
---
Azure Scheduler today doesn’t allow submitting long running jobs to Service bus queues/topics but does allow submitting them to storage queues. What if, there is a requirement to submit jobs to service bus queues/topics so that features like duplicate detection can be leveraged to eliminate duplicate jobs from getting executed. One can submit messages to Service bus queue from Scheduler dashboard using REST APIs. Here is how we do it

![Job Action](/images/SB.jpg)

Select **Post** method in Scheduler Job dashboard, which displays above template. URI will represent Queue or Topic where ever we want to send message. URI template for Service bus would be *http{s}://{serviceNamespace}.servicebus.windows.net/{queuePath|topicPath}/messages*

we can add timeout for POST request by appending timeout at the end of Uri. POST Uri with timeout will be *http{s}://{serviceNamespace}.servicebus.windows.net/{queuePath|topicPath}/messages?timeout=*

**example**: *http{s}://{serviceNamespace}.servicebus.windows.net/{queuePath|topicPath}/messages?timeout=60*

**BrokerProperties** allows one to set properties like TimeToLive, MessageId, Label etc… These properties have to beaded in Json format and this is mandatory

**Authorization** property is used to define AccessToken.

**Content-Type** value can be anything but it is mandatory to have this parameter added to header.

When Scheduler executes the job, a message will be posted to Service Bus Queue or Topic. In Service Bus Explorer screen snapshot below, we can see properties that we set while creating the job
![Service Bus Explorer](/images/SBExp.jpg)


You can do this from code as well and below is the sample

		using System;
		using Microsoft.WindowsAzure.Scheduler;
		using Microsoft.Azure;
		using Microsoft.WindowsAzure.Scheduler.Models;
		using Microsoft.WindowsAzure.Management.Scheduler;

		namespace SchedulerFault
		{
		    public class Scheduler
		    {
		        private SchedulerClient SchdClient;
		        SubscriptionCloudCredentials credentials;
		        public Scheduler(SubscriptionCloudCredentials creds)
		        {
		            credentials = creds;
		            SchdClient = new SchedulerClient("<Scheduler Service>", "Job Collection Name", credentials);                
		        }

		        public void inject()
		        {
		            var Request = new JobHttpRequest(new Uri(@"<Topic/Queue URI>?timeout=60"), "POST");
		            Request.Body = "Test Newton Properties";
		            Request.Headers.Add("BrokerProperties", "{\"Label\":\"M2\", \"MessageId\":\"Id1\"}");
		            Request.Headers.Add("Content-Type","application/xml");
		            Request.Headers.Add("Authorization", "<Access Token>");

		            var StartTime = DateTime.Now;
		            var Recurrence = new JobRecurrence()
		            {
		                Frequency = JobRecurrenceFrequency.Minute,
		                Interval = 1,
		                Count = 5
		            };
		            var Action = new JobAction();
		            Action.Type = JobActionType.Http;
		            Action.Request = Request;
		            var jobParam = new JobCreateParameters();
		            jobParam.Action = Action;
		            jobParam.Recurrence = Recurrence;
		            jobParam.StartTime = StartTime;

		            try
		            {
		                var Response = SchdClient.Jobs.Create(jobParam);
		                Console.WriteLine(Response.RequestId);
		                Console.WriteLine(Response.StatusCode);
		                Console.WriteLine(Response.Job.Id);
		            }
		            catch(Exception ex)
		            {
		                Console.WriteLine(ex.Message);            
		            }            
		        }
		    }
		}

We need to install Microsoft WindowsAzure Management Scheduler Nuget package has necessary SDK for managing Scheduler and jobs in it.

Scheduler service name(first parameter for SchedulerClient constructor) can be obtained from Scheduler dashboard or it follows following format **CS-RegionName-scheduler**. If scheduler is created int Southeast Asia then Service name will be **CS-SoutheastAsia-scheduler**. If it is created in SouthCentralUS then service name will be **CS-SouthCentralUS-scheduler**.

Job Collection name(second parameter for SchedulerClient constructor) is a container in which Job will be created.

Simple aah, If anyone says to you that Azure Scheduler cannot submit messages to Service Bus topic or queue then you know what to say and where to re-direct them.