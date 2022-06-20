Kodekloud CKS Challenge - 1 Solution


1. There are 6 images listed in the diagram on the right. Using Aquasec Trivy (which is already installed on the controlplane node), identify the image that has the least number of critical vulnerabilities and use it to deploy the alpha-xyz deployment.

Secure this deployment by enforcing the AppArmor profile called custom-nginx.

Expose this deployment with a NodePort type service and make sure that only incomings connections from the pod called middleware is accepted and everything else is rejected.

Permitted images are: 'nginx:alpine', 'bitnami/nginx', 'nginx:1.13', 'nginx:1.17', 'nginx:1.16'and 'nginx:1.14'. Use 'trivy' to find the image with the least number of 'CRITICAL' vulnerabilities.

Solution:

root@controlplane ~ ➜ trivy image nginx:alpine | grep -i total
Total: 4 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 2, CRITICAL: 2)

root@controlplane ~ ➜  trivy image bitnami/nginx | grep -i total
Total: 107 (UNKNOWN: 0, LOW: 16, MEDIUM: 41, HIGH: 43, CRITICAL: 7)

root@controlplane ~ ➜  trivy image nginx:1.13 | grep -i total
Total: 471 (UNKNOWN: 9, LOW: 22, MEDIUM: 176, HIGH: 183, CRITICAL: 81)

root@controlplane ~ ➜  trivy image nginx:1.17 | grep -i total
Total: 278 (UNKNOWN: 1, LOW: 15, MEDIUM: 112, HIGH: 109, CRITICAL: 41)

root@controlplane ~ ➜  trivy image nginx:1.16 | grep -i total
Total: 282 (UNKNOWN: 1, LOW: 15, MEDIUM: 114, HIGH: 111, CRITICAL: 41)

root@controlplane ~ ➜  trivy image nginx:1.14 | grep -i total
Total: 396 (UNKNOWN: 7, LOW: 18, MEDIUM: 161, HIGH: 150, CRITICAL: 60)

Image nginx:alpine is having the least CRITICAL vulnerabilities as seen from above output.

root@controlplane /etc/apparmor.d ➜  kubectl -n alpha create deployment alpha-xyz --image=nginx:alpine
deployment.apps/alpha-xyz created

Solution:

root@controlplane ~ ➜  mv usr.sbin.nginx /etc/apparmor.d/
root@controlplane ~ ➜  cat /etc/apparmor.d/usr.sbin.nginx | grep -i profile
profile custom-nginx flags=(attach_disconnected,mediate_deleted) {

root@controlplane ~ ➜  cd /etc/apparmor.d
root@controlplane /etc/apparmor.d ➜  apparmor_parser ./usr.sbin.nginx 

Update with apparmor annotation in deployment:
-- SNIPPED -- 
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: alpha-xyz
      annotations:
        container.apparmor.security.beta.kubernetes.io/nginx: localhost/custom-nginx
    spec:
      containers:
      - image: nginx:alpine
    
-- SNIPPED --   

Solution:

root@controlplane ~ ➜  kubectl -n alpha get pv alpha-pv 
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    REASON   AGE
alpha-pv   1Gi        RWX            Delete           Available           local-storage            31m

root@controlplane ~ ➜  kubectl -n alpha get pvc alpha-pvc 
NAME        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    AGE
alpha-pvc   Pending                                      local-storage   31m

-


 name: alpha-pvc
  namespace: alpha
  resourceVersion: "13782"
  uid: 36abd81c-55f2-4e5c-953f-4a3d23520d13
spec:
  accessModes:
  - ReadWriteMany   ---> Changed to match PV

 Delete and Recreate pvc with new YAML

  root@controlplane ~ ➜ kubectl -n alpha get pv alpha-pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS    REASON   AGE
alpha-pv   1Gi        RWX            Delete           Bound    alpha/alpha-pvc   local-storage            35m

root@controlplane ~ ➜  kubectl -n alpha get pvc alpha-pvc 
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
alpha-pvc   Bound    alpha-pv   1Gi        RWX            local-storage   32s

Solution:
Update the volumeMount alpha-pvc in deployment:
-- SNIPPED --                
   spec:
      volumes:
      - name: data-volume
        persistentVolumeClaim:
           claimName: alpha-pvc
      containers:
      - image: nginx:alpine
        imagePullPolicy: IfNotPresent
        name: nginx
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: data-volume
-- SNIPPED -- 

Solution: 

root@controlplane ~ ➜ kubectl -n alpha expose deployment alpha-xyz --type=ClusterIP --port=80 --target-port=80 --dry-run=client -o yaml >alpha-xyz-svc.yaml

Modify alpha-xyz-svc.yaml

root@controlplane ~ ➜  cat alpha-xyz-svc.yaml 
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: alpha-svc
  name: alpha-svc
  namespace: alpha
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: alpha-xyz
  type: ClusterIP
status:
  loadBalancer: {}

Expose deployment with given service name.
  root@controlplane ~ ➜  kubectl create -f alpha-xyz-svc.yaml 
service/alpha-svc created

Solution:

root@controlplane ~ ➜  cat restrict-inbound.yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-inbound
  namespace: alpha
spec:
  podSelector:
    matchLabels:
      app: alpha-xyz
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: middleware
      ports:
        - protocol: TCP
          port: 80

root@controlplane ~ ➜  kubectl create -f restrict-inbound.yaml 
networkpolicy.networking.k8s.io/restrict-inbound created