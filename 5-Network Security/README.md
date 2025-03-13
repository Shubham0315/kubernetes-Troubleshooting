# Secure Database using Network Security

- On our K8S cluster, there are 2 namespaces
  - **Secure NS** :- deployed DB here, which has access from a particular application. To secure this DB from other NS which contains different pods. How to secure DB from pods in other NS? As both namespaces on same K8S cluster, we need to secure for security prupose. If pod gets hacked in one NS, if we dont restrict pods from accessing DB in other NS, we need to do so. In our organization, we may've some sensitive pods which shouldnt connect to some other pods, it should have access to pods only which are required to communicate.
    - For this we've "network policies". One is ingress using which we can restrict inward traffic to application/pod. One is egress where we can restrict any traffic going out of pod.
    - Egress :- If we dont want our pod to have access to google.com. We can put list of websites/IPs inside network policies egress config so pods cant talk to them
   

- We'll now create network policy using which pod in one NS cant talk to pod in other NS

- Network policies might now work on local minikube, KIND as we need network plugin enabled and need to have compatible container network interface

#1. Write DB.yml file
-

![image](https://github.com/user-attachments/assets/cb2d0868-bb68-46c9-81b4-9de2fd8472dc)

- First create namespace :- **kubectl create ns secure-namespace**
- Apply DB.yaml with namespace created :- **kubectl apply -f db.yml secure-namespace**
- And check if redis pods (named in db.yml) is running on that namespace

![image](https://github.com/user-attachments/assets/e64ccf11-b912-4b33-8b92-718440d9f1d7)

- Now login to the db pod created :- **kubectl -n secure-namespace exec -it $NAME -- /bin/bash**

![image](https://github.com/user-attachments/assets/10b4bad2-e316-487d-ba1f-e2b170cb7f35)

#2. If there is a pod with default namespace

- Apply hack.yml containing http pod :- **kubectl apply -f hack.yml**       (dont provide NS here, use default)

![image](https://github.com/user-attachments/assets/ffce15bb-db36-446e-8ae3-98bfb8d8cc4b)

- Now login to this default pod :- **kubectl exec -it $NAME -- /bin/bash**

![image](https://github.com/user-attachments/assets/f46bee60-0bc4-4031-ac53-e6192ac46674)

- Once anyone has access to this pod in default namespace, they need to have client installed to connect to DB or install redis CLI to connect (from redis doc)

- Now once logged into redis, run below command :- redis-cli -h #IP. It gets connected to pod, so there is issue of security

![image](https://github.com/user-attachments/assets/be66f73e-4138-4524-b314-075c726909a5)

- Now we can implement network policy to restrict access from IP addresses (pods with certain IPs can talk to our DB), specific DB, restrict pods with labels.
  - Write network policy yml like below. Write network policy to secure redis pod(podSelector), network policy will be applied to pod which have label as app: redis.
  - We can check label for the pod in db.yml as well
  - Psss namespace as well in yml
 
![image](https://github.com/user-attachments/assets/0671ed6f-5633-418e-874d-6292e11a084a)

  - Here we've also defined policy type as ingress
  - If we deploy this network policy :- **kubectl apply -f network-policy.yml -n nsecure-namespace**
  - Now when we try to connect from redis to secure pod, it wont connect.

To block ingress traffic to pod
- Pod selector
- Namespace Selector
- IP Address (CIDR block)
