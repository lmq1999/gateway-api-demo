Let’s now check that traffic based on the URL path is proxied and altered by the Gateway API:

curl http://$GATEWAY/prefix/one

The request is received by an echo server that copies the original request and sends the reply back in the body of the packet.

JSON:
{
 "path": "/one",
 "host": "20.115.194.177",
 "method": "GET",
 "proto": "HTTP/1.1",
 "headers": {
  "Accept": [
   "*/*"
  ],
  "User-Agent": [
   "curl/8.1.2"
  ],
  "X-Envoy-External-Address": [
   "A.B.C.D"
  ],
  "X-Envoy-Original-Path": [
   "/prefix/one"
  ],
  "X-Forwarded-For": [
   "A.B.C.D"
  ],
  "X-Forwarded-Proto": [
   "http"
  ],
  "X-Request-Id": [
   "7b09721a-2162-45e0-a0a8-c55829f1603a"
  ]
 },
 "namespace": "default",
 "ingress": "",
 "service": "",
 "pod": "infra-backend-v1-57dc9df649-lwxc6"
}                      

As you can see, the Gateway changed the original request from /prefix/one to /one.

As we use the Envoy proxy for L7 traffic processing, note that Envoy also adds the information about the original path in the packet (see "X-Envoy-Original-Path").

We can also combine this, with previous Gateway API features we explored in the previous tutorial. You might want to rewrite traffic and add some headers to it to add some metadata (so that the receiving server can interpret it accordingly).

If you look at the HTTPRoute we’ve just created, you will see that traffic to /rewrite-path-and-modify-headers will not only be partially rewritten, but that some headers can be added, removed or modified.

curl http://$GATEWAY/prefix/rewrite-path-and-modify-headers

Expect an output such as:

JSON:
{
  "path": "/prefix",
  "host": "20.115.194.177",
  "method": "GET",
  "proto": "HTTP/1.1",
  "headers": {
    "Accept": [
      "*/*"
    ],
    "User-Agent": [
      "curl/8.1.2"
    ],
    "X-Envoy-External-Address": [
      "A.B.C.D"
    ],
    "X-Envoy-Original-Path": [
      "/prefix/rewrite-path-and-modify-headers"
    ],
    "X-Forwarded-For": [
      "A.B.C.D"
    ],
    "X-Forwarded-Proto": [
      "http"
    ],
    "X-Header-Add": [
      "header-val-1"
    ],
    "X-Header-Add-Append": [
      "header-val-2"
    ],
    "X-Header-Set": [
      "set-overwrites-values"
    ],
    "X-Request-Id": [
      "de0055e7-5d91-49fd-98cc-181ed132a08d"
    ]
  },
  "namespace": "default",
  "ingress": "",
  "service": "",
  "pod": "infra-backend-v1-57dc9df649-lwxc6"
}