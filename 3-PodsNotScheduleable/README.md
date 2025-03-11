# Node Selector, Node Affinity, Taints and Tolerations

- These things help devops engineers like which pods've to be delpoyed on which nodes, which nodes should avoid certain pods to be scheduled on them.
- If these things are not implemented properly, it will lead pods to go into non scheduleable state which can create downtime for customers

- For this concept we need multi-node K8S cluster (master and worker). Use KIND or K3D for this. KIND helps our entore K8S cluster to run in docker container. Also as our K8S cluster is running inside container, they're very lightweight and we can spin up many K8S clusters and with large no of nodes
- Below K8S cluster is with 4 nodes.
- To get docker running, use docker desktop

![image](https://github.com/user-attachments/assets/c9eaa4c5-6429-4fd6-a7e5-02afe6a701ec)

- To install kind and run below on windows powershell, kind will up running on machine
  - Command :- curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.27.0/kind-windows-amd64
               Move-Item .\kind-windows-amd64.exe c:\some-dir-in-your-PATH\kind.exe

- Now create cluster on kind
  - Command :- **kind create cluster --name=shubham**

![image](https://github.com/user-attachments/assets/8626f826-6c61-487a-a623-5e02eb3bed62)

- So single node cluster is created for us now.

- For multi node cluster, we can write yml file and define our requirements. Get from KIND docs. Using this we can create K8S cluster with N nodes without any performance issues.

![image](https://github.com/user-attachments/assets/4743c6ee-0417-49e4-aa76-f8f5ead6ad94)

- To create multi node K8S cluster using yml file defined. We can see cluster name as well in 
  - Command :- **kind create cluster --name shubham315 --config=kind-cluster.yml**
  - Command :- **kind get nodes**
  - Command :- **kind get clusters**
 
![image](https://github.com/user-attachments/assets/0d61fd16-97f5-4713-a1ee-3091eafd7f09)

- To delete cluster :- **kind delete cluster --name=$name**
- Once we delet cluster and check nodes, we see our kubectl is still pointing to deleted cluster

![image](https://github.com/user-attachments/assets/727d1bbd-83af-4940-a5e4-ca5887b1e6a5)

- Here we can check to which clusters our kube config is connected to :- kubectl config view | grep kind

![image](https://github.com/user-attachments/assets/6e365a8f-dc24-4b9d-ae1d-73d054dbf55b)

- If our kubeconfig is pointing to multiple K8S clusters, using below command we can switch between K8S clusters
  - Command :- **kubectl config use-context kind-multi**
 
![image](https://github.com/user-attachments/assets/61745733-bfc8-4357-b309-69dfd498b833)

-----------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------

#1. Node Selector
-
- Node selector helps us implement whcih pod can be executed on which node.
  - If our application will only work on node of windows
  - So we'll go to node and add label for the node
  - We'll go to pod and add node-selector
  - If not done properly, pod goes to non scheduling status.
  - If we dont define it, K8S will schedule pods acc to its scheduling algorithm

- Write simple deployment.yml. Add "nodeSelector" field and update key-value pair there as **node-name: amd-worker**

![image](https://github.com/user-attachments/assets/d207e22e-0d9b-4ee4-a0f5-f01d4f675a7f)

- Now apply the yml and create :- **kubectl apply -f deployment-node-selector.yml**
- When we check get pods, we see they're in pending status
- Now pick up one pod and describe, and there we can see warning as "Failed Scheduling"

![image](https://github.com/user-attachments/assets/3f5ef430-09fa-4ce2-9e7d-85a43271550e)

- If we dont use node selector properly, our node might go to failed scheduling status
- In above error we can see scheduling is failed from one scheduler, where one node has untolerated taints (ignore this for now)

- Here we've 4 nodes but none of them are available to schedule our pods. If we see deployment yml, we see this node/pods are using nodeselector named arm-worker
- Node selector is a field in K8S pod which helps node to schedule pods only on specific node. As devops engineer within yml file, we'll provide labels so any node with that label exist, schedule pod on that node. If no node, then keep pod in non scheduling state
- If our pod is running an application which only works if it works on arm processor linux machine.  SO we can do scheduling of pods on node with arm processor.
- So within node put that node selector label which tells kube scheduler to schedule on node with that label (node selector is basically a label)

- So acc to requirement we can create cluster with some nodes having arm processor running. We'll go to these nodes (edit node), go to labels section and add **node-name: arm**
  - Command :- **kubectl edit node $NodeName**

<img width="583" alt="image" src="https://github.com/user-attachments/assets/20f9ae79-79c4-4b3a-9e95-fa7105335ef2" />

- Now when we check pods status, we can see them in running state

![image](https://github.com/user-attachments/assets/2704d484-756c-474e-9cb1-7a4220851d32)

-----------------------------------------------------------------------------------------------------------------------------------

#2 Node Affinity
-
- With node selector we force the scheduler to schedule the pod only if you find the right node (hard match). If we dont find the required processor on node, node selector tells scheduler not to schedule pod on that node
- Node affinity feature has 2 options :- Preferred and Required
  - **Required** works almost as node selector where we tell scheduler to schedule only if node is matching 100% requirement
  - **Preferred** option tries to find exact same node where label is provided and if found use that node. If not, we can schedule it anywhere.
 
- With node selector we're doing hard selection. With affinity we provide more flexibility to scheduler if preferred option is used.

- Write yml file like below

![image](https://github.com/user-attachments/assets/2bd68f44-3cd3-4be9-9509-caad07b83918)

  - Here we've changed replica to 1 as its preferred scheduling. Also we've added affinity related configuration which is standard one.
  - Here we've used "preferredDuringSchedulingIgnoreDuringExecution". Prefer a node which has label where key as "**node-arch**" where arch value is "**windows**"

- Apply the yml :- **kubectl apply -f affinity-preferred-scheduler.yml**
- Check pods. Pods are running. That pod gets scheduled on any node as we've used "preferred" scheduling

![image](https://github.com/user-attachments/assets/12153756-c8cc-4ae9-8906-601c880e49d8)

- If we change the yml to key as "node-name" and value as "arm-worker" which was configured previously on our cluster and apply the yml
  - This time pod gets scheduled on "multi-worker" which has configuration named "arm-worker". Here K8S found node which is our preferred config which is already available and schedules pod there
 
- If we change it to "reuired", only instead of "preferredDuringSchedulingIgnoredDuringExecution", we've to enter "requiredDuringSchedulingIgnoredDuringExecution" and it gets scheduled on 100% match pod.

Difference between Node selector and node affinity
-
- Although the features are little similar, node affinity grants more granular level of configuration or more flexibility for devops engineer. It checks for preferred configuration if not found it will schedule anywhere

-----------------------------------------------------------------------------------------------------------------------------------

#3 Taints
-
- Till now in node selector and affinity, we're talking from pod POV.
- If we've K8S nodes and we dont want to schedule anything there, any pod should not be scheduled. This requirement can be due to upgrades
- If we've prod K8S cluster with specific version having 3 worker nodes. We've to upgrade the K8S cluster to new version. We cannot upgrade all worker nodes at once.
  - In this case we drain particular node where we'll move all pods running on it to different node, then we make that node unscheduleable and idle. Kube scheduler will not schedule anythin on this node.
  - Then we can bring this node down, upgrade it and remove unscheduleable status and make newer version up and running
 
- There are multiple ways of doing K8S upgrades.

- Here taints is used. It is a label or stamp which we apply to node which explains kube scheduler to make node **unscheduleable** or make it not **NoExecutable** status or we can do **prefereNoSchedule** in node affinity
- 3 types of taints
  - **No Schedule** :- Node will go to non scheduleable status
  - **No Execute** :- Dangerous. All pods on our node will immediately stopped working
  - **PreferredNoSchdule** :- If we've node whose performance is declining (CPU, memory issues), node not behaving as expected, we can use this taint. It is worst case scenario

- To taint use command :- **kubectl taint nodes node1 key1=value1:NoSchedule**
  - No schedule is the taint we're applying to one of the nodes and key value pair will come into picture when we do tolerations
 
![image](https://github.com/user-attachments/assets/9377303e-b4b4-499a-8eb6-ba78a0ef4943)

- Now edit our master node :- kubectl edit node $master
  - We can see this node already has taint named "NoSchedule". This is because usually we dont schedule pods on control plane. So all our pods go to worker nodes

![image](https://github.com/user-attachments/assets/28dcad94-0111-4ffb-8e17-90a2ba0ebab5)

- If we make all worker nodes tainted as "NoSchedule" except one of the worker noded, our pods should get scheduled on that node only
  - So when we apply the deployment and get pods we can see all pods get scheduled on that remaining not which is not tainted.
 
- Using taints we can decide behavior of our node during upgrades and make node as unscheduled and upgrade version of K8S

-----------------------------------------------------------------------------------------------------------------------------------

#4 Tolerations
-
- It is exception which is given to certain pods.
- Lets say we've 3 nodes in NoSchedule status but for some reason we want certain high priority pods which are critical for prod and performance
- To these pods we can add toleration or exception we can still run on NoSchehule or any tainted loop.
- In any of the states we can still run pods if we've tolerations set on that pod.

- While tainting node, we used key value pair for tolerations
  - NoSchedule is effect on node but if we still add tolerations to pods which're matching these key-value pairs of our effect, scheduler will understand there is pod which has tolerations matching key value pair, we can still schedule resources on that node
 
- Create yaml and add tolerations field there. We can put any values for key and value

![image](https://github.com/user-attachments/assets/0e0609cc-d328-4e91-a59d-4bc390852318)
 
- Apply the yml and make all node unscheduleable (taint them) :- **kubectl apply -f tolerations.yml**
  - Ideally all nodes are unscheduled so nothing is unavlailable
  - But check to get pods. One-one copy is running on worker1, worker2 and worker3
  - This is due to pods are tolerated. Tolerations help you schedule pods.
 
![image](https://github.com/user-attachments/assets/86a08dba-cd6c-4857-9aa6-c9815ad76cd5)



