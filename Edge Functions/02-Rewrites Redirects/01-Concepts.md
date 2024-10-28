# Concepts

## Redirects

- This is a client side request and URL will be updated to the new URL in the web browser
- The purpose of the URL redirection is to take the browser to a whole new resource or a different website. The most common HTTP response codes are 301,302,and 308. Once the browser received response by the Server, it will fetch the content from the URL the server returned.
- It is useful for maintaining SEO value when moving content to a new URL, redirecting old URLs to new ones during a site redesign, or guiding users through a specific workflow.

[Additional Information for redirection](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections)

## Rewrites

- This is a Server-side and URL will not change in the browser
- The purpose of the URL rewrite is to serve a different content a different content from the Web site/application backend, without the viewer ever knowing that happened.It will essentially serve a different content from the original request without the server ever sending the viewer to a totally different URL/page.
- It makes the URL more user friendly and search engine friendly (Useful in Single page applications and for SEO optimizations)

