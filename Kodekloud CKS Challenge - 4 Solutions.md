Kodekloud CKS Challenge - 4 Solutions

There are a number of Kubernetes objects created inside the omega, citadel and eden-prime namespaces. However, several suspicious/abnormal operations have been observed in these namespaces!.

For example, in the citadel namespace, the application called webapp-color is constantly changing! You can see this for yourself by clicking on the citadel-webapp link and refreshing the page every 30 seconds. Similarly there are other issues with several other objects in other namespaces.

To understand what's causing these anomalies, you would be required to configure auditing in Kubernetes and make use of the Falco tool.

Inspect the issues in detail by clicking on the icons of the interactive architecture diagram on the right and complete the tasks to secure the cluster. Once done click on the Check button to validate your work.

Task:

API server running? 

Solution:

root@controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
controlplane   Ready    control-plane,master   73m   v1.23.0

Task:

The audit policy file should be stored at '/etc/kubernetes/audit-policy.yaml'

Use a volume called 'audit' that will mount only the file '/etc/kubernetes/audit-policy.yaml' from the controlplane inside the api server pod in a read only mode.

Create a single rule in the audit policy that will record events for the 'two' objects depicting abnormal behaviour in the 'citadel' namespace. 

This rule should however be applied to all 'three' namespaces shown in the diagram at a 'metadata' level. Omit the 'RequestReceived' stage.

audit-log-path set to '/var/log/kubernetes/audit/audit.log'

Solution:

Edit the audit policy file and update as below:
root@controlplane ~ ➜  cat /etc/kubernetes/audit-policy.yaml
apiVersion: audit.k8s.io/v1 
kind: Policy
omitStages:
  - "RequestReceived"
rules:
- level: Metadata

At this point we have not configured for the 2 objects depicting abnormal behaviour as the exact objects are not known.

Edit the file /etc/kubernetes/manifests/kube-apiserver.yaml and add the following lines for audit policy & log. The path in task is same as that in kubernetes audit documentation.

https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/

Add the volumes and volumeMounts appropriately.

Afte making the changes for enabling audit and audit logs observe the logs for namespace citadel.

root@controlplane ~ ➜  cat /var/log/kubernetes/audit/audit.log  | grep citadel | grep resource | head
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"3217e7fa-92c7-4067-a340-3ae0689f35c7","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps/webapp-config-map","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Success","code":200},"requestReceivedTimestamp":"2022-06-23T13:38:21.709680Z","stageTimestamp":"2022-06-23T13:38:21.721166Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"2f4ae2e1-937c-4d0d-9efb-b55446932ddd","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dwebapp-config-map","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:38:21.724057Z","stageTimestamp":"2022-06-23T13:38:21.725806Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"07edb2b4-2d01-4b80-9e48-5636a8f0d08d","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/resourcequotas","verb":"list","user":{"username":"system:apiserver","uid":"a65e22fb-cdd7-4db3-bbd2-622b503a4636","groups":["system:masters"]},"sourceIPs":["::1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"resourcequotas","namespace":"citadel","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:38:21.796565Z","stageTimestamp":"2022-06-23T13:38:21.798615Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"f21c941a-f5d8-49aa-8ffe-dae8c66f8713","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2022-06-23T13:38:21.794635Z","stageTimestamp":"2022-06-23T13:38:21.800981Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"cc296a89-f5d3-4574-a210-2855e406c80b","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:38:21.871164Z","stageTimestamp":"2022-06-23T13:38:21.881241Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"bdf39120-cb1d-43c1-9bde-cd16d6c6c25a","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldSelector=metadata.name%3Dwebapp-color","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:38:21.882908Z","stageTimestamp":"2022-06-23T13:38:21.884471Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}

From the above logs we observe the two objects are configmaps and pod. Modifying the audit policy to include these. 
root@controlplane ~ ➜  cat /etc/kubernetes/audit-policy.yaml 
root@controlplane /etc/kubernetes/manifests ➜  cat /etc/kubernetes/audit-policy.yaml 
apiVersion: audit.k8s.io/v1 
kind: Policy
omitStages:
  - "RequestReceived"
rules:
- level: Metadata
  resources:
  - group: ""
    resources: ["pods","configmaps"]
  namespaces: ['omega','citadel','eden-prime']

After the changes are made ensure the kube-api pod is restarted.

root@controlplane ~ ➜ cd /etc/kubernetes/manifests/

root@controlplane /etc/kubernetes/manifests ➜  mv kube-apiserver.yaml ../

root@controlplane /etc/kubernetes/manifests ➜  mv ../kube-apiserver.yaml .

root@controlplane ~ ➜  truncate -s 0 /var/log/kubernetes/audit/audit.log


Task:
Install the 'falco' utility on the controlplane node and start it as a systemd service

Solution:
root@controlplane ~ ➜  apt install -y falco

root@controlplane ~ ➜  systemctl start falco.service 

root@controlplane ~ ➜  systemctl enable falco.service 
Created symlink /etc/systemd/system/multi-user.target.wants/falco.service → /usr/lib/systemd/system/falco.service.

Task:
Configure falco to save the event output to the file '/opt/falco.log'

Solution:
Edit file /etc/falco/falco.yaml

Change the below entries:

file_output:
  enabled: false
  keep_alive: false
  filename: ./events.txt

to:

file_output:
  enabled: true
  keep_alive: false
  filename: /opt/falco.log

Restart falco service.
root@controlplane ~ ➜  systemctl restart falco.service

Task:

Inspect the API server audit logs and identify the user responsible for the abnormal behaviour seen in the 'citadel' namespace. Save the name of the 'user', 'role' and 'rolebinding' responsible for the event to the file '/opt/blacklist_users' file (comma separated and in this specific order).

Inspect the 'falco' logs and identify the pod that has events generated because of packages being updated on it. Save the namespace and the pod name in the file '/opt/compromised_pods' (comma separated - namespace followed by the pod name)

Solution:

Observe the '/var/log/kubernetes/audit/audit.log' file log mentioned below -

"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"09c1f422-4c4c-4445-8564-2e21a85aaeb6","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldSelector=metadata.name%3Dwebapp-color","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T14:08:21.473864Z","stageTimestamp":"2022-06-23T14:08:21.492509Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}

