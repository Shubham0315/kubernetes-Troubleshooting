# CrashLoopBackoff Error

- CrashLoopBackoff is basically a pod status. When we do "kubectl get pods", for some pods we might see status as **CrashLoopBackoff**
- This error arise due to pods getting crashed multiple times in a loop, K8S will try to tell status of pods as CrashLoopBackoff

- We've developer writing a "Python Flask Application". He has tested application on his local machine which is working fine. He came to DevOps engineer, asks to write dockerfile, create container and deploy app as pod on K8S cluster. So we wrote a dockerfile and created a deployment manifest. Using apply -f we created deployment then request went to API server, from API server scheduler identifies a suitable node for our application/pod and scheduler will forward the request to kubelet on that particular node.
- Now kubelet will try to run our K8S pod. Initially, K8S pod is just container and applictaion. Pod has container and within container we've our application running. Firstly our pod will move to "**ContainerCreatingStatus**". After container creation 2 things can happen, either container and app inside it runs seamlessly or due to some issue we get error while running pod/container. Sometimes our pod might get crashed.
- Lets say initially our pod is in running state. After a while (min or days) it gets crashed as it didn't get as many resources needed like CPU, memory,etc.
- Now as our pod is in crashed state, K8S will not leave it in crashed state, it will try to restart the pod due to K8S default restart pod policy. It will again go to "Container Creating" status.

- Cycle of Container/Pod Creation:-  **Container Creation --> Error starting pod --> Gets crashed --> Tries to restart**

- This cycle repeats in loop which is called **"CrashedLoopBackoff"**

- **BackoffDelay** :- When K8S tries to restart, K8S will try to restart pod after 10 sec. Next time it will restart in incremental manner increasing delay.

- **CrashLoppBackoff** :- Our K8S pod is crashed and its happening again and again so pod is getting crashed multile times in a loop. And K8S is trying to restart the pod using Backoff Delay
