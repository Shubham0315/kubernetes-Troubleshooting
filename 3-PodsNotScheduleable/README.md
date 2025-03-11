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

#1. Node Selector

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

