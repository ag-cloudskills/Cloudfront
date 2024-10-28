# Configurations
##  Cacheable use cases


### URI Based Configuration- Redirects handled by lambda@edge functions

- Define the behavior in the cloudfront created as part of [Cloudformation](/Edge%20Functions/01-Sol-Deployment/01-Cloudfront.yaml) 
    - with "Path pattern"as /uri-main.html") 
    - Origin as S3 bucket
    - Viewer as "Redirect HTTP to HTTPS"
    - Rest of the configuration as default

- Create the lambda@Edge function
    - Choose the role created as a part of [Cloudformation](/Edge%20Functions/01-Sol-Deployment/01-Cloudfront.yaml)
    - Define the python code as shown below for python and Publish the lambda function

        <details>
        <summary>Expand to check Python code</summary>

            ```py
            import json
            def lambda_handler(event, context):
                # Let's extract the URI from the request
                get_uri = event['Records'][0]['cf']['request']['uri']
                print(get_uri)
                
                # Let's see what is the uri on the request and redirect it to a different one
                if (get_uri == '/uri-main.html'):
                    response = {
                        'status': '301',
                        'statusDescription': 'Permanent Redirect',
                        'headers': {
                            'location': [{
                                'key': 'Location',
                                'value': '/uri-redirect.html'
                            }]
                        }
                    }
                    # The response above has been created and a response will be sent to the viewer to redirect it to the uri we want them to be served.
                    return response
                else:
                    # if there is a different uri it will move the request as it was made originally
                    request = event['Records'][0]['cf']['request']
                    return request
            ```
        </details>
    
    - This Lambda function intercepts CloudFront requests. If the URI is '/uri-main.html', it redirects to '/uri-redirect.html' with a 301 status. Otherwise, it forwards the original request. 

- Add CloudFront trigger for the lambda function
    - Add Trigger on the trigger configuration page
    - Select source as Cloudfront as trigger and Deploy to Lambda@Edge
     - Select the distribution -> Created as a part of Cloudformation 
     - Cache behavior-> /ur-main.html
     - CloudFront event -> Origin Request
     - Deploy

- Testing the Configuration

    - Run the following command cloudshell or cmd prompt
        ```bash
        curl -v -o /dev/null https://<DISTRIBUTION-DOMAIN-NAME>/uri-main.html
        ```
    - Expected https response is " HTTP/1.1 301 Moved Permanently"
     


### geoLocation Redirects


- Create a cache policy which forwards the country header
    - Under "Cache key settings", expand "Headers" and select "Include the following headers"
        - under "Add header", search for "CloudFront-Viewer-Country and select it


- Create the lambda@Edge function
    - Choose the role created as a part of [Cloudformation](/Edge%20Functions/01-Sol-Deployment/01-Cloudfront.yaml)
    - Define the python code as shown below for python and Publish the lambda function

        <details>
        <summary>Expand to check Python code</summary>

            ```py
            import json
            def lambda_handler(event, context):
                #Let's first get the Country Code from the Request coming in
                get_country_viewer_header = event['Records'][0]['cf']['request']['headers']['cloudfront-viewer-country']
                define_country = get_country_viewer_header[0]['value']
                #Now let's define which country that is and create our redirected response
                if (define_country == 'US'):
                    response = {
                        'status': '301',
                        'statusDescription': 'Permanent Redirect',
                        'headers': {
                            'location': [{
                                'key': 'Location',
                                'value': '/en-us.html'
                            }]
                        }
                    }
                    #The response above has been created and a response will be sent to the viewer to redirect it to the /en-us.html
                    return response
                else:
                    #if the country has not been identified then move on with the request
                    request = event['Records'][0]['cf']['request']
                    return request
            ```
        </details>
    
    - AWS Lambda function lambda_handler checks the viewer's country from the CloudFront request and redirects US users to /en-us.html with a 301 status. For other countries, it allows the original request to proceed unchanged.

- Define the behavior in the cloudfront created as part of [Cloudformation](/Edge%20Functions/01-Sol-Deployment/01-Cloudfront.yaml) 
    - with "Path pattern"as /geo.html") 
    - Origin as S3 bucket
    - Viewer as "Redirect HTTP to HTTPS"
    - For "Cache key and origin requests", select "Cache policy and origin request policy (recommended)" and under "Cache policy" select the cache policy created in the earlier step
    - Rest of the configuration as default

- Associate CloudFront trigger for the lambda function
    - Add Trigger on the trigger configuration page
    - Select source as Cloudfront as trigger and Deploy to Lambda@Edge
     - Select the distribution -> Created as a part of Cloudformation 
     - Cache behavior-> /geo.html
     - CloudFront event -> Origin Request
     - Deploy

- Testing the Configuration

    - Run the following command cloudshell or cmd prompt form the US location
        ```bash
        curl -v -o /dev/null https://<DISTRIBUTION-DOMAIN-NAME>/geo.html
        ```
        - Expected https response is " HTTP/1.1 301 Moved Permanently" and content /en-us.html is served

    - Run the following command cloudshell or cmd prompt form the Non-US location
        ```bash
        curl -v -o /dev/null https://<DISTRIBUTION-DOMAIN-NAME>/geo.html
        ```
        - Expected https response is "HTTP/1.1 200 OK" and content /geo.html is served

