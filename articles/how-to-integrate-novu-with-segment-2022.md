---
description: >-
  This was originally published on Aviyel but its been taken down. Original
  Link: https://aviyel.com/post/3381/how-to-integrate-novu-with-segment
---

# How to integrate Novu with Segment(2022)

> This was originally published on Aviyel(June21, 2022) but its been taken down. Original Link: [https://aviyel.com/post/3381/how-to-integrate-novu-with-segment](https://web.archive.org/web/20220621005841/https://aviyel.com/post/3381/how-to-integrate-novu-with-segment). Now Novu have published their original docs and given credits as an inspiration source [https://docs.novu.co/integrations/segment](https://docs.novu.co/integrations/segment) (the footer)

In this article, we would be learning how to send Push notifications for Segment events using Novu.

### Introduction

[Novu](https://novu.co/) is a high performant, extremely simplified notification management center. It lets you receive notifications and events from various channels under one UI component. Finally, Novu offers simple components and APIs to manage all your communication channels (Email, SMS, Direct, Push) all in one place. Although in this tutorial we would only be looking at how to receive Push notifications for Segment events.

#### Goals

For every track event on Segment, we would like to receive a notification on the Novu Notification center. So this tutorial will be extremely beneficial if you are looking for a neat solution to manage Segment events in one place.

#### Prerequisites

* A little bit of HTML and JS.
* An existing Segment account If you don’t have one, head over to [segment.com](http://segment.com) and create an account there. The process is fairly straightforward!

### Overview

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Here is a simple block diagram describing our workflow.

1. The Segment engine works with Sources and Destinations. Sources are entities that provide data in the engine. Destinations are entities that receive the data flowing out of the engine. Unfortunately, Segment doesn’t have a native Novu Destination integration yet. So we would need something else to deliver data to Novu from Segment.
2. Introducing Segment [Destination Actions](https://segment.com/docs/connections/destinations/actions/). It’s a cloud function that runs once a destination event has been triggered at Segment. This is a seamless integration into the Segment system and doesn't require any server setup of our own! So using this, payloads could be sent to Novu servers. _This is such an awesome feature that lets us send data to any custom destination we wish to._
3. The cloud function gets triggered and sends a payload to the Novu servers which then forwards it to our specific client based on the AppID.
4. Finally, our UI element on the website will be updated as result.

Out of these steps, our part is just to

* Write a cloud function that delivers the payload to Novu servers.
* Embed the Novu Notification center onto our website.

### Steps

Now that we have established a broader overview of the process. Let’s get started!

#### Setting up Novu

1. Head over to [https://web.novu.co/](https://web.novu.co/) and sign up with your email. You can also use GitHub to log in.
2.  You should be seeing your Novu dashboard after providing some basic information. Below is a screenshot of what my dashboard looks like.\\

    <figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
3. In Novu, we create a notification template with a unique name to distinguish it from the others. Let’s create our first notification template. Hit the New button on the top right. Give a unique identifier to your notification. Then click on Add channel → In-app. Here you can specify a title and a redirect link upon clicking the notification. You can also choose to opt for SMS and email notifications. But as mentioned earlier in this tutorial, we would only be focussing on Push notifications.
4.  You should see the notification has been successfully created once you hit create. Make sure it’s also enabled.

    <figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>
5.  Now go grab your API key from Settings → API keys. Store this somewhere safe, we will need it later as we set up Segment next.

    <figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

#### Setting up Segment

1.  Log in to your Segment Dashboard. From the sidebar, select Destinations. Click on the Add Destination button to proceed.

    <figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>
2.  Click on Create Function then pick Destination as Function type.\
    \\

    <figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

    <figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
3.  Segment offers an in-built code editor and testing environment to build our Destination Functions. As discussed in the overview section earlier, the destination function needs to send the payload to Novu servers every time an event is triggered here at Segment.

    <figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>
4. Let us first look at how Destination Function is structured. For every Track event of Segment, the below `onTrack` function is triggered. The code inside this function gets run every time basically. As we would like to hit our Novu servers every time that happens, we would simply be making an API call to NOVU servers with the event payload.
5.  Make the following changes to the below code snippet.

    1. Notification Name: This is the unique name that was set to the intended notification template at Novu.
    2. API endpoint and KEY: Paste your API key inside the placeholder `YOUR-API-KEY` Feel free to customize the payload according to your needs. I shall be sending only a Hello world text as my goal is just to see if the notification hits Novu center. Replace your API key that you copied from Novu dashboard and paste it below.
    3. Payload: Feel free to customise it as per your needs and the `event` object received from Segment.

    ```jsx
    async function onTrack(event, settings) {
    	const payload = {
    		name: "on-boarding-notification",
    		to: {
    			"subscriberId": "yoyo",
    			"email": "hi@hi.com",
    			"firstName": "Vitalik",
    			"lastName": "Buterin"
    		},
    		payload: {
    			name: "Hello World",
    			organization: {
    					logo: '<https://happycorp.com/logo.png>',
    			}
    		}
    	}
      const endpoint = '<https://api.novu.co/v1/events/trigger>'; 
      const ApiKey = 'YOUR-API-KEY'; // paste your NOVU API key here
      let response;

      try {
        response = await fetch(endpoint, {
      		method: 'POST',
      		headers: {
      			Authorization: `ApiKey ${ApiKey}`,
      			'Content-Type': 'application/json'
      		},
      		body: JSON.stringify(payload)
        });
      } catch (error) {
        // Retry on connection error
        throw new RetryError(error.message);
      }

      if (response.status >= 500 || response.status === 429) {
        // Retry on 5xx (server errors) and 429s (rate limits)
        throw new RetryError(`Failed with ${response.status}`);
      }
    }
    ```
6.  Once the code is ready, hit the Run button to test the function. A successful request should be sent. You can also click on the record below from Request Trace to view the response sent by the Novu server. It should be a successful response(2XX)\
    \\

    <figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

    <figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>
7. If it didn’t work as expected, please trace back and troubleshoot your code setup before proceeding ahead. **Insights for troubleshooting**
8. The notification name needs to be an exact match
9. Make sure the API key is correct
10. The status code and error response must be helpful. Refer to [this doc](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) to know what each status code means.
11. To verify our communication with Novu servers, look for the payload delivered on the Activity feed of the Novu Dashboard.\\

    <figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
12. Now that we have tested and verified our Destination function. Let’s wrap up the process by providing basic information about our Destination function.

    <figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>
13. Amazing work 🚀 We’re almost there The destination function is now ready to be connected to a destination.

    <figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
14. Either you can connect this function to any of the existing Destinations or create a new one. Here is a sample configuration connection on Segment. I have my `Test source` and `Novu instance #1` destination connected to the `destination function`\\

    <figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

#### Novu Notification center

1. We have already confirmed the payload being sent to Novu and received at our Activity Feed. But wouldn’t it be nice to see the notification received on our notification center in real-time? Yeap! let’s create a simple HTML page with the Novu embed on it.
2.  Novu offers two options to render the notification center

    1. IFrame Embed
    2. React Component

    Follow this [guide](https://docs.novu.co/docs/notification-center/iframe-embed) and pick the one that best suits your needs.
3.  I would be creating a simple web page with an IFrame embed. The code snippet for the same can be found here.

    <figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>
4.  Here is a final view of our Notification center in action. As we can see the notifications are already showing up here. On clicking the notification, it should be redirecting you to the custom redirect URL you set earlier for this Notification template.\
    \\

    <figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>
5. After configuring everything at Novu. Make sure you toggle to Production to lock in your changes

### Summary

Congratulations…… 🎉

In this tutorial, we have learned, configured, and set up both Segment and Novu so we receive real-time updates for Segment events.

#### Final remarks

This was a basic demonstration of how to leverage Segment’s destination function and Novu APIs to receive notifications. In a real-world scenario, one is expected to have multiple notification templates corresponding to the events delivered by Segment. I haven’t created anything of that sort for the sake of simplicity. But you can get creative and satisfy all your needs as you now wield the power of building Novu integrations.
