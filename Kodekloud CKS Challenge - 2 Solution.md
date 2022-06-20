Kodekloud CKS Challenge - 2 Solution

A number of applications have been deployed in the dev, staging and prod namespaces. There are a few security issues with these applications.

Inspect the issues in detail by clicking on the icons of the interactive architecture diagram on the right and complete the tasks to secure the applications. Once done click on the Check button to validate your work.

Task:

Run as non root(instead, use correct application user)
Avoid exposing unnecessary ports
Avoid copying the 'Dockerfile' and other unnecessary files and directories in to the image. Move the required files and directories (app.py, requirements.txt and the templates directory) to a subdirectory called 'app' under 'webapp' and update the COPY instruction in the 'Dockerfile' accordingly.
Once the security issues are fixed, rebuild this image locally with the tag 'kodekloud/webapp-color:stable'

Solution:
root@controlplane ~ ➜  cd webapp/

root@controlplane ~/webapp via 🐍 v3.6.9 ➜  mkdir app

root@controlplane ~/webapp via 🐍 v3.6.9 ➜  ls
app  app.py  Dockerfile  requirements.txt  templates

root@controlplane ~/webapp via 🐍 v3.6.9 ➜  mv app.py app/;mv templates app/;mv requirements.txt app/

root@controlplane ~/webapp ➜  ls
app  Dockerfile

root@controlplane ~/webapp ➜  cat Dockerfile 
FROM python:3.6-alpine

## Install Flask
RUN pip install flask

## Copy All files to /opt
COPY ./app/ /opt/

## Flask app to be exposed on port 8080
EXPOSE 8080

## Flask app to be run as 'worker'
RUN adduser -D worker

WORKDIR /opt

USER worker

ENTRYPOINT ["python", "app.py"]

root@controlplane ~/webapp ➜  docker build -t kodekloud/webapp-color:stable .
Sending build context to Docker daemon  8.192kB
Step 1/8 : FROM python:3.6-alpine
3.6-alpine: Pulling from library/python
..........
..........
Removing intermediate container c94d7c207d66
 ---> 1eab5845c7fb
Successfully built 1eab5845c7fb
Successfully tagged kodekloud/webapp-color:stable

root@controlplane ~/webapp ➜  

Task:
Before you proceed, make sure to fix all issues specified in the 'Dockerfile(webapp)' section of the architecture diagram. Using the 'kubesec' tool, which is already installed on the node, identify and fix the following issues:

Fix issues with the '/root/dev-webapp.yaml' file which was used to deploy the 'dev-webapp' pod in the 'dev' namespace.
Redeploy the 'dev-webapp' pod once issues are fixed with the image 'kodekloud/webapp-color:stable'
Fix issues with the '/root/staging-webapp.yaml' file which was used to deploy the 'staging-webapp' pod in the 'staging' namespace.
Redeploy the 'staging-webapp' pod once issues are fixed with the image 'kodekloud/webapp-color:stable'

