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

- For multi node cluster, we can write yml file and define our requirements. Get from KIND docs