Based on the log make entries for'user', 'role' and 'rolebinding' responsible for the event to the file '/opt/blacklist_users' file (comma separated and in this specific order).

root@controlplane ~ ➜  echo "agent-smith,important_role_do_not_delete,important_binding_do_not_delete" >/opt/blacklist_users

root@controlplane ~ ✖ tail /opt/falco.log 
14:08:26.401677143: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=4bb5785dd832 container_name=k8s_eden-software2_eden-software2_eden-prime_4c932b2a-32b7-4a6a-b98d-87ad100e54e8_0 image=ubuntu:latest)
14:08:46.410428958: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=4bb5785dd832 container_name=k8s_eden-software2_eden-software2_eden-prime_4c932b2a-32b7-4a6a-b98d-87ad100e54e8_0 image=ubuntu:latest)
14:09:01.774432134: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=4bb5785dd832 container_name=k8s_eden-software2_eden-software2_eden-prime_4c932b2a-32b7-4a6a-b98d-87ad100e54e8_0 image=ubuntu:latest)
14:09:26.753906355: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=4bb5785dd832 container_name=k8s_eden-software2_eden-software2_eden-prime_4c932b2a-32b7-4a6a-b98d-87ad100e54e8_0 image=ubuntu:latest)
14:09:46.755057146: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=4bb5785dd832 container_name=k8s_eden-software2_eden-software2_eden-prime_4c932b2a-32b7-4a6a-b98d-87ad100e54e8_0 image=ubuntu:latest)
14:10:02.091455480: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=4bb5785dd832 container_name=k8s_eden-software2_eden-software2_eden-prime_4c932b2a-32b7-4a6a-b98d-87ad100e54e8_0 image=ubuntu:latest)
14:10:27.071531605: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=4bb5785dd832 container_name=k8s_eden-software2_eden-software2_eden-prime_4c932b2a-32b7-4a6a-b98d-87ad100e54e8_0 image=ubuntu:latest)
14:10:47.071361384: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=4bb5785dd832 container_name=k8s_eden-software2_eden-software2_eden-prime_4c932b2a-32b7-4a6a-b98d-87ad100e54e8_0 image=ubuntu:latest)
14:11:02.296693605: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=4bb5785dd832 container_name=k8s_eden-software2_eden-software2_eden-prime_4c932b2a-32b7-4a6a-b98d-87ad100e54e8_0 image=ubuntu:latest)
14:11:26.372805376: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=4bb5785dd832 container_name=k8s_eden-software2_eden-software2_eden-prime_4c932b2a-32b7-4a6a-b98d-87ad100e54e8_0 image=ubuntu:latest)

