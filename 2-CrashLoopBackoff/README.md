# CrashLoopBackoff Error

- If our pod is not in the running state and there is issue with app/container/pod, pod status we see as "CrashLoopBackoff".
- This dont happen due to single reason, there can be multiple reasons when our pod gets into this state.
- CrashLoopBackoff is basically a pod status. When we do "**_kubectl get pods_**", for some pods we might see status as **CrashLoopBackoff**
- This error arise due to pods getting crashed multiple times in a loop, K8S will try to tell status of pods as **CrashLoopBackoff** (that our pod is getting crashed multiple times)

- Suppose, We've developer writing a "Python Flask Application". He has tested application on his local machine which is working fine. He came to DevOps engineer, asks to write dockerfile, create container and deploy app as pod on K8S cluster. So we wrote a dockerfile and created a deployment.yml manifest. Using apply -f we created deployment then request went to API server, from API server scheduler identifies a suitable node for our application/pod and scheduler will forward the request to kubelet on that particular node.
- Now kubelet will try to run our K8S pod. Initially, K8S pod is just container and application. Pod has container and within container we've our application running. Firstly our pod will move to "**ContainerCreatingStatus**". After container creation 2 things can happen, either container and app inside it runs seamlessly (pod status will be running) or due to some issue we get error while running pod/container (pod status can be error). Sometimes our pod might get crashed.
- Lets say initially our pod is in running state. After a while (min or days) it gets crashed as it didn't get as many resources needed like CPU, memory,etc.
- Now as our pod is in crashed state, K8S will not leave it in crashed state, it will try to restart the pod due to K8S's **default restart pod policy**. It will again go to "Container Creating" status.

- Cycle of Container/Pod Creation:-  **Container Creation --> Pod moves to running status or Error starting pod status --> Gets crashed --> K8S Tries to restart by default restart pof policy**

- This cycle repeats in loop which is called **"CrashedLoopBackoff"**

- **BackoffDelay** :- When K8S tries to restart, K8S will try to restart pod after 10 sec. Next time it will restart in incremental manner increasing delay.

- **CrashLoppBackoff** :- Our K8S pod is crashed and its happening again and again so pod is getting crashed multiple times in a loop. And K8S is trying to restart the pod using Backoff Delay (incremental)

# Note:- CrashedLoopBackoff doesn't happen due to one single error. There can be multiple reasons. If our pod/container was initially in running status, there can be multiplee reasons for our pod to get crashed

Below are some:-

Misconfigurations
-
- If we provide incorrect env variable for our container, give reference of non-existing persistent volumes
- Misconfigurations can encompass a wide range of issues, from incorrect environment variables to improper setup of service ports or volumes. These misconfigurations can prevent the application from starting correctly, leading to crashes. For example, if an application expects a certain environment variable to connect to a database and that variable is not set or is incorrect, the application might crash as it cannot establish a database connection.

Errors in the Liveness Probes
-
- For some reasons our liveness/readiness probe might fail, then also our container can go into running state. As our liveness probe is failing, container might get crashed and it can go to CrashLoopBackoff state
- Liveness probes in Kubernetes are used to check the health of a container. If a liveness probe is incorrectly configured, it might falsely report that the container is unhealthy, causing Kubernetes to kill and restart the container repeatedly. For example, if the liveness probe checks a URL or port that the application does not expose or checks too soon before the application is ready, the container will be repeatedly terminated and restarted.

The Memory Limits Are Too Low
-
- If memory resources allocated to our already running container are not enough, it gets crashed
- If the memory limits set for a container are too low, the application might exceed this limit, especially under load, leading to the container being killed by Kubernetes. This can happen repeatedly if the workload does not decrease, causing a cycle of crashing and restarting. Kubernetes uses these limits to ensure that containers do not consume all available resources on a node, which can affect other containers.

Wrong Command Line Arguments
-
- If CLI arguments we need to pass are "python 3 app.py" to start the application, but instead we misconfigured to "python 3 app1.py"
- Containers might be configured to start with specific command-line arguments. If these arguments are wrong or lead to the application exiting (for example, passing an invalid option to a command), the container will exit immediately. Kubernetes will then attempt to restart it, leading to the CrashLoopBackOff status. An example would be passing a configuration file path that does not exist or is inaccessible.

Bugs & Exceptions
-
- Bugs in the application code, such as unhandled exceptions or segmentation faults, can cause the application to crash. For instance, if the application tries to access a null pointer or fails to catch and handle an exception correctly, it might terminate unexpectedly. Kubernetes, detecting the crash, will restart the container, but if the bug is triggered each time the application runs, this leads to a repetitive crash loop.

