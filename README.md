# GCP-challenge
gcp-ACE-challenge
## Before starting task1 need to auth gcloud and set project id which is best practice and check console just to create instance and note down zone/region
## Task 1: Task 1: create jump host instance
gcloud compute instances create <name of the host> \
--network nucleus-vpc \
--zone us-central1-b \
--machine-type f1-micro \
--image-family debian-9 \
  ##  Task2 : Create kubernetes cluster 
  gcloud container clusters create nucleus-backend \
    --release-channel regular \
    --zone <default regionus-central1-b \
    --node-locations us-central1-b
 ## Deploy and expose the cluster as per the given port in lab
kubectl create deployment hello-server \
         --image=gcr.io/google-samples/hello-app:2.0

kubectl expose deployment hello-server \
          --type=LoadBalancer \
          --port <8082>
  
  ## Task 3: Create Load balancer and note down the firewall rule name
  ## write start up script 
  cat << EOF > startup.sh
  #! /bin/bash
 apt-get update
 apt-get install -y nginx
 service nginx start
 sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
 EOF
  
  ##create instance template validate your region properly
  gcloud compute instance-templates create web-server-template \
--metadata-from-file startup-script=startup.sh \
         --network nucleus-vpc \
         --machine-type=g1-small \
         --region us-central1
  ## create instance group from the template
  gcloud compute instance-groups managed create web-server-group \
       --base-instance-name web-server \
       --size 2 \
       --template web-server-template \
       --region us-central1
  ##Create firewall rule : note the given rule name in lab
gcloud compute firewall-rules create grant-tcp-rule-827 \
       --allow tcp:80 \
       --network nucleus-vpc
  
  ## create health check
  gcloud compute http-health-checks create http-basic-check
gcloud compute instance-groups managed \
       set-named-ports web-server-group \
       --named-ports http:80 \
       --region us-central1
  ## create backend services and add to instance groups
gcloud compute backend-services create web-server-backend \
       --protocol HTTP \
       --http-health-checks http-basic-check \
       --global

gcloud compute backend-services add-backend web-server-backend \
       --instance-group web-server-group \
       --instance-group-region us-central1 \
       --global
  
  
  ##Create url maps:

gcloud compute url-maps create web-server-map \
       --default-service web-server-backend

gcloud compute target-http-proxies create http-lb-proxy \
       --url-map web-server-map
  
  ##  Create tcp url
gcloud compute forwarding-rules create grant-tcp-rule-827 \
     --global \
     --target-http-proxy http-lb-proxy \
     --ports 80
  
                    
