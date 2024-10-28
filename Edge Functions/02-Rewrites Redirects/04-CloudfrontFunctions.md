## Dynamic Use cases - Redirect handled by cloudfront functions

### Dynamic Geo Location Redirects

- Create the Origin Request policy which adds the CloudFront-Viewer-Country header
    - Under "Origin request settings", expand "Headers" and select "Include the following headers"
        - under "Add header", search for "CloudFront-Viewer-Country" and select it


- Define the behavior in the cloudfront created as part of [Cloudformation](/Edge%20Functions/01-Sol-Deployment/01-Cloudfront.yaml) 
    - with "Path pattern"as /cff-geo.html"
    - Origin as S3 bucket
    - Viewer as "Redirect HTTP to HTTPS"
    - For "Cache key and origin requests", select "Cache policy and origin request policy (recommended)" and under "Origin request policy" select our policy created on Step 1


- Create the CloudFront function
    - Define the python code  under the "Build" sections as shown below and Save changes
        <details>
        <summary>Expand to check Python code</summary>

         ```py
        function handler(event) {
            //extracting location information from request and creating the redirected response
            var request = event.request;
            var headers = request.headers;
            var host = request.headers.host.value;
            var country = 'US' // Choose a country code
            var newurl = `/cff-geo-en-us.html` // Change the redirect URL to your choice

            if (headers['cloudfront-viewer-country']) {
                var countryCode = headers['cloudfront-viewer-country'].value;
                if (countryCode === country) {
                    var response = {
                        statusCode: 302,
                        statusDescription: 'Found',
                        headers:
                            { "location": { "value": newurl } }
                        }

                    return response;
                }
            }
            return request;
        }
            ```
        </details>
    
    - The function redirects US-based users to a specific page (/cff-geo-en-us.html) using a 302 temporary redirect, while other users continue with their original request.
    - Once the changes to the code have been saved, click the "Publish" tab, and then "Publish function". 
    - Under "Associated distributions", click on "Add association". In the new windows, choose the distribution created by the CloudFormation, select "Viewer request" for "Event type" and the "/cff-geo.html" for "Cache behavior". Finally click on "Add association".


- Testing the Configuration

    - Run the following command cloudshell or cmd prompt from the US region
        ```bash
        curl -v  https://<YOUR-DISTRIBUTION-DOMAIN-NAME>/cff-geo.html
        ```
        - Expected https response is " HTTP/1.1 302 Found" and cff-geo-en-us.html is served

    - Run the following command cloudshell or cmd prompt from the Non-US region
        ```bash
        curl -v  https://<YOUR-DISTRIBUTION-DOMAIN-NAME>/cff-geo.html
        ```
        - Expected https response is " HTTP/1.1 302 Found" and COntent is served from


### Cookie Based Redirect
- Create the CloudFront function which will evaluate cookie values
    - Define the python code  under the "Build" sections as shown below and Save changes
        <details>
        <summary>Expand to check Python code</summary>

         ```py
            function handler(event)  { 
                console.log('code is starting')
                var request = event.request;
                var cookie_exists = request.cookies
                //evaluates if a request has a cookie, if it does not immediately redirects the viewer to the login page
                if (cookie_exists.session == null) {
                    console.log("start if block")
                    var response = {
                        statusCode: 301,
                        statusDescription: 'Permanent Redirect',
                        headers:
                            { "location": { "value": "/login.html" } }
                        }
                    return response;
                //in case the first condition is false, it means we have a cookie, we need to know now if the cookie we need is set to the value we need.
                } else if (cookie_exists.session.value != "true") {
                    console.log("start else if block")
                    var response = {
                        statusCode: 301,
                        statusDescription: 'Permanent Redirect',
                        headers:
                            { "location": { "value": "/login.html" } }
                        }
                    return response;
                // if none of the above conditions are met it means the cookie is set, then we just allow the request to proceed as is
                } else {
                return request;
                }
            }
            ```
        </details>
    
    - The function ensures that only requests with a valid session cookie (where session.value is "true") can continue to the requested page. If the session cookie is missing or has an incorrect value, the user is redirected to a login page (/login.html). This is a common approach to enforce session-based authentication and access control in web applications.
    - Once the changes to the code have been saved, click the "Publish" tab, and then "Publish function". 
    - Under "Associated distributions", click on "Add association". In the new windows, choose the distribution created by the CloudFormation, select "Viewer request" for "Event type" and the "Default" for "Cache behavior". Finally click on "Add association".