### Device Redirects

- Create a cache policy which forwards the Device header
    - Under "Cache key settings", expand "Headers" and select "Include the following headers"
        - under "Add header", search for "CloudFront-Is-Mobile-Viewer" and select it


- Create the lambda@Edge function
    - Choose the role created as a part of [Cloudformation](/Edge%20Functions/01-Sol-Deployment/01-Cloudfront.yaml)
    - Define the python code as shown below for python and Publish the lambda function

        <details>
        <summary>Expand to check Python code</summary>

            ```py
            import json
            def lambda_handler(event, context):
                print(event)
                #Let's first get the Device Type from the Request coming in
                get_device_viewer_header = event['Records'][0]['cf']['request']['headers']['cloudfront-is-mobile-viewer']
                define_device_type = get_device_viewer_header[0]['value']
                print(define_device_type)
                #Now let's see if this device is a mobile viewer or not and create the redirected response based on that
                if (define_device_type == 'true'):
                    response = {
                        'status': '301',
                        'statusDescription': 'Permanent Redirect',
                        'headers': {
                            'location': [{
                                'key': 'Location',
                                'value': '/mobile.html'
                            }]
                        }
                    }
                    #The response above has been created and a response will be sent to the viewer to redirect it the right device page
                    return response
                else:
                    #if the device is not mobile, then move along with the request as is.
                    request = event['Records'][0]['cf']['request']
                    return request
            ```
        </details>
    
    - AWS Lambda function lambda_handler checks the viewer's device from the CloudFront request and redirects US users to /mobile.html with a 301 status. For other devices, it allows the original request to proceed unchanged.

- Define the behavior in the cloudfront created as part of [Cloudformation](/Edge%20Functions/01-Sol-Deployment/01-Cloudfront.yaml) 
    - with "Path pattern"as /device.html"
    - Origin as S3 bucket
    - Viewer as "Redirect HTTP to HTTPS"
    - For "Cache key and origin requests", select "Cache policy and origin request policy (recommended)" and under "Cache policy" select the cache policy created in the earlier step
    - Rest of the configuration as default

- Associate CloudFront trigger for the lambda function
    - Add Trigger on the trigger configuration page
    - Select source as Cloudfront as trigger and Deploy to Lambda@Edge
     - Select the distribution -> Created as a part of Cloudformation 
     - Cache behavior-> /device.html
     - CloudFront event -> Origin Request
     - Deploy

- Testing the Configuration

    - Run the following command cloudshell or cmd prompt
        ```bash
        curl -v -o /dev/null https://<DISTRIBUTION-DOMAIN-NAME>/device.html
        ```
        - Expected https response is " HTTP/1.1 200 Oy" and content device.html is served

    - Run the following command cloudshell or cmd prompt /or run the above command from mobile device
        ```bash
        curl -v https://<DISTRIBUTION-DOMAIN-NAME>/device.html  -A "Mozilla/5.0 (iPhone; CPU iPhone OS 6_1_3 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) CriOS/28.0.1500.12 Mobile/10B329 Safari/8536.25"

        ```
        - Expected https response is "HTTP/1.1 301 Moved Permanently" and content /mobile.html is served

### URI based Rewrites

- Define the behavior in the cloudfront created as part of [Cloudformation](/Edge%20Functions/01-Sol-Deployment/01-Cloudfront.yaml) 
    - with "Path pattern"as /uri-rewrite.html"
    - Origin as S3 bucket
    - Viewer as "Redirect HTTP to HTTPS"


- Create the lambda@Edge function
    - Choose the role created as a part of [Cloudformation](/Edge%20Functions/01-Sol-Deployment/01-Cloudfront.yaml)
    - Define the python code as shown below for python and Publish the lambda function

        <details>
        <summary>Expand to check Python code</summary>

            ```py
            import json
            def lambda_handler(event, context):
                #let's first extract the URI from the request
                get_uri = event['Records'][0]['cf']['request']['uri']
                #let's check what is the URI and decide if the URI sent to the Origin should be modified
                if (get_uri == '/uri-rewrite.html'):
                    event['Records'][0]['cf']['request']['uri'] = '/rewrite.html'
                    request = event['Records'][0]['cf']['request']
                    return request
                #if the uri should not be modified then just continue with the request as is
                else:
                    request = event['Records'][0]['cf']['request']
                    return request
            ```
        </details>
    
    - AWS Lambda function lambda_handler checks the checks the URI in the CloudFront request.If the URI is /uri-rewrite.html, it rewrites it to /rewrite.html before sending the request to the origin.If the URI is different, the request proceeds unchanged.



- Associate CloudFront trigger for the lambda function
    - Add Trigger on the trigger configuration page
    - Select source as Cloudfront as trigger and Deploy to Lambda@Edge
     - Select the distribution -> Created as a part of Cloudformation 
     - Cache behavior-> *
     - CloudFront event -> Origin Request
     - Deploy

- Testing the Configuration

    - Run the following command cloudshell or cmd prompt
        ```bash
        curl -v  https://<YOUR-DISTRIBUTION-DOMAIN-NAME>/uri-rewrite.html
        ```
        - Expected https response is " HTTP/1.1 200 OK" and content is served