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

- 