- Define the behavior in the cloudfront created as part of [Cloudformation](/Edge%20Functions/01-Sol-Deployment/01-Cloudfront.yaml) 
    - with "Path pattern"as /login.html"
    - Origin as S3 bucket
    - Viewer as "Redirect HTTP to HTTPS"

- Testing the Configuration

    - Run the following command cloudshell or cmd prompt
        ```bash
        curl -v  https://<YOUR-DISTRIBUTION-DOMAIN-NAME>
        ```
        - Expected https response is " HTTP/1.1 301 Permanent Redirect" and /login.html is served

    - Run the following command cloudshell or cmd prompt 
        ```bash
        curl -v --cookie "session=false"  https://<YOUR-DISTRIBUTION-DOMAIN-NAME>
        ```
        - Expected https response is " HTTP/1.1 301 Permanent Redirect" and /login.html is served
    
    - Run the following command cloudshell or cmd prompt 
        ```bash
        curl -v --cookie "session=true"  https://<YOUR-DISTRIBUTION-DOMAIN-NAME>
        ```
        - Expected https response is "HTTP/1.1 200 OK" and /index.html is served

### Bot Signatures Based Rewrite
- Configure cloudFront with WAF web acl created as part of cloudformation template
    - Under the security tab , Select the WAD and associate the ACL with the cloudfront

- Configure the custom rule to the WAF ACL
    - Create a rule with Label as awswaf:managed:aws:bot-control:bot:category:social_media


- Create the CloudFront function which will redirect when social media bot is detcted
    - Define the python code  under the "Build" sections as shown below and Save changes
        <details>
        <summary>Expand to check Python code</summary>

         ```py
        function handler(event) {
            //extract bot header from Request
            var request = event.request
            var botRequest = request.headers['x-amzn-waf-bot'] ? request.headers['x-amzn-waf-bot'].value : undefined;
            //in case there is no bot header, then just move forward with the request, this would essentially mean which is a valid user
            if (botRequest == undefined) {
                console.log(request)
                return request
            //in case there is a safe bot identified by WAF, let's reply with a different resource
            } else if (botRequest == "true"){
                request.uri = "/bot-rewrite.html"
            }
            return request
        }
        ```
        </details>
    
    - The function checks if a request contains the x-amzn-waf-bot header. If it's a bot, the request is rewritten to serve /bot-rewrite.html; otherwise, the request proceeds unchanged.
    - Once the changes to the code have been saved, click the "Publish" tab, and then "Publish function". 

- Define the behavior in the cloudfront created as part of [Cloudformation](/Edge%20Functions/01-Sol-Deployment/01-Cloudfront.yaml) 
    - with "Path pattern"as /bot.html"
    - Origin as S3 bucket
    - Viewer as "Redirect HTTP to HTTPS"
    - Viewer request - choose CloudFront function created in earlier step 

- Testing the Configuration

    - Run the following command cloudshell or cmd prompt
        ```bash
        curl -v  https://<YOUR-DISTRIBUTION-DOMAIN-NAME>bot.html
        ```
        - Expected https response is "  HTTP/1.1 403 Forbidden" because WAF detects curl as a "http_library" bot and it blocks that

    - Open the URL  in [linkedin inspector](https://www.linkedin.com/post-inspector/) and it will redirect to BOT ReWrite HTML