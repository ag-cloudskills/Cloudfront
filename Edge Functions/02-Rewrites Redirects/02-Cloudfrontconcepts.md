
## CloudFront Edge compute Features

There are two main types of Edge features of CloudFront Distributions :

- CloudFront Functions
- Lambda@Edge

![alt text](/Edge%20Functions/03-Diagrams/Images/01-image.png)

### Lambda@Edge

- It allows to run code closer to users of application, which improves performance and reduces latency.
- There is no need to provision or manage infrastructure in multiple locations around the world. 
- It is charged on-demand basis.
- Lambda@Edge functions with origin-facing triggers are executed after CloudFront's cache is checked. If a cached response exists, it will be served, avoiding the need for function invocation. This reduces function execution, speeds up responses, and lowers costs, as the Lambda function is triggered only when there is no cached response


### CloudFront Functions

It is lightweight functions in JavaScript for high-scale, latency-sensitive CDN customizations. It can manipulate the requests and responses that flow through CloudFront, perform basic authentication and authorization, generate HTTP responses at the edge, and more.The CloudFront Functions runtime environment offers submillisecond startup times, scales immediately to handle millions of requests per second, and is highly secure


## CloudFront Edge Compute Triggers

CloudFront offers 4 different type of event types to customize the request and response being exchanged between viewer and Server

- Viewer Request – The function executes when CloudFront receives a request from a viewer and before it checks to see whether the requested object is in the edge cache

- Origin Request – The function executes only when CloudFront forwards a request to your origin. When the requested object is in the edge cache, the function doesn’t execute.

- Origin Response – The function executes after CloudFront receives a response from the origin and before it caches the object in the response.

- Viewer Response – The function executes before returning the requested object to the viewer. The function executes regardless of whether the object is already in the edge cache.

All 4 trigger options are available with Lambda@Edge while only viewer triggers are available with CloudFront Functions. This is one of the most important differences between the two features.

## CloudFront Edge Locations and Regional Edge Caches (RECs)


![alt text](/Edge%20Functions/03-Diagrams/Images/02-image.png)

### CloudFront Edge Locations

These are the points of presence where user requests will be sent to based on the lowest latency for the user sending that request.

### Regional Edge Cache

It is middle tier caching layer which sits between the Edge Location and the Origin. These Regional Edge Cache servers have been introduced to allow for more content to be cached closer to users.


## Lambda@Edge Use cases

### URI based Redirects

![alt text](/Edge%20Functions/03-Diagrams/Images/03-image.png)

This use case is commonly used for the redirect of the URL 
 
Configuration for the above use case can be found at [link](03-Lambda@edge.md#uri-based-configuration--redirects-handled-by-lambdaedge-functions)


### Geo Location Redirects

![alt text](/Edge%20Functions/03-Diagrams/Images/04-image.png)

This use case is commonly used to redirect the viewer to the proper country page of website

Configuration for the above use case can be found at [link](03-Lambda@edge.md#geolocation-redirects)


### Device Redirects

![alt text](/Edge%20Functions/03-Diagrams/Images/05-image.png)

This use case is commonly used to redirect the viewer to the proper page of the website, based on the device type the request is coming from.

Configuration for the above use case can be found at [link](03-Lambda@edge.md#device-redirects)



### URI based Rewrites

![alt text](/Edge%20Functions/03-Diagrams/Images/06-image.png)

This technique involves rewriting the URI without redirecting the client browser. While the browser continues to see the original URI, CloudFront retrieves content from a different path or origin using Lambda@Edge or CloudFront Functions. For example, a request to www.example.com/uri-rewrite.html could fetch content from www.example.com/rewrite.html without the viewer being aware of the change.

Configuration for the above use case can be found at [link](03-Lambda@edge.md#uri-based-rewrites)


## CloudFront Function Use cases

CloudFront functions gets launched before any interaction of the request with CloudFront's cache, hence making it so every single request landing on CloudFront gets that function triggers.


### Dynamic Geo Location Redirects

![alt text](/Edge%20Functions/03-Diagrams/Images/07-image.png)

Using Lambda@Edge for redirects is efficient when the response can be cached for reuse. However, for dynamic redirects based on varying request data, CloudFront Functions are more cost-effective, as frequent Lambda@Edge triggers can lead to higher costs.

Configuration for the above use case can be found at [link](04-CloudfrontFunctions.md#dynamic-geo-location-redirects)

### Cookie Based Redirect

![alt text](/Edge%20Functions/03-Diagrams/Images/08-image.png)

Cookies are valuable for developers to manage client behavior in web applications by storing specific values set by the server. They help identify valid users and their active sessions, allowing the server to tailor responses based on previous interactions. Additionally, cookies can be used to redirect users based on their last explored resources on the site.

Configuration for the above use case can be found at [link](04-CloudfrontFunctions.md#cookie-based-redirect)


### Bot Signatures Based Rewrite

![alt text](/Edge%20Functions/03-Diagrams/Images/09-image.png)


AWS WAF provides a Managed Rule that detects and blocks common bots. Some bots, like social media and search engine bots, are allowed by default. If needed, it's possible to let the bot through the firewall but serve a different page or resource instead of the one originally requested.

Configuration for the above use case can be found at [link](04-CloudfrontFunctions.md#bot-signatures-based-rewrite)