Persistent Volume
-
- PV is a piece of storage in cluster that has been provisioned by administrator or dynamically by a StorageClass
- It like physical or cloud disk made available to cluster

Persistent Volume Claims
-
- PVC is a request for storage by user
- It specifies how much storage and what access mode is needed (RW, ReadOnly)
- K8S then tries to match PVC with available PV that satisfies the claim

  - Admin creates PV
  - Dev/User creates PVC to request storage
  - K8S binds PVC to appropriate PV automatically
  - Pod uses PVC to mount volume and access storage

# K8S Troubleshooting with Stateful Sets and Persistent Volumes

- Using JIRA bug request, developer reports issue to devops engineer where he mentions one of the stateful sets apps is working fine on AWS EKS but not working on other cloud platforms or local minikube clusters.
- This ticket went to DevOps engineer as he is SME for K8S. Developers also share yml for stateful set or any K8S resource they've issue with in bug request.
- Below is the example of yml for stateful set where we create nginx app with 3 replicas. We also attach persistent volume claim template where we use storage class as "ebs"

![image](https://github.com/user-attachments/assets/1829bd3a-76d2-4d07-8181-7707524a445e)

- Apply this yml and create stateful set :- **kubectl apply -f statefulset.yml**

- Check the stateful set :- **kubectl get statefulset**
- Here we can see none of the pod is in running status

![image](https://github.com/user-attachments/assets/7b4ce111-baf2-4166-a4e5-1574aae63133)

- There has to be issue with pods so check them. So when we check get pods we can see only 1 replica is there instead of 3 which is also in pending state

![image](https://github.com/user-attachments/assets/e79f95c7-e0fe-4e01-9ce2-89da3962d7fd)

- When we check describe pod we can see the warning which says "pod has immediate persistent volume claims"
  - Here we're trying to deploy pod through stateful set but that pod is requesting persistent volume but is unable to find it.
  - In stetful set, even we've 3 replicas 2nd replica will be created only if 1st is successfully scheduled. Thats why we see only one pod while we check get pods
  - If the first moves from pending to seccess state, 2nd moves to container creating status. (Unique nature of stateful set)
 
- Why we're unable to find persistent volume?
- When we create steful set and request for persistent volumes, developer mentions persistent volume clain inside stateful set. It is claim request stateful set is making
- In stateful set, we can say we need 1 GB persistent volume. Every PVC has storage class which using provision creates persistent volumes

# Workflow
- Stateful set -- Persistent volume claim --Storage class -- Provisioner -- Persistent volume

- Suppose our stateful set works fine on AWS EKS. If we look at statefulset.yml, in volume claim template, storage class name is "ebs" which means in AWS env.
  - In AWS there are storage service like ebs, efs, fsx. So in stateful set we define storage class type we're looking for.
  - So on AWS EKS there is "ebs provisioner" which creates "ebs storage" which is used as persistent volumes by stateful states
 
- When dev tries to deploy on AKS,GKE, minikube, it fails.
- In minikube, when we create SS, within it we provide PVC. If SS has to be run on minikube, PVC should have storage class name as standard. When we put this on minikube there is a provisioner (kubectl get pods -A), standard provisioner is there.We can mentione provisioner name as standard and that will create PV for us.

- So inside yml, we ccan change storage class to "standard" for minikube and apply the yml again to create stateful set deleting old one.

- Also we need to delete PVCs which are old so SS are deleted explicitly.

![image](https://github.com/user-attachments/assets/f72a3db9-a442-49a4-a51c-b0b182f9f7cc)

- Now apply yml and check for pods. They'll get created one by one.

![image](https://github.com/user-attachments/assets/c16c5484-e001-48a3-a950-b87772b05044)

- Now all three pods are running.

- What if we want to use external provisioner or a persistent volume from external storage service??
  - There is CSI driver for it.
  - There are some default storage services for K8S so code for the storage provisioner is there.
  - Lets we want to use Netapp storage service on EKS. Here NETAPP will provide CSI driver, we've to install it on EKS cluster(Container Storage Interface). That driver will act as provisioner which will create PV on NETAPP
 
  - 
# Workflow:-
-To create SS with external storage, in SS yml we've to provide PVC template, provide storage class (NETAPP), install CSI driver on EKS, it acts as provisioner which create PV