Solution:
root@controlplane ~/webapp ➜  kubesec scan /root/dev-webapp.yaml 
[
  {
    "object": "Pod/dev-webapp.dev",
    "valid": true,
    "fileName": "/root/dev-webapp.yaml",
    "message": "Failed with a score of -34 points",
    "score": -34,
    "scoring": {
      "critical": [
        {
          "id": "CapSysAdmin",
          "selector": "containers[] .securityContext .capabilities .add == SYS_ADMIN",
          "reason": "CAP_SYS_ADMIN is the most privileged capability and should always be avoided",
          "points": -30
        },
        {
          "id": "AllowPrivilegeEscalation",
          "selector": "containers[] .securityContext .allowPrivilegeEscalation == true",
          "reason": "",
          "points": -7
        }
      ],

Edit the file and update image with stable, remove SYS_ADMIN and change allowPrivilegeEscalation to false.

root@controlplane ~/webapp ➜  cat /root/dev-webapp.yaml 

----------SNIP---------------
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: darkblue
    image: kodekloud/webapp-color:stable
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    securityContext:
      runAsUser: 0
      allowPrivilegeEscalation: false
      capabilities:
        add:
        - NET_ADMIN
----------SNIP---------------

root@controlplane ~/webapp ➜  kubectl -n dev delete po dev-webapp --force 
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "dev-webapp" force deleted

root@controlplane ~/webapp ➜  kubectl create -f /root/dev-webapp.yaml 
pod/dev-webapp created

root@controlplane ~/webapp ➜  kubesec scan /root/staging-webapp.yaml 
[
  {
    "object": "Pod/staging-webapp.staging",
    "valid": true,
    "fileName": "/root/staging-webapp.yaml",
    "message": "Failed with a score of -34 points",
    "score": -34,
    "scoring": {
      "critical": [
        {
          "id": "CapSysAdmin",
          "selector": "containers[] .securityContext .capabilities .add == SYS_ADMIN",
          "reason": "CAP_SYS_ADMIN is the most privileged capability and should always be avoided",
          "points": -30
        },
        {
          "id": "AllowPrivilegeEscalation",
          "selector": "containers[] .securityContext .allowPrivilegeEscalation == true",
          "reason": "",
          "points": -7
        }
      ],

Edit the file and update image with stable, remove SYS_ADMIN and change allowPrivilegeEscalation to false.


root@controlplane ~/webapp ➜  cat /root/staging-webapp.yaml 

----------SNIP---------------
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: pink
    image: kodekloud/webapp-color:stable
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    securityContext:
      allowPrivilegeEscalation: false
      runAsUser: 0
      capabilities:
        add:
        - NET_ADMIN
----------SNIP---------------

root@controlplane ~/webapp ➜  kubectl -n staging delete po staging-webapp --force 
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "staging-webapp" force deleted

root@controlplane ~/webapp ➜  kubectl create -f /root/staging-webapp.yaml 
pod/staging-webapp created

Task:

Ensure that the pod 'dev-webapp' is immutable:

This pod can be accessed using the 'kubectl exec' command. We want to make sure that this does not happen. Use a startupProbe to remove all shells before the container startup. Use 'initialDelaySeconds' and 'periodSeconds' of '5'. Hint: For this to work you would have to run the container as root!
Image used: 'kodekloud/webapp-color:stable'
Redeploy the pod as per the above recommendations and make sure that the application is up.

Solution:
Edit the file and update spec.
----------SNIP---------------
startupProbe:
      exec:
        command:
        - /bin/rm
        - /bin/sh
        - /bin/ash
      initialDelaySeconds: 5
      periodSeconds: 5
----------SNIP---------------

root@controlplane ~/webapp ➜  kubectl -n dev delete pod dev-webapp --force 
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "dev-webapp" force deleted

root@controlplane ~/webapp ➜  kubectl create -f /root/dev-webapp.yaml 
pod/dev-webapp created

Task:
Ensure that the pod 'staging-webapp' is immutable:

This pod can be accessed using the 'kubectl exec' command. We want to make sure that this does not happen. Use a startupProbe to remove all shells (sh and ash) before the container startup. Use 'initialDelaySeconds' and 'periodSeconds' of '5'. Hint: For this to work you would have to run the container as root!
Image used: 'kodekloud/webapp-color:stable'
Redeploy the pod as per the above recommendations and make sure that the application is up.

Solution:

Edit the file and update spec
----------SNIP---------------
startupProbe:
      exec:
        command:
        - /bin/rm
        - /bin/sh
        - /bin/ash
      initialDelaySeconds: 5
      periodSeconds: 5
----------SNIP---------------

root@controlplane ~/webapp ➜  kubectl -n staging delete pod staging-webapp --force 
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "staging-webapp" force deleted

root@controlplane ~/webapp ➜  kubectl create -f /root/staging-webapp.yaml 
pod/staging-webapp created

Task:

The deployment has a secret hardcoded. Instead, create a secret called 'prod-db' for all the hardcoded values and consume the secret values as environment variables within the deployment.

Solution:

Create service account sa in prod namespace

root@controlplane ~ ➜  kubectl -n prod create sa sa
serviceaccount/sa created

Create secret prod-db in prod namespace with env values from deployment.

root@controlplane ~ ➜  kubectl -n prod create secret generic prod-db --from-literal=DB_Host=prod-db --from-literal=DB_User=root --from-literal=DB_Password=paswrd
secret/prod-db created

Edit the deployment and update spec after removing env values.

root@controlplane ~ ➜  kubectl -n prod edit deployments.apps prod-web 
----------SNIP---------------
    spec:
      serviceAccountName: sa 
      containers:
      - envFrom:                                
        - secretRef:
            name: prod-db
----------SNIP---------------                                  

Task:

Use a network policy called 'prod-netpol' that will only allow traffic only within the 'prod' namespace. All the traffic from other namespaces should be denied.

Solution:

root@controlplane ~ ➜  kubectl get ns prod --show-labels 
NAME   STATUS   AGE   LABELS
prod   Active   40m   kubernetes.io/metadata.name=prod

root@controlplane ~ ➜  kubectl label ns prod env=prod
namespace/prod labeled

root@controlplane ~ ➜  kubectl get ns prod --show-labels 
NAME   STATUS   AGE   LABELS
prod   Active   41m   env=prod,kubernetes.io/metadata.name=prod

root@controlplane ~ ➜  cat prod-netpol.yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: prod-netpol
  namespace: prod
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              env: prod 

root@controlplane ~ ➜  kubectl create -f prod-netpol.yaml 
networkpolicy.networking.k8s.io/prod-netpol created

root@controlplane ~ ➜  kubectl -n prod describe networkpolicies.networking.k8s.io prod-netpol 
Name:         prod-netpol
Namespace:    prod
Created on:   2022-06-20 14:37:42 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      NamespaceSelector: env=prod
  Not affecting egress traffic
  Policy Types: Ingress