root@controlplane ~ ✖ crictl ps | grep 4bb5785dd832
4bb5785dd8326       ubuntu@sha256:b6b83d3c331794420340093eb706a6f152d9c1fa51b262d9bf34594887c2c7ac                   46 minutes ago      Running             eden-software2            0                   5324aa1576e98

root@controlplane ~ ➜  crictl pods | grep 5324aa1576e98
5324aa1576e98       47 minutes ago       Ready               eden-software2                         eden-prime          0                   (default)

root@controlplane ~ ➜  echo "eden-prime,eden-software2" > /opt/compromised_pods

Task:
Delete the role causing the constant deletion and creation of the configmaps and pods in this namespace. Do not delete any other role!

Solution:

root@controlplane ~ ➜  kubectl -n citadel delete role important_role_do_not_delete
role.rbac.authorization.k8s.io "important_role_do_not_delete" deleted

root@controlplane ~ ➜  kubectl -n citadel delete rolebindings.rbac.authorization.k8s.io important_binding_do_not_delete 
rolebinding.rbac.authorization.k8s.io "important_binding_do_not_delete" deleted

Task:

Delete pods belonging to the 'eden-prime' namespace that were flagged in the 'Security Report' file '/opt/compromised_pods'. Do not delete the non-compromised pods!

Solution:

root@controlplane ~ ➜  kubectl -n eden-prime delete pod eden-software2 --force 
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "eden-software2" force deleted

Task:


Delete pods belonging to the 'omega' namespace that were flagged in the 'Security Report' file '/opt/compromised_pods'. Do not delete the non-compromised pods!

Solution:

No action was required for this as this was already showing solved. The audit logs showed as below -

root@controlplane ~ ✖ cat /var/log/kubernetes/audit/audit.log | grep omega
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"172c0875-d0be-4bd6-b406-a26562fd14af","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/configmaps?fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026limit=500\u0026resourceVersion=0","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"omega","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:57:37.605395Z","stageTimestamp":"2022-06-23T13:57:37.605794Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"03d54219-9a32-4b36-a203-83d960b1a0cc","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/omega/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=9217\u0026timeout=8m11s\u0026timeoutSeconds=491\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"omega","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:57:37.608959Z","stageTimestamp":"2022-06-23T13:57:37.609229Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"82607355-5fcd-45e3-924b-c9bfa228f450","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/pods/omega-software4","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"omega","name":"omega-software4","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:57:42.163472Z","stageTimestamp":"2022-06-23T13:57:42.167624Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"3486a2b9-7352-4b8b-a178-744aa0997d20","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/pods/omega-fe-678c4ccf75-bvpdt","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"omega","name":"omega-fe-678c4ccf75-bvpdt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:57:42.362586Z","stageTimestamp":"2022-06-23T13:57:42.365247Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"d4f40d80-196b-478a-84ac-4515b566f4e1","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/pods/omega-software5","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"omega","name":"omega-software5","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:57:42.561723Z","stageTimestamp":"2022-06-23T13:57:42.564046Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"645692a4-d8cb-4395-b65d-1b33daa755ff","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/pods/omega-software6","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"omega","name":"omega-software6","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:57:42.761573Z","stageTimestamp":"2022-06-23T13:57:42.764045Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"c91f8d7e-6ef4-42ca-acda-6f9398f8a35e","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/pods?limit=500","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"kubectl/v1.23.0 (linux/amd64) kubernetes/ab69524","objectRef":{"resource":"pods","namespace":"omega","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:57:58.230342Z","stageTimestamp":"2022-06-23T13:57:58.233539Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"6634e280-12ec-4b6c-bf54-b7f3f3f3ef6b","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/pods/omega-software4","verb":"get","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"kubectl/v1.23.0 (linux/amd64) kubernetes/ab69524","objectRef":{"resource":"pods","namespace":"omega","name":"omega-software4","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2022-06-23T13:57:58.310385Z","stageTimestamp":"2022-06-23T13:57:58.312753Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}