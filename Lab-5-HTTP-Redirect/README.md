HTTP Redirect

One common use case for ingress gateways is to send HTTP redirects to clients in order to tell them that the resource they are trying to access in the cluster has moved to a different location.
This is a really common requirement for migration, content optimization, SSL/TLS enforcement – the list goes on.

Redirects return HTTP 3XX responses to a client, instructing it to retrieve a different resource. With the Gateway API, we can use redirect filters to substitute various URL components independently, as you will see below.

Let’s use this HTTPRoute YAML for the four examples in this section. We will review each rule in detail below.

Path Redirect

In this first example, we will do a simple redirect: we only replace a portion of the URL and redirect them there:

You should see the IP address allocated to the Gateway (such as 20.115.194.177).

The following rule will match traffic to /original-prefix and redirect the client to a different URL:

```
- matches:
    - path:
        type: PathPrefix
        value: /original-prefix
    filters:
    - type: RequestRedirect
      requestRedirect:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /replacement-prefix
```

Let’s try, using curl.

curl -l -v http://$GATEWAY/original-prefix

Notice we use -l in the curl request to follow the redirects (by default, curl will not follow redirects) and that we use the verbose option of curl to see the response headers. The prefix was replaced from /original-prefix to /replacement-prefix. Note we support different models to replace the URL – we can replace just a portion of the path or the entire one. 

Host & Path redirects

You can also redirect the client to a different host. 

Let’s try, with this specification, to redirect the client to example.org:

```
  - matches:
    - path:
        type: PathPrefix
        value: /path-and-host
    filters:
    - type: RequestRedirect
      requestRedirect:
        hostname: example.org
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /replacement-prefix
```

Let’s make HTTP requests to that external address and path:

curl -l -v http://$GATEWAY/path-and-host

Redirect to new prefix and custom status code

Next, you can also modify the status code. By default, as you saw above, the redirect status code is 302. It means that the resources have been moved temporarily.

To indicate that the resources the client is trying to access have moved permanently, you can use the status code 301. You can also combine it with the prefix replacement.

Let’s use this example:

  - matches:
    - path:
        type: PathPrefix
        value: /path-and-status
    filters:
    - type: RequestRedirect
      requestRedirect:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /replacement-prefix
        statusCode: 301

Try to access this URL:

curl -l -v http://$GATEWAY/path-and-status

Redirect to new prefix and custom status code

Finally, we can also use the Gateway API to impose tighter security controls. You can redirect the client to use HTTPS instead of HTTP; changing the scheme used by the client.

Look at the last line in this specification:

  - matches:
    - path:
        type: PathPrefix
        value: /scheme-and-host
    filters:
    - type: RequestRedirect
      requestRedirect:
        hostname: example.org
        scheme: "https"

Let’s try it.

curl -l -v http://$GATEWAY/scheme-and-host