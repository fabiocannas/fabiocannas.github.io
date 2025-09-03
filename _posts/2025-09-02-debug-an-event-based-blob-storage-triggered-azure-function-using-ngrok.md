---
layout: single
classes: wide
title: "Debug An Event-Based Blob Storage Triggered Azure Function Using ngrok"
date: 2025-09-02
permalink: "/2025/09/02/debug-an-event-based-blob-storage-triggered-azure-function-event-grid-blob-triggered-using-ngrok/"
category: [Azure Development]
tags: [Azure Functions, Azure Event Grid, Azure Storage Account, Development]
toc: true
toc_label: "Debug An Event-Based Blob Storage Triggered Azure Function Using ngrok"
toc_icon: "book"
toc_sticky: true
---

Debugging an event-based Blob Storage triggered Azure Function may seem complicated at first glance.
As you continue reading this post, you will see that it is actually very simple.

## Prerequisites
- Visual Studio Code with Azure Resources extension;
- <a href="https://learn.microsoft.com/en-us/azure/azure-functions/functions-event-grid-blob-trigger?pivots=programming-language-csharp" target="_blank" rel="noopener noreferrer">A Blob triggered function</a>;
- An Azure Storage Account with a Blob container;
- An Event Grid system topic of type "Microsoft.Storage.StorageAccounts" with your storage account as source;
- A <a href="https://ngrok.com/docs/getting-started/" target="_blank" rel="noopener noreferrer">ngrok account</a> and a ngrok auth token;

## Why Use ngrok?
ngrok is needed to expose your locally running Azure Function to the public, so it can be triggered by Azure Event Grid webhooks event handlers.

**IMPORTANT:** there is already a very detailed tutorial on <a href="https://learn.microsoft.com/en-us/azure/azure-functions/functions-event-grid-blob-trigger?pivots=programming-language-csharp" target="_blank" rel="noopener noreferrer">Microsoft Learn</a> about this topic.
 As you can see in the official tutorial, you should use Azurite to emulate Azure Storage services when runnig locally.
 In my case, I had to attach to a real Azure Storage Account to debug an Azure Function that is triggered by blobs created by a third party service.
I wanted to make this point because exposing development resources publicly is considered an unsafe practice.
{: .notice--warning}

## Connect Your ngrok Account

```bash
ngrok config add-authtoken <your_auth_token>
```

## Start ngrok

```bash
ngrok http 7071
```

{% raw %}<img src="/assets/images/2025-09-02-Debug_An_Event_Based_Blob_Storage_Triggered_Azure_Function_Using_ngrok/ngrok-forwarding-endpoint.jpg" alt="ngrok-forwarding-endpoint">{% endraw %}

## Create An Event Grid Event Subscription Of Type Webhook
In your Event Grid System Topic, create an event subscription with the following characteristics:
- endpoint type: webhook
- event type filter: "Blob Created"
- subscriber endpoint: use the forwarding endpoint provided by ngrok at the previous step to create an event subscription in Event Grid System Topic:
    > https://{random_identifier}.ngrok-free.app/runtime/webhooks/EventGrid?functionName={function_name}

> Note: for this guide, I created an event subscription with basic settings.
> Depending on the context, further configuration may be required (see "Filters", "Additional Features", "Delivery Properties" blades on event subscription creation wizard in Azure Portal).

{% raw %}<img src="/assets/images/2025-09-02-Debug_An_Event_Based_Blob_Storage_Triggered_Azure_Function_Using_ngrok/eventgrid-event-subscription.jpg" alt="eventgrid-event-subscription">{% endraw %}

> IMPORTANT: You have to start your Azure Function locally to create the event subscription.

If the event subscription creation has been completed successfully, you will see a 200 status code in ngrok output:

{% raw %}<img src="/assets/images/2025-09-02-Debug_An_Event_Based_Blob_Storage_Triggered_Azure_Function_Using_ngrok/event-subscription-created.jpg" alt="event-subscription-created">{% endraw %}

## Start Debugging
Now that you have the event subscription in place, you can finally debug your application.

Use the Azure Resources extension in Visual Studio Code to upload files to the blob container and debug the blob triggered function.

Thank you for reading and happy debugging!