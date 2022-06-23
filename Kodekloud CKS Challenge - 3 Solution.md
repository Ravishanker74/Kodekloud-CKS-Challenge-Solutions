Kodekloud CKS Challenge - 3 Solutions

This is a two node kubernetes cluster. Using the kube-bench utility, identify and fix all the issues that were reported as failed for the controlplane and the worker node components.


Inspect the issues in detail by clicking on the icons of the interactive architecture diagram on the right and complete the tasks to secure the cluster. Once done click on the Check button to validate your work.

Task:

Download 'kube-bench' from AquaSec and extract it under '/opt' filesystem. Use the appropriate steps from the kube-bench docs to complete this task.
Run 'kube-bench' with config directory set to '/opt/cfg' and '/opt/cfg/config.yaml' as the config file. Redirect the result to '/var/www/html/index.html' file.

Solution:

root@controlplane ~ ➜  cd /opt
root@controlplane /opt ➜  curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.6.2/kube-bench_0.6.2_linux_amd64.tar.gz -o kube-bench_0.6.2_linux_amd64.tar.gz
root@controlplane /opt ➜  tar -xvf kube-bench_0.6.2_linux_amd64.tar.gz

root@controlplane /opt ➜  ./kube-bench --config-dir=/opt/cfg --config=/opt/cfg/config.yaml >/var/www/html/index.html
-su: /var/www/html/index.html: No such file or directory

root@controlplane /opt ➜ mkdir -p /var/www/html
root@controlplane /opt ➜  ./kube-bench --config-dir=/opt/cfg --config=/opt/cfg/config.yaml >/var/www/html/index.html


Task:

Ensure that the --protect-kernel-defaults argument is set to true (node01)

Solution:
root@controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
controlplane   Ready    control-plane,master   16m   v1.23.0
node01         Ready    <none>                 15m   v1.23.0

root@controlplane ~ ➜  ssh node01

root@node01 ~ ➜  

https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/

protectKernelDefaults
bool	
protectKernelDefaults, if true, causes the Kubelet to error if kernel flags are not as it expects. Otherwise the Kubelet will attempt to modify kernel flags to match its expectation. Default: false

Edit and update
root@node01 ~ ➜  cat /var/lib/kubelet/config.yaml | tail -1
protectKernelDefaults: true


root@node01 ~ ✖ systemctl daemon-reload 

root@node01 ~ ➜  systemctl restart kubelet.service 

Task:
Ensure that the --protect-kernel-defaults argument is set to true (controlplane)

Solution:
Edit and update
root@controlplane ~ ➜  cat /var/lib/kubelet/config.yaml | tail -1
protectKernelDefaults: true

root@controlplane ~ ➜  systemctl daemon-reload 
root@controlplane ~ ➜  systemctl restart kubelet.service 

Task: 
Ensure that the --profiling argument is set to false
Ensure PodSecurityPolicy admission controller is enabled
Ensure that the --insecure-port argument is set to 0
Ensure that the --audit-log-path argument is set to /var/log/apiserver/audit.log
Ensure that the --audit-log-maxage argument is set to 30
Ensure that the --audit-log-maxbackup argument is set to 10
Ensure that the --audit-log-maxsize argument is set to 100

Solution:
Addlne as below in appropriate place.
root@controlplane ~ ➜  cat /etc/kubernetes/manifests/kube-apiserver.yaml | egrep "profiling|PodSecurityPolicy|insecure-port|audit-log-path|audit-log-maxage|audit-log-maxbackup|audit-log-maxsize"
    - --profiling=false
    - --enable-admission-plugins=NodeRestriction,PodSecurityPolicy
    - --audit-log-path=/var/log/apiserver/audit.log
    - --audit-log-maxage=30
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
    - --insecure-port=0

root@controlplane ~ ➜  cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -A2 -B1 /var/log/apiserver
--SNIPPED--
--
    - mountPath: /var/log/apiserver/
      name: audit-log
      readOnly: false
--
  - hostPath:
      path: /var/log/apiserver
      type: DirectoryOrCreate
    name: audit-log

--SNIPPED--

Task:

kube-scheduler in controlplane. 
Ensure that the --profiling argument is set to false

Solution:
Addlne as below in appropriate place.
root@controlplane ~ ➜  cat /etc/kubernetes/manifests/kube-scheduler.yaml | grep profiling
    - --profiling=false

Task:

kube-controller-manager in controlplane.
Ensure that the --profiling argument is set to false

Solution:
Addlne as below in appropriate place.
root@controlplane ~ ➜  cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep profiling
    - --profiling=false

Task:

Correct the etcd data directory ownership

Solution:
root@controlplane ~ ➜  cat /var/www/html/index.html | grep chown
chown root:root <path/to/cni/files>
For example, chown etcd:etcd /var/lib/etcd

root@controlplane ~ ➜  ls -ld /var/lib/etcd
drwx------ 3 etcd root 4096 Jun 20 18:13 /var/lib/etcd

root@controlplane ~ ➜  chown etcd:etcd /var/lib/etcd

root@controlplane ~ ➜  ls -ld /var/lib/etcd
drwx------ 3 etcd etcd 4096 Jun 20 18:13 /var/lib/etcd



















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

