---
title: "Building Streaming APIs with Axway API Management and Streamdata"
permalink: /api-streaming/
---

With streaming APIs it's possible to push relevant updates to client devices from the server, which offers a couple of interesting benefits for large, distributed systems. In this post I'd like to show you how to use a combination of Axway's API Management solution and [Streamdata](streamdata.io) to securely virtualize and stream a JSON/REST API to end clients.

![](https://raw.githubusercontent.com/u1i/articles/master/axway_api_streaming/setup.png)


To do this exercise we need the following:

* a test installation of Axway's [API Manager & Gateway 7.5.3](https://www.axway.com/en/enterprise-solutions/api-management-plus) [1]
* a REST backend that gives us JSON data with frequent updates. Here we're using the Stock Quote container from the [Yoisho project](https://github.com/u1i/yoisho)
* a free account at [http://streamdata.io](http://streamdata.io)

Let's get started! You can choose to host the setup on a machine in the cloud, here I'm running all components (except, of course, Streamdata) on my laptop. After starting the Stock Quote service we can inspect the JSON data in the browser:  

![](https://raw.githubusercontent.com/u1i/articles/master/axway_api_streaming/01-localquote.png)

Now we're virtualizing the API in API Manager, to do this we can import the Swagger using the URL that the Yoisho service exposes:
![](https://raw.githubusercontent.com/u1i/articles/master/axway_api_streaming/03_import_swagger.png)

In this scenario, API Manager helps us to protect the backend with quota settings, preventing malicious content, and only allowing authorized access. Specifically, we're keen to ensure that only Streamdata can access the endpoint. For this, we're definining API Keys as the authentication mechanism and specify that the key has to be in the HTTP Query String:

![](https://raw.githubusercontent.com/u1i/articles/master/axway_api_streaming/05_add_API_keys.png)

We now have a secured REST endpoint, protected with an API Key. This is the URL we need to specify in the streamdata interface. Let's test it in the browser to see if it works:

![](https://raw.githubusercontent.com/u1i/articles/master/axway_api_streaming/06_virt_quote_with_key.png)

In the [Streamdata admin tool](https://portal.streamdata.io/?_ga=2.9625269.1899764506.1516547906-820332238.1511881229#/app/1d53bf8b-95ee-4a72-aed8-b4d94f55386a/getting-started) we can now create a new stream API. Here we're adding the URL of our endpoint and the API Key that the service needs to use in order to authenticate [2]:

![](https://raw.githubusercontent.com/u1i/articles/master/axway_api_streaming/08_define_streamdata_API_with_key.png)

And we're done! Now we can use cURL to trigger a client request and see the initial content and the updated stock quotes. Your command should look something like the following:

`curl -v "https://streamdata.motwin.net/https://16b5b64d.ngrok.io/quote/get_quote?X-Sd-Token=ZTI3NGJjYjctMTJjNy00ZThjLTlhYjgtZDViM2YwMTg2Mjdj"`

![](https://raw.githubusercontent.com/u1i/articles/master/axway_api_streaming/09_curl.png)

You will see the difference to a "classic" API request immediately - the connection does not terminate, which you can also notice from the `Content-Type: text/event-stream` header. Furthermore, while the initial response will return all data fields, the subsequent pushs will only return the ones that have changed ([JSON-Patch](http://jsonpatch.com/)) since the last poll from Streamdata to the backend.

I hope this helps you understand both the concept of streaming APIS, the benefits and how this can be implemented using the Axway API Management solution.

[1] Important Note: If you're keen to try this yourself, please enable the `Content-Length` setting in API Manager as described in [this KB article](https://support.axway.com/kb/163662/language/en).

[2] I'm using [ngrok](https://ngrok.com/) to expose the HTTP service running on my laptop to the internet, it's even giving me an HTTPS endpoint with a valid certificate - Streamdata requires this
