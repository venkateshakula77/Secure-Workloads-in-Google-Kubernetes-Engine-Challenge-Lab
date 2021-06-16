# Secure-Workloads-in-Google-Kubernetes-Engine-Challenge-Lab

---Secure Workloads in Google Kubernetes Engine:Challenge Lab---

Open cloud Shell and run the below commands

gsutil cp gs://spls/gsp335/gsp335.zip .
unzip gsp335.zip

gcloud container clusters create kraken-cluster \
>  --zone us-central1-c \                                                                                                                                                          >  --machine-type n1-standard-4 \
>  --num-nodes 2 \
>  --enable-network-policy

gcloud sql instances create kraken-cloud-sql --region us-central1

gcloud iam service-accounts create kraken-wordpress-sa

gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \ --member="serviceAccount:kraken-wordpress-sa@DEVSHELL_PROJECT_ID.iam.gserviceaccount.com" \ --role="roles/cloudsql.client"

gcloud iam service-accounts keys create key.json --iam-account=kraken-wordpress-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com

gcloud sql databases create wordpress --instance kraken-cloud-sql --charset utf8 --collation utf8_general_ci

gcloud sql users create wordpress --host % --instance kraken-cloud-sql \ --password passwOrd

In navigation menu, click sql, then click add user account, user as username and default password

In cloud shell run the command - kubectl create secret generic cloudsql-instance-credentials --from-file key.json

kubectl create secret generic cloudsql-db-credentials \
    --from-literal username=wordpress \
    --from-literal password='passwOrd'

kubectl create -f volume.yaml

open editor in new tab, then wordpress.yaml, in line 61 delete "instance connection name" and go to navigation menu, click sql, then copy connection name and add that in deleted line 61

open cloud shell,
kubectl create -f wordpress.yaml

In navigation menu, open kubernetes engine, click workloads check whether it is created or not, if not refresh it until created

In cloud shell run the command - helm version

helm repo add stable https://charts.helm.sh/stable

helm repo update

helm install nginx-ingress stable/nginx-ingress --set rbac.create=true

kubectl get service

check once in service & ingress at kubernetes engine(nav menu), then click controller endpoints external ip

. add_ip.sh    run this in cloud shell and copy the DNS Record and save it in notepad

kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.4.0/cert-manager.yaml

kubectl create clusterrolebinding cluster-admin-binding \
    --clusterrole=cluster-admin \
    --user=$(gcloud config get-value core/account)

Refresh workloads in kubernetes engine

Go for the previously open editor and click "issuer.yaml" and paste the username1 as your email and save it(ctrl-s)

In cloud shell run the command - kubectl apply -f issuer.yaml

In open editor tab, click ingress.yaml and paste the previously saved DNS Record in the place of hostname(line 11 and 14) by removing the hyphens for example(student_01_aec2 - student01aec2) like this

In cloud shell run the command - kubectl apply -f ingress.yaml

click services & ingress,refresh it, scroll down then click on DNS Record id, if you are not getting error then refresh, click on advanced, proceed to DNS and click continue, site title- lab, username- lab, auto password, email- username1, select search engine visibility and Install, next click log in and go to lab and check your progress for task 3

In cloud shell run the command - kubect1 apply -f network-policy.yaml

open editor, click on network policy.yaml and paste the below code atlast and save it

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress

In cloud shell, again run the command "kubect1 apply -f network-policy.yaml" and check progress

Go to nav menu, click on security, binary authorization and enable it, after that click on configure policy and select disallow all images, create rule and select cluster and add specific rule, in cluster resource id - us-central1-c.kraken-cluster
and add image path- docker.io/library/wordpress:latest
Again add image path- us.gcr.io/k8s-artifacts-prod/ingress-nginx/*
Again add image path- gcr.io/cloudsql-docker/*
Again add image path- quay.io/jetstack/*

Click on save policy

Go to Kubernetes engine(nav menu), clusters,click on cluster id, scroll down edit binary authorization select enable binary authorization and save it and wait until it updated and check progress

In cloud shell, kubect1 apply -f psp-*

After that run the command - ls

open editor, click psp-restrictive.yaml(change the apiversion - policy/v1beta1) and save it

In cloud shell run the command - kubect1 apply -f psp-restrictive.yaml

kubect1 apply -f psp-role.yaml

kubect1 apply -f psp-use.yaml

Go to kubernetes engine(nav menu), click workloads, refresh it and check progress

Thank you, lab is finished

 




