# Intro to Kubernetes: Hands-On

Welcome to this introductory Hands-On session for Kubernetes. 

### Table of Contents

[Starting Minikube](https://github.com/cedricfeist/kubernetes-hands-on#0-starting-minikube)

[Introduction to kubectl](https://github.com/cedricfeist/kubernetes-hands-on#1-introduction-to-kubectl)

[Starting a Pod - Imperative](https://github.com/cedricfeist/kubernetes-hands-on#2-starting-a-pod---imperative)

[Declarative Deployments](https://github.com/cedricfeist/kubernetes-hands-on#3-declarative-deployments)

[Stateless vs Stateful Apps](https://github.com/cedricfeist/kubernetes-hands-on#4-stateless-vs-stateful-apps)

[Network Policy](https://github.com/cedricfeist/kubernetes-hands-on#5-network-policy)


First things first:

Start by cloning the repo into your favourite directory.

```
git clone https://github.com/cedricfeist/kubernetes-hands-on

cd kubernetes-hands-on
```

### 0. Starting Minikube
------
Reference on how to install minikube, as well as potential errors, check the [kubernetes website](https://minikube.sigs.k8s.io/docs/start/).

Start minikube, remember to set the cni flag to calico as we will be using network policies at the end of the lab. 

```
minikube start --network-plugin=cni --cni=calico 
# Wait for all system pods to be in the `Running 1/1` state

kubectl get pods -n kube-system
```

> :warning: Remember to start docker desktop if using docker as driver

> :warning: If you had an old installation of Minikube which you updated and are getting errors, try:

```minikube delete --all --purge```

```docker system prune```

#### (Optional but convenient) On Mac: 

You can easily create an Alias for kubectl. This way you can enter "k" instead of kubectl every time you want to enter a kubernetes command. 

Completion can also be enabled for this alias, this way Tab can be pressed to show a list of available commands (instead of entering help every time).

##### On Mac:

###### BASH

If you are using BASH, use these steps:

```
source <(kubectl completion bash #This sets up autocompletion for the current shell

echo "source <(kubectl completion bash)" >> ~/.bashrc
``` 

This sets up autocomplete permanently - but only takes effect for new terminal sessions. 


```
alias k=kubectl

complete -F __start_kubectl k
```

###### ZSH

If you are using ZSH, use these steps:

```
source <(kubectl completion zsh

echo "[[ $commands[kubectl] ]] && source <(kubectl completion zsh)" >> ~/.zshrc

alias k=kubectl

complete -F __start_kubectl k
```

#### Convenient Resources

[Official Kubernetes.io CheatSheet Page](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

[A more compact Kubernetes Cheat Sheet pdf](https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf)


### 1. Introduction to kubectl
------
Kubernetes can be interacted with via the Kubernetes command line interface, Kubectl. Kubectl talks to the Kubernetes API to interact with the cluster. When starting Minikube, kubectl will automatically be configured to connect to the minikube cluster. 

First, we will try some basic "read" commands to show how you can get information from your cluster. See the cheat sheets above for some more commands to try, as well as shortened versions of the commands for real Pros. :sunglasses:

```kubectl get nodes``` shows the nodes that belong to your Kubernetes cluster. Try adding the flag `-o wide` to the command to see how the output changes. 


Namespaces are a logical way to group resources in a cluster into separate spaces. Take a look which Namespaces exist in minikube clusters by default.

```kubectl get namespaces``` 


```kubectl get pods -A``` - the -A flag shows pods in all Namespaces with an extra column to show which namespace the pod belongs to. 

Some other useful commands include checking the logs of a container running in a Pod:

```kubectl logs my-pod-name```

Or, for more involved troubleshooting, opening a terminal session to a container. 
```kubectl exec --stdin --tty my-pod-name -- /bin/sh```


### 2. Starting a Pod - Imperative
------
It's time to start our first Pod. 

```
kubectl run firstpod --image=oguzpastirmaci/hostname
```

To make this pod accessible from outside the cluster, we need to create a service.

```
kubectl expose pod firstpod --port=8000 --type=LoadBalancer
```

In a production environment, we would have an IP Management system that associates available IPs with the LoadBalancer Service we just created. 

Minikube has built-in functionality to allow us to access this traffic from our local machine. 

> open a new terminal/cmd instance

```
minikube tunnel
```

You might have to wait a few seconds before the tunnel is established

```kubectl get service``` :warning: Note that a port number and the IP of our Localhost are displayed here

> Open [localhost:8000](http://localhost:8000) on the browser of your choice.

This simple app displays the hostname of the container we are connected to. Since we only have one right now, this won't change when refreshing the page. 


We can now observe the objects we just created and look into the specification using the `describe` keyword.

```
kubectl get pods

kubectl describe pod firstpod

kubectl get services

kubectl describe service firstpod
```

Because we manually created this app, we will delete it for now as there are better ways to deploy on Kubernetes. 

```
kubectl delete service firstpod

kubectl delete pod firstpod

:warning: On some versions of kubectl the "run" command may create a deployment.
:warning: In this case, deleting the pod will cause Kubernetes to restart a new one.  
:warning: To delete the deployment, enter `kubectl delete deployment firstpod`
```


### 3. Declarative Deployments
------
Before entering the next commands, take a look at the *hostname-namespace*, *hostname-deployment*, and *hostname-service* YAML files in the cloned repository. 

Once you have taken note of the configuration, apply the configuration stored in these files via the following commands.

```
kubectl apply -f hostname-namespace.yaml

kubectl apply -f hostname-deployment.yaml

kubectl apply -f hostname-service.yaml
```

We just created a number of resources. We could look at these individually using the commands above, or - because we grouped them in a namespace - we can look at all the resources in the namespace using the following command. 

```
kubectl get all -n hostname
```

From this output, you should be able to see which port the app is running on. 

> Open localhost on the port you see in the service we just created

:warning: Your `minikube tunnel` session should still be open for this. Start a new one if it is closed, or if it is not working within a few seconds. 

You might have seen that the deployment only started 1 pod. This is not much different to our last app, so let's edit the hostname-deployment.yaml file to have 3 replicas. 

> Change the number next to `replicas: ` to 3, save it, and update the configuration on your cluster using `kubectl apply -f hostname-deployment.yaml`

You should see an output: `deployment.apps/hostname configured`

> Refresh the page on your browser (ctrl/cmd + shift + r) to show change between pods. Every now and then the hostname should change, to show that the LoadBalancer is directing traffic to a new Pod. 

> :warning: Caching mechanisms might prevent the page from refreshing properly to see the hostname change. 
> Try `curl http://localhost:port` in your Terminal to see the LadBalancing mechanism switch between the pods. 

You can leave these pods running, but if you want to delete them here are the steps to do so. 

```
kubectl delete -f hostname-namespace.yaml
```

> :warning: This operation may take a few minutes. 

Deleting a namespace will delete all resources within it. 

### 4. Stateless vs Stateful Apps
------
Before we find out how Persistent Volumes and Persistent Volume Claims can help us with designing Apps, look at message-board-all-in-one.yaml. Instead of keeping each resource in a separate file, we have combined them all in one YAML file which contains the entire configuration of the application. 

Once you have taken a look, apply the configuration...

```
kubectl apply -f message-board-all-in-one.yaml
```

...and see all the resources created. 

```
kubectl get all -n message-board
```

Some of the pods might take a while to be fully available. Instead of running the same command over and over to monitor the readiness of a resource, we can use the `-w` flag to watch the resource for changes. 

```
kubectl get pods -n message-board -w
```

Wait for The Pod to be Running and Ready. 

:warning: Once the pods are running, press `ctrl + c` to go back. 

> If not open from previous exercises: open up a new Terminal window using `minikube tunnel` 

> Open [localhost:5000](http://localhost:5000), sign up to the message board, log in, and write a message. 

We now have a message displayed in our app. What happens when we delete the pod that is hosting the website?

The following command will delete our pod running in the message-board namespace.

```
kubectl delete pod -l name=message-board -n message-board

kubectl get pods
```

> Wait for the new container to be started. Note: Kubernetes noticed the missing container, and automatically started another one to reach the desired state. 

> Reload the Page 

Notice that the message is still there. This is because our message was saved in our Persistent Volume, which gets attached to our container. The containers themselves are stateless and do not store any data. There are a few use-cases for storing data in a container (short-lived caches for example) but generally it is best practice to store persistent data in a Persistent Volume. 

### 5. Network Policy
------
The last thing we will look at in this session is Network Policies. 

Before applying the configuration file, look at *guestbook-all-in-one.yaml* file

```
kubectl apply -f guestbook-all-in-one.yaml

kubectl get pods -w
```

> Wait for all the Pods to be Running and Ready

:warning: Once the pods are Ready you can press `ctrl + c` to go back. 

> If not running yet enter `minikube tunnel` in a second window

Run `kubectl get service` to verify the port:

> Open [localhost:80](http://localhost:80) on your browser

:warning: Port 80 will require root privileges, so check your `minikube start` window to enter your password, if the page doesn't open. 

> Write a note into the guestbook

More detailed information on Kubernetes Network Policies can be found [here](https://kubernetes.io/docs/concepts/services-networking/network-policies/) but for now we will create a simple one. 

> Take a look at the network policy in *network-policy.yaml*. All it does is prevent ingress traffic to the redis services to stop the frontend from retrieving messages.

```
kubectl apply -f network-policy.yaml
```

> Reload the webpage. Within a few moments the new policy should be enforced and the message disappear

> :warning: This can take a few seconds. 

```
kubectl delete -f network-policy.yaml
```

> Reload the webpage again

It might take a few more moments as before, but eventually the message will reappear as the policy allows traffic to the redis pods once again. 


### Stopping Minikube
------
Feel free to keep the cluster running and experimenting with Kubernetes. 

Once you are done, you can stop the cluster:

```
minikube stop
```

This stops the minikube cluster, but retains the state for when you start the cluster again via `minikube start`.

If you ever break your cluster, or want to start fresh you can delete the cluster with `minikube delete`. If you have multiple clusters this changes to `minikube delete --all`. 






