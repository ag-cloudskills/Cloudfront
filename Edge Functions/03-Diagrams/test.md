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
