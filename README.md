# Salesforce Durable  Streaming Demo
This repository contains all the code you need to set up a Durable PushTopic Streaming and a Durable Generic Streaming client inside of Visualforce pages in your Salesforce org.  Durable Streaming adds the capability to replay any of the messages sent to the PushTopic for the last 24 hours.

This is handled through the use of an Replay ID.  Every event sent through the Streaming API will have an associated ID.  With Durable Streaming, you will have the ability to provide a Replay ID (ostensibly the last ID that your client has received) and the system will send every event that has happened in the last 24 hours after that specific ID.

If you do not provide an ID, or the ID you provide is -1, your client will be subscribed to the tip of the queue (you will receive all new messages).  If you provide an ID of -2, we resend ALL events that happened in the last 24 hours. 

## Initial Setup Instructions
1. Download this repo as zip file
    * This contains all of the needed code to run the client inside of Salesforce
2. Deploy the code to a Summer '16 that has the API enabled org using the MdAPI through Workbench, Force.com IDE, or Ant Migration Tool
3. Assign the included DurableStreamingDemo permission set to your user

## Durable PushTopic Demo Instructions
1. Create the `/topic/TestAccountStreaming` PushTopic by navigating to `/apex/DurablePushTopicStreamingDemo` - this Visualforce page will create the topic then auto-subscribe to `/topic/TestAccountStreaming` using Durable PushTopic Streaming.
2. Create notifications for the new topic
    * Use the on-screen utility to create, update and then delete an Account record
    * Or, use the UI or API to create, update, delete or undelete an Account record  
3. Observe events being received in bottom panel
4. Update replay subscription settings by changing the Replay From attribute. This will change where in the event stream you are subscribed to
    * -2 will replay from eariest available event
    * -1 (or no ID) will replay only new events
    * Provide a specific Replay ID to replay all events after that specific ID

## Durable Generic Demo Instructions
1. Create the `/u/TestStreaming` StreamingChannel by subscribing to that channel name
    * You can create the channel by using the Workbench and going to Queries > Streaming Push Topics and selecting Generic Subscriptions.  Enter the subscription as `/u/TestStreaming` and click 'Subscribe' (this will create the channel)
2. Push an event to the new channel
    * Query the ID of the channel by using the Workbench REST Explorer and doing a GET to `/services/data/v37.0/sobjects/StreamingChannel`.  In thenRecentItems response, you should see the ID for `/u/TestStreaming`
    * Do a POST to `/services/data/v37.0/sobjects/StreamingChannel/<CHANNEL_ID>/push` with a payload of:
        * `{ "pushEvents": [{"payload": "first push"} ]}`  
3. Navigate to `/apex/DurableGenericStreamingDemo`.  This Visualforce page will auto-subscribe to `/u/TestStreaming` using Durable Generic Streaming  
    * This could fail if you haven't created the streaming channel and haven't pushed at least 1 event to that channel)
4. Generate new events using on-page controls. 
    * You can also push new events through the REST API per the standard [Generic Streaming documentation](https://developer.salesforce.com/docs/atlas.en-us.api_streaming.meta/api_streaming/resources_push.htm)
    * Observe events being received in bottom panel. 
9. Update replay subscription settings by changing the Replay From attribute. This will change where in the event stream you are subscribed to
    * -2 will replay from eariest available event
    * -1 (or no ID) will replay only new events
    * Provide a specific Event ID to replay all events after that specific ID

## Issues to Remember
1. You cannot replay using an ID outside of your event stream window (if you provide an ID from 2 weeks ago, it will be invalid)
2. The current event stream window is 24 hours
