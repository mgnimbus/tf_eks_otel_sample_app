# aws_eks_tf
Deploy AWS EKS Cluster using terraform 


## Configure kubeconfig for kubectl
```t
# Configure kubeconfig for kubectl
aws eks --region <region-code> update-kubeconfig --name <cluster_name>
aws eks --region us-east-1 update-kubeconfig --name meda-dev-ram-eksdemotest

```
Added new context arn:aws:eks:us-east-1:328268088738:cluster/meda-test-eksdemo1 to C:\Users\medag\.kube\config

# Verify EBS CSI Controller and Node pods running in kube-system namespace
kubectl -n kube-system get pods
```


kubectl apply -f 01_storage_class.yaml

storageclass.storage.k8s.io/ebs-sc created

kubectl apply -f 02_pvc_claim.yaml

persistentvolumeclaim/ebs-mysql-pv-claim created

kubectl apply -f .\03_app_config_map.yaml 
configmap/usermanagement-dbcreation-script created


kubectl apply -f .\04_mysqldb_deployment.yaml
deployment.apps/mysql created
service/mysql created


kubectl apply -f .\05_app_deployment.yaml


## Step-11: Verify Kubernetes Resources created
```t
# Verify Storage Class
kubectl get storageclass
kubectl get sc
Observation:
1. You should find two EBS Storage Classes
  - One created by default with in-tree EBS provisioner named "gp2". Future it might get deprecated
  - Recommended to use EBS CSI Provisioner for creating EBS volumes for EKS Workloads
  - That said, we should the one we created with name as "ebs-sc"

# Verify PVC and PV
kubectl get pvc
kubectl get pv
Observation:
1. Status should be in BOUND state

# Verify Deployments
kubectl get deploy
Observation:
1. We should see both deployments in default namespace
- mysql
- usermgmt-webapp

# Verify Pods
kubectl get pods
Observation:
1. You should see both pods running

# Describe both pods and review events
kubectl describe pod <POD-NAME>
kubectl describe pod mysql-6fdd448876-hdhnm
kubectl describe pod usermgmt-webapp-cfd4c7-fnf9s

# Review UserMgmt Pod Logs
kubectl logs -f usermgmt-webapp-cfd4c7-fnf9s
Observation:
1. Review the logs and ensure it is successfully connected to MySQL POD

# Verify Services
kubectl get svc
```

## Step-12: Connect to MySQL Database Pod
```t
# Connect to MySQL Database 
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pdbpassword11

# Verify usermgmt schema got created which we provided in ConfigMap
mysql> show schemas;
mysql> use webappdb;
mysql> show tables;
mysql> select * from user;

## Sample Output for above query
+--------+----------------------------+------------+-----------+--------------------------------------------------------------+--------+-----------+
| userid | email_address              | first_name | last_name | password                                                     | ssn    | user_name |
+--------+----------------------------+------------+-----------+--------------------------------------------------------------+--------+-----------+
|    101 | admin101@stacksimplify.com | Kalyan     | Reddy     | $2a$10$w.2Z0pQl9K5GOMVT.y2Jz.UW4Au7819nbzNh8nZIYhbnjCi6MG8Qu | ssn101 | admin101  |
+--------+----------------------------+------------+-----------+--------------------------------------------------------------+--------+-----------+
1 row in set (0.00 sec)
mysql> 

Observation:
1. If UserMgmt WebApp container successfully started, it will connect to Database and create the default user named admin101
Username: admin101
Password: password101
```
## Step-13: Access Sample Application
```t
# Verify Services
kubectl get svc

# Access using browser
http://<CLB-DNS-URL>
http://<NLB-DNS-URL>
Username: admin101
Password: password101

# Create Users and Verify using UserMgmt WebApp in browser
admin102/password102
admin103/password103

# Verify the same in MySQL DB
## Connect to MySQL Database 
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pdbpassword11

## Verify usermgmt schema got created which we provided in ConfigMap
mysql> show schemas;
mysql> use webappdb;
mysql> show tables;
mysql> select * from user;
```



## Step-14: Node Port Service Port - Update Node Security Group
- **Important Note:** This is not a recommended option to update the Node Security group to open ports to internet, but just for learning and testing we are doing this. 
- Go to Services -> Instances -> Find Private Node Group Instance -> Click on Security Tab
- Find the Security Group with name `eks-remoteAccess-`
- Go to the Security Group (Example Name: sg-027936abd2a182f76 - eks-remoteAccess-d6beab70-4407-dbc7-9d1f-80721415bd90)
- Add an additional Inbound Rule
   - **Type:** Custom TCP
   - **Protocol:** TCP
   - **Port range:** 31280
   - **Source:** Anywhere (0.0.0.0/0)
   - **Description:** NodePort Rule
- Click on **Save rules**


## Step-15: Access Sample using NodePort Service 
```t
# List Nodes
kubectl get nodes -o wide
Observation: Make a note of the Node External IP

# List Services
kubectl get svc
Observation: Make a note of the NodePort service port "myapp1-nodeport-service" which looks as "80:31280/TCP"

# Access the Sample Application in Browser
http://<EXTERNAL-IP-OF-NODE>:<NODE-PORT>
http://54.165.248.51:31280
Username: admin101
Password: password101
```

## Step-16: Remove Inbound Rule added  
- Go to Services -> Instances -> Find Private Node Group Instance -> Click on Security Tab
- Find the Security Group with name `eks-remoteAccess-`
- Go to the Security Group (Example Name: sg-027936abd2a182f76 - eks-remoteAccess-d6beab70-4407-dbc7-9d1f-80721415bd90)
- Remove the NodePort Rule which we added.

## Step-17: Clean-Up
```t
# Delete Kubernetes  Resources
kubectl delete -f app_db_manifests

storageclass.storage.k8s.io "ebs-sc" deleted
persistentvolumeclaim "ebs-mysql-pv-claim" deleted
configmap "usermanagement-dbcreation-script" deleted
deployment.apps "mysql" deleted
service "mysql" deleted
deployment.apps "usermgmt-webapp" deleted
service "usermgmt-webapp-nodeport-service" deleted

# Verify Kubernetes Resources
kubectl get pods
kubectl get svc
```