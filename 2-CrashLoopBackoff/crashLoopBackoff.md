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

# Note:- CrashedLoopBackoff doesn't happen due to one single error. There can be multiple reasons. Below are some:-

1. Misconfigurations
- Misconfigurations can encompass a wide range of issues, from incorrect environment variables to improper setup of service ports or volumes. These misconfigurations can prevent the application from starting correctly, leading to crashes. For example, if an application expects a certain environment variable to connect to a database and that variable is not set or is incorrect, the application might crash as it cannot establish a database connection.

2. Errors in the Liveness Probes
- Liveness probes in Kubernetes are used to check the health of a container. If a liveness probe is incorrectly configured, it might falsely report that the container is unhealthy, causing Kubernetes to kill and restart the container repeatedly. For example, if the liveness probe checks a URL or port that the application does not expose or checks too soon before the application is ready, the container will be repeatedly terminated and restarted.

3. The Memory Limits Are Too Low
- If the memory limits set for a container are too low, the application might exceed this limit, especially under load, leading to the container being killed by Kubernetes. This can happen repeatedly if the workload does not decrease, causing a cycle of crashing and restarting. Kubernetes uses these limits to ensure that containers do not consume all available resources on a node, which can affect other containers.

4. Wrong Command Line Arguments
- Containers might be configured to start with specific command-line arguments. If these arguments are wrong or lead to the application exiting (for example, passing an invalid option to a command), the container will exit immediately. Kubernetes will then attempt to restart it, leading to the CrashLoopBackOff status. An example would be passing a configuration file path that does not exist or is inaccessible.

5. Bugs & Exceptions
- Bugs in the application code, such as unhandled exceptions or segmentation faults, can cause the application to crash. For instance, if the application tries to access a null pointer or fails to catch and handle an exception correctly, it might terminate unexpectedly. Kubernetes, detecting the crash, will restart the container, but if the bug is triggered each time the application runs, this leads to a repetitive crash loop.

# Practical Demo

1. Wrong Command Crashloop 

- Suppose, in dockerfile instead of app.py in entrypoint we've defined app1.py. After execution our pod will go to container creating state. Then it will go to error state instead of running state. Then pod will get crashed. Again pod will try to restart and cycle continues.

- Now build image with improper entrypoint :- docker build -t shubham0315/crashlooptest:v1 .

<img width="766" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/79020ebb-253b-43ed-a848-32a2c4047a9f">

- Now push the image to dockerhub :- docker push shubham315/crashlooptest:v1

<img width="774" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/02b18041-4086-40b8-bb46-5782618fc8e2">

- Now we've **"wrong-cmd-crashloop.yml"**. Edit our docker registry name and apply the deployment to create one. Now we can see the image/pod gets into crashLoopBackoff state

<img width="782" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/f0dc1630-7bbf-43cf-8c34-e056b7cc432b">

- To fix this, change the dockerfile entrypoint to correct one and again build then push image to dockerhub changing image version (v1 to v2).
- After build and push, go to deployment file and chage version there also. Now apply the deployment, we can see container from error status to running state.

<img width="779" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/ad1d9f91-6e3e-4db1-aa2a-20f8a816d619">

2. 






