# gcp-forwarding-rules-failover
Tests for forwarding rules over GCP MIG

- Upload mig.yaml through the cloud shell (in the console)
```gcloud deployment-manager deployments create mig-deployment --config mig.yaml
gcloud compute addresses list
gcloud compute ssh client-instance --zone=us-central1-a
curl 10.128.15.192 
curl 10.128.15.193 
telnet 10.128.15.192 80
```
- SSH into one of the instances in MIG a and execute
```sudo iptables -I INPUT 1 -m state --state NEW -s 35.191.0.0/16 -p tcp --destination-port 80 -j REJECT --reject-with tcp-reset
sudo iptables -I INPUT 1 -m state --state NEW -s 130.211.0.0/22 -p tcp --destination-port 80 -j REJECT --reject-with tcp-reset
sudo iptables -I INPUT 1 -m state --state NEW -s 209.85.152.0/22 -p tcp --destination-port 80 -j REJECT --reject-with tcp-reset
sudo iptables -I INPUT 1 -m state --state NEW -s 209.85.204.0/22 -p tcp --destination-port 80 -j REJECT --reject-with tcp-reset
gcloud compute ssh client-instance --zone=us-central1-a
telnet 10.128.15.192 80
```
- While telnet waits above, from SSH on the MIG a instance execute:
```sudo iptables -D INPUT -m state --state NEW -s 35.191.0.0/16 -p tcp --destination-port 80 -j REJECT --reject-with tcp-reset
sudo iptables -D INPUT -m state --state NEW -s 130.211.0.0/22 -p tcp --destination-port 80 -j REJECT --reject-with tcp-reset
sudo iptables -D INPUT -m state --state NEW -s 209.85.152.0/22 -p tcp --destination-port 80 -j REJECT --reject-with tcp-reset
sudo iptables -D INPUT -m state --state NEW -s 209.85.204.0/22 -p tcp --destination-port 80 -j REJECT --reject-with tcp-reset
```
- Outcome: the TCP connection is closed after 50seconds, which is the http server timeout, not ~10 seconds when the failback occurred
- Note: in a similar test, the TCP connection is not closed on failover either (it still takes 50s to be closed)