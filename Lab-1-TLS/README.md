root@server:~# kubectl get gateway tls-gateway
NAME          CLASS    ADDRESS          READY   AGE
tls-gateway   cilium   172.18.255.202   True    33s
root@server:~# 
root@server:~# GATEWAY_IP=$(kubectl get gateway tls-gateway -o jsonpath='{.status.addresses[0].value}')
echo $GATEWAY_IP
172.18.255.202