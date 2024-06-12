The Gateway will forward the traffic to infra-backend-v1 as standard, but will also copy it across to infra-backend-v2 (purple arrow in the image below). The response from infra-backend-v1 will be processed normally by the Gateway but the responses from infra-backend-v2 will be ignored (red arrow).

Let’s make a request to the /mirror path.

curl http://$GATEWAY/mirror

Look at the output below. Traffic was sent to the infra-backend-v1 Service. Was it also mirrored to infra-backend-v2 ? How can we prove it?

JSON:
{
 "path": "/mirror",
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
  "X-Forwarded-For": [
   "A.B.C.D"
  ],
  "X-Forwarded-Proto": [
   "http"
  ],
  "X-Request-Id": [
   "3caacef1-bbcb-4e01-a088-d3d411bd6dc1"
  ]
 },
 "namespace": "default",
 "ingress": "",
 "service": "",
 "pod": "infra-backend-v1-57dc9df649-xgvkr"
}

The image used by the echo Pods serving this Service is minimal. Instead of trying to install tcpdump on it, let’s use the Kubernetes debug functionality instead, with an image (nicolaka/netshoot) that is designed to troubleshoot networking issues.

The command below will deploy a container in the same Pod as the infra-backend-v2 Pod and will see the traffic coming in:

BACKEND_POD_NAME=$(kubectl get pods --no-headers=true -o custom-columns=":metadata.name" -l app=infra-backend-v2 | head -n 1)

kubectl debug $BACKEND_POD_NAME -it --image=nicolaka/netshoot  -- tcpdump -i eth0

Once you run the curl command again from a different terminal, you should see traffic appearing (you may have to run it on a few occasions as traffic will be load-balanced between multiple infra-backend-v2 Pods):

11:30:15.778113 IP 10.244.2.149.59950 > infra-backend-v2-664bffc4-8hbll.3000: Flags [F.], seq 235, ack 627, win 503, options [nop,nop,TS val 4109474445 ecr 389226307], length 0
11:30:15.778233 IP infra-backend-v2-664bffc4-8hbll.3000 > 10.244.2.149.59950: Flags [F.], seq 627, ack 236, win 501, options [nop,nop,TS val 389286309 ecr 4109474445], length 0

And this shows that the Gateway mirrored traffic to the infra-backend-v2 Service.