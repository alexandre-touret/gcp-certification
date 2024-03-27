# Create a project jumphost instance
gcloud compute instances create nucleus-jumphost-498 \
          --zone us-west1-a  \
          --machine-type e2-micro  \
          --image-family debian-11  \
          --image-project debian-cloud \
          --scopes cloud-platform \
          --no-address


# Task2
```jshelllanguage
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF


gcloud compute instance-templates create web-server-template \
          --metadata-from-file startup-script=startup.sh \
          --machine-type e2-medium \
          --tags=allow-health-check \
          --region us-west1


gcloud compute instance-groups managed create web-server-group \
          --base-instance-name web-server \
          --size 2 \
          --template web-server-template \
          --region us-west1
          --zone us-west1-a

gcloud compute instance-groups managed \
          set-named-ports web-server-group \
          --named-ports http:80 \
          --region us-west1


gcloud compute firewall-rules create allow-tcp-rule-561 \
  --network=default \
  --action=allow \
  --target-tags=allow-health-check \
  --rules=tcp:80

gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 \
  --global

gcloud compute health-checks create http http-basic-check \
  --port 80

gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global

gcloud compute backend-services add-backend web-backend-service \
  --instance-group=web-server-group \
  --instance-group-region=us-west1 \
  --global

gcloud compute url-maps create web-map-http \
    --default-service web-backend-service

gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http

gcloud compute forwarding-rules create http-content-rule \
   --address=lb-ipv4-1\
   --global \
   --target-http-proxy=http-lb-proxy \
   --ports=80

```