-----------------------------------------------------------------------------------------------------------------------

# Practical Demo

1. Wrong Command Crashloop 
-
- Suppose we've simple python app given by developer and inside which we've file named app.py

![image](https://github.com/user-attachments/assets/e78a98b5-45e0-464f-912f-d8ceed20bd5e)

- Suppose, in dockerfile's entrypoint instead of app.py in entrypoint we've defined app1.py. After execution our pod will go to container creating state. Then it will go to error state instead of running state. Then pod will get crashed. Again pod will try to restart and cycle continues.

![image](https://github.com/user-attachments/assets/f5079281-aa4a-4a92-b507-bacddd24ed7e)

- No first build the dockerfile and push to dockerhub and use deployment.yml to apply to K8S cluster
- Now build image with improper entrypoint :- **docker build -t shubham0315/crashlooptest:v1 .**

<img width="766" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/79020ebb-253b-43ed-a848-32a2c4047a9f">

- Now push the image to dockerhub :- **docker push shubham315/crashlooptest:v1**

<img width="774" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/02b18041-4086-40b8-bb46-5782618fc8e2">

- Now we've **"wrong-cmd-crashloop.yml"**. Edit our docker registry name and apply the deployment to create one. Now we can see the image/pod gets into crashLoopBackoff state
- Command :- **kubectl apply -f wrong-CLI-args.yml**
- Intially it came into "containerCreating" state, then went to "Error" state then again "CrashLoopBackoff" and again "error"

<img width="782" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/f0dc1630-7bbf-43cf-8c34-e056b7cc432b">

- To fix this, change the dockerfile entrypoint to correct one and again build then push image to dockerhub changing image version (v1 to v2 while passing CLI command tag where V1 is crashLoopBackoff and V2 is actual).
- After build and push, go to deployment file and change version there also (tag to V2). Now apply the deployment, we can see container from error status to running state.

<img width="779" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/ad1d9f91-6e3e-4db1-aa2a-20f8a816d619">

- While creating actual running container, when we apply perfect yml, we can see how many times image went into crashLoopBackoff (restarts 4 in below SS) and previous pods getting terminated which had errors. Then new pod gets into running status

![image](https://github.com/user-attachments/assets/31b803b4-d42e-40c1-822e-4d671c35490e)


2. Liveness Probe
-
- Here the yml file is same just liveness probe is added.

![image](https://github.com/user-attachments/assets/4d93080b-0132-4a6d-bee5-7e0dfda32731)

- There are 2 probes in K8S :- liveness and readiness.
- Liveness probe is Similar to LB concept
- How LB understand targets where it has to forward the request are healthy or not? LB will continuously check health of VM, if good, the user request is forwarded to the VM.
- If we've multiple VMs and both are in healthy state and user requests something, it will go to VMs depending on the algorithm we've chosen.(Round robin)
- LB have a way to check health of its targets.

- Similarly in K8S, kubelet verifies if pod is healthy or not if we configure "Liveness Probe". If liveness probe passes, pod is healthy, if fails pod is unhealthy.
- If liveness probe is failing, kubelet will immediately restart our pod. If it doesnt restart and it gets customer request, customer will not receive any response as pod/app is hung.
- To avoid this, we can configure **Liveness Probe** for our pod which acts as health check. Kubelet checks this health using parameters or script provided to it.

- In our yml, we just added liveness probe and ask it to check the file "/tmp/healthy". If the file does not exist, liveness probe will fail and then kubelet will try to restart it

- Firstly apply the created deployment :- **kubectl apply -f liveness-probe-crashloop.yml**
- Again our cycle starts :- container creating - Container moves to running status - In running status after 5 secs liveness probe will trigger - liveness probe will fail - kubelet will restart our container - again goes to container creating state
- So our container goes to "**crashLoopBackoff**"

- As we've provided periodSeconds as 1, after 1 sec it will restart the pod after getting into running state. Then it will wait for probe for 5 secs to get executed.
- Eventually after some iterations we see status of pod as "crashLoopBackoff"

![image](https://github.com/user-attachments/assets/d5b89d2f-1308-416b-b4ac-bffb586ca924)

- In the real environment, we'll not create liveness probe, we get it from dev. This is because in liveness probe we get API of kubelet to hit, context path. So when kubelet tries to hit API, it will receive response as 200 or 500, depending on which kubelet understand it has to restart pod or not.

![image](https://github.com/user-attachments/assets/37bf2dd2-0077-492f-b4be-c270d7eee188)

Liveness probe is used to check if our pod is healthy or not
-
- Readiness probes is used to check if our pod is ready to receive traffic or not
-
- 







