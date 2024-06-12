root@server:~# GATEWAY=$(kubectl get gateway my-gateway -o jsonpath='{.status.addresses[0].value}')
echo $GATEWAY
172.18.255.201

HTTP Path matching

Let’s now check that traffic based on the URL path is proxied by the Gateway API.

Because of the path starts with /details, this traffic will match the first rule and will be proxied to the details Service over port 9080. The curl request is successful:

root@server:~# curl --fail -s http://$GATEWAY/details/1 | jq
{
  "id": 1,
  "author": "William Shakespeare",
  "year": 1595,
  "type": "paperback",
  "pages": 200,
  "publisher": "PublisherA",
  "language": "English",
  "ISBN-10": "1234567890",
  "ISBN-13": "123-1234567890"
}

HTTP Header Matching

This time, we will route traffic based on HTTP parameters like header values, method and query parameters.

rules:
  - matches:
   - headers:
      - type: Exact
        name: magic
        value: foo
      queryParams:
      - type: Exact
        name: great
        value: example
      path:
        type: PathPrefix
        value: /
      method: GET
    backendRefs:
    - name: productpage
      port: 9080

With curl, we can specify the headers values and query parameters to match this particular rule above:

root@server:~# curl -v -H 'magic: foo' http://"$GATEWAY"\?great\=example
*   Trying 172.18.255.201:80...
* Connected to 172.18.255.201 (172.18.255.201) port 80 (#0)
> GET /?great=example HTTP/1.1
> Host: 172.18.255.201
> User-Agent: curl/7.81.0
> Accept: */*
> magic: foo
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
[Output truncated for brevity]

TLS Termination

