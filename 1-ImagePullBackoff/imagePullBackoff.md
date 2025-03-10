# kubernetes-Troubleshooting

# 1. ImagePullBackoff Error

- When we start with K8S, firstly we deploy pod onto our K8S cluster. ImagePullBackoff is related to this task.
- This error is related to pulling container image on K8S cluster.
- When we deploy our app on K8S cluster, it can be using pod, deployments, stateful states, daemon set or replica sets where we deploy applications as container image on K8S cluster. Here we can run into this error.
- This can be arised due to 2 scenarios :-

  a.  Invalid image name or non-existent image name
  b.  Private Images (secure) in dockerhub

* _**Invalid Image**_ :- If we want to deploy nginx application on K8S cluster, and we've taken yml from official K8S docs. But while creating pod/deployment using yml apply, we mistyped "**nginx**" as "**nginy**", it gets invalid image name as no such image will be there on dockerhub. So we get above error.

* _**Non-existent Image**_ :- Similarly in our organization suppose there is image "foo:1.1.1" (image with tag) but mistakenly the image was deleted but there is deployment created means we have deployed application which is pointing towards **non-existent image**. So we get above error.

* We're trying to deploy application as container image but image is private. In dockerhub, most of the images are public to pull or push. Some images are private that noone without owner's authorization can download the image. Here also we can run into "**ImagePullBackoff**" error. To deploy the private image, we've to use concept of "**ImagePullSecret**".
* Here, we can create nginx-deploy.yml. There create ImagePullSecret and pass name of secret which stores docker credentials

- To check by which scenario we landed into this error, there are ways :- Use describe command or Use events command.

**What is Backoff?**
-
- It means Backoff Delay.
- If we've provided invalid image name, K8S wont throw ImagePullBackoff error directly. It throws "**ErrorImagePull**". After this error, K8S will wait for sometime as this error can be due to Network Issue as well. Due to Network issue, kubelet which is running our pod, might not be able to pull the image or there can be some intermittent issues. So K8S wait for sometime and try one more time. Again it will wait and try one more time. Tis is continuous process where K8S will incrementally increase the wait time. (Initially 5 sec, 20 sec and till 5 mins).
- So K8S will not just attempt to pull container image once, it will attempt multiple times and each time it will add delay incrementally and try to pull the image. This process is called as "_**Backoff Delay**_"

- That's why the error is called "**ImagePullBackoff**"

- In short, the error is related to pulling image on your K8S cluster if image is invalid, non-existent, private.
- Initial error thrown by K8S is **ErrImagePull** and later it will try to pull image with incremental delay. Then the error is converted to "**ImagePullBackoff**"

# Practical demo for Image Pull Backoff

1. Write yml file to create deployment (Refer from internet)
   <img width="773" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/c2ccf5b0-c672-4616-98ad-f8f49b02d01d">

   <img width="695" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/02e11091-2314-4c9e-af6f-fb15bc21bc6c">
- We can check container/pod creation using :- kubectl get pods -w (watch continuously and report status)

2. As the Image is valid and existing, delete the created deployment
- Now edit yml file and change "nginx" to "nginy"
- Again apply the deployment which gets created but when we do "kubectl get pods -w", we get below

   <img width="778" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/bed9a701-5348-4d7f-bc6c-e677b7f8f596">
- As we can check, ErrImagePull gets eventually transformed to ImagePullBackoff.

3. ImagePullBackoff in Private repository
- Firstly, create container image on dockerhub repository and make the repository private.
- Pull image :- docker pull nginx:1.14.2

<img width="773" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/4c826a10-a74e-4150-b52f-468d7257f5cf">

- Now to tag image :- docker tag nginx:1.14.2 shubham315/nginx-image:v1

<img width="779" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/b820e4b0-1bdb-449c-8de0-bcfd34299a52"> 

- Now to push image to dockerhub. So it creates "nginx-image" repo on our dockerhub :- docker push shubham315/nginx-image:v1

<img width="781" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/b075387c-80bd-480e-abf9-01fab5662219">

- Here by default container repository will be public. Now go to dockerhub, in settings of our repository just created make the image private.

- Now use the image name in deployment file, change image name to "nginx-image:v1" which is pushed to dockerhub as private.

<img width="718" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/0ed27bd1-1099-401d-8a33-2029b805e79c">

- Now delete previos deployment and try to reapply. We again get ErrImagePull followed by ImagePullBackoff as our repo is private

<img width="780" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/e8a96839-5c10-4c05-8f25-7d98bfd37309">

- Here kubelet run pods when it tries to pull image, kubelet check access to oull from container registry. If it doesn't find access, it throws error.

# Troubleshooting ImagePullBackoff and ErrImagePull

- We can create secret for this using below command
- Command :-  kubectl create secret docker-registry demo --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
- Docker server name will be :- https://index.docker.io/v1/  (If we're using dockerhub)

<img width="779" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/4605d8cb-9452-4ede-a045-a7054f1668ef">

- Secret is of type dockerconfigjson
- Now add secret to yml file and reapply deployment

<img width="529" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/60689bea-2281-428d-b20f-408b0e5e3276">

- Still we get ImagePullBackoff error as we havent configured our dockerhub username in yml

- <img width="784" alt="image" src="https://github.com/Shubham0315/kubernetes-Troubleshooting/assets/105341138/6ca4afd3-f8be-4247-8cdb-2f6f056748dc">
   
- Now if we check and watch pods, we get they're getting created.








