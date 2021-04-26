# Intro to Kubernetes: Hands-On

Welcome to this introductory Hands-On session for Kubernetes. 

### 0. Starting Minikube

Reference on how to install minikube, as well as potential errors, check the [website](https://minikube.sigs.k8s.io/docs/start/).

``` minikube start``````

> :warning: Remember to start docker desktop if using docker as driver

> :warning: If you have an older installation of Minikube and are getting errors, try:

```minikube delete --all --purge```

```docker system prune```

#### (Optional but convenient) On Mac: 

You can easily create an Alias for kubectl. This way you can enter "k" instead of kubectl every time you want to enter a kubernetes command. 

Completion can also be enabled for this alias, this way Tab can be pressed to show a list of available commands (instead of entering help every time).

##### On Mac:

###### BASH

If you are using BASH, use these steps:

```source <(kubectl completion bash)``` This sets up autocompletion for the current shell

```echo "source <(kubectl completion bash)" >> ~/.bashrc``` This sets up autocomplete permanently - but only takes effect for new bash shells

```alias k=kubectl```

```complete -F __start_kubectl k```

###### ZSH

If you are using ZSH, use these steps:

```source <(kubectl completion zsh)```

```echo "[[ $commands[kubectl] ]] && source <(kubectl completion zsh)" >> ~/.zshrc```

```alias k=kubectl```

```complete -F __start_kubectl k```

#### Convenient Resources

[Official Kubernetes.io CheatSheet Page](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

[A more compact Kubernetes Cheat Sheet pdf](https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf)


### 1. Introduction to kubectl

Kubernetes can be interacted with via the Kubernetes command line interface, Kubectl. Kubectl talks to the Kubernetes API to interact with the cluster. When starting Minikube, kubectl will automatically be configured to connect to the minikube cluster. 

First, we will try some basic "read" commands to show how you can get information from your cluster. See the cheat sheets above for some more commands to try, as well as shortened versions of the commands for real Pros. :cool:

```kubectl get nodes``` shows the nodes that belong to your Kubernetes cluster. Try adding the flag `-o wide` to the command to see how the output changes. 


Namespaces are a logical way to group resources in a cluster into separate spaces. Take a look which Namespaces exist in minikube clusters by default.

```kubectl get namespaces``` 

```kubectl get pods -A``` - the -A flag shows pods in all Namespaces with an extra column to show which namespace the pod belongs to. 

Some other useful commands include checking the logs of a container running in a Pod:

```kubectl logs my-pod-name```

Or, for more involved troubleshooting, opening a terminal session to a container. 
```kubectl exec --stdin --tty my-pod-name -- /bin/sh```


### 2. Run pod

It's time to start our first Pod. 

```kubectl run firstpod --image=oguzpastirmaci/hostname```

```kubectl expose pod firstpod --port=8000 --type=LoadBalancer```


```new terminal```

```minikube tunnel```

```kubectl get service - check localhost and port number```

open localhost:8000

```kubectl get service```

```kubectl describe service firstpod```

```kubectl get pods```

```kubectl describe pod firstpod```


```kubectl delete service firstpod```

```kubectl delete pod firstpod```


### 3. Declarative Deployment

look at hostname-namespace, deployment, and service YAML files 

```kubectl apply -f hostname-namespace.yaml```

```kubectl apply -f hostname-deployment.yaml```

```kubectl apply -f hostname-service.yaml```


```kubectl get all -n hostname```


open localhost:port in output

if not working, restart minikube tunnel


edit deployment .yaml to 2-3 replicas


```kubectl apply -f hostname-deployment.yaml -n hostname```


refresh page (ctrl/cmd + shift + r) to show change between pods


```kubectl delete -f namespace.yaml```

```kubectl delete -f hostname-deployment.yaml -n hostname```

```kubectl delete -f hostname -service -n hostname```


### 4. Stateless vs Stateful 

look at message-board-all-in-one.yaml

```kubectl apply -f message-board-all-in-one.yaml```

```kubectl get all -n message-board```


Open up a new Terminal window ```minikube tunnel``` 

localhost:5000

access website - sign up and write something


```kubectl get pods```

```kubectl delete pod message-board-xxxxx```


reload website - data still there even though pod was deleted


### 5. Network Policy

look at guestbook-all-in-one

```kubectl apply -f guestbook-all-in-one.yaml```

```kubectl get pods```

If not running yet enter `minikube tunnel` in a second window

open up website

localhost:80

write a note into the guestbook


```kubectl apply -f network-policy.yaml```

look at policy - no ingress traffic allowed into redis


reload website - message disappeared because no comms

might take a few moments

```kubectl delete -f network-policy.yaml```


reload website - message appears again

might take a few moments







