+++
aliases = ["/blog/2020/08/01/k8s-api-auth-client-go/"]
author = "Ghotfall"
categories = ["Programming", "DevOps", "Golang"]
date = 2020-08-01T00:00:00Z
description = "You would learn how to authenticate to the Kubernetes API from an Golang application running inside or outside the Kubernetes cluster"
images = ["https://miro.medium.com/max/1190/1*NFv453pxVi1goLh-yVa4pQ.png"]
tags = ["go", "golang", "k8s", "kubernetes", "client-go"]
title = "Authenticating to the Kubernetes API with client-go"

+++
Client-go, go library for talking to a kubernetes cluster, provides few ways for authenticating to the Kubernetes API. Possible steps are quite different for apps which are running inside and outside the cluster.

Your actions would be quite simple if you planning to pack your golang app in container and use it within pod. Let's look at the example:

```go
config, err := rest.InClusterConfig()
if err != nil {
	panic(err.Error()) // oops, something went wrong, you can't make calls to k8s API :(
}
```

But keep in mind that this approach uses the **Service Account token**.

Service accounts are usually associated with pods running in the cluster through the ServiceAccount Admission Controller. Bearer tokens are mounted into pods at well-known locations, and allow in-cluster processes to talk to the API server.

So, client-go would try to find mounted token at the **/var/run/secrets/kubernetes.io/serviceaccount** path. In addition, you will need to tweak your Service Account, if you want to perform some opperations within pod and have **RBAC enabled**. So prepare RBAC Role or ClusterRole and binding for it before deploying your app to the cluster.

For working outside of the cluster you can try whis way:

```go
kubeconfig := clientcmd.NewNonInteractiveDeferredLoadingClientConfig(
    clientcmd.NewDefaultClientConfigLoadingRules(),
    &clientcmd.ConfigOverrides{},
)

config, err := kubeconfig.ClientConfig()
if err != nil {
	panic(err.Error()) // oops, something went wrong, you can't make calls to k8s API :(
}
```

This will work if you have kubeconfig file already in place.

Do you want more flexible approach? Try this:

```go
var configPath = ""

if value, ok := os.LookupEnv("KUBECONFIG"); ok {
    configPath = value
} else if _, err :=os.Stat("~/.kube/config"); err == nil {
	configPath = "~/.kube/config"
}

config, err := clientcmd.BuildConfigFromFlags("", configPath)
if err != nil {
	panic(err.Error()) // oops, something went wrong, you can't make calls to k8s API :(
}
```

This code snippet will firstly try to get path to kubeconfig from environment variable. If it fails, it will try to find file by default path. This kubeconfig will be used for talking to your cluster.

But you can see that I didn't add another else branch with panic() after file searching, so configPath variable will be empty if kubeconfig is missing. Under the hood **clientcmd.BuildConfigFromFlags** function will try to use "in-cluster" method, if you pass empty string to it as arguments.

So you can treat this variant as hybrid method.

After obtaining config you can finnaly make calls to the Kubernetes API:

```go
clientSet, err := kubernetes.NewForConfig(config)
if err != nil {
	panic(err.Error()) // ugh, something wrong again >_<
}

// Simple example from client-go repository: get info about all pods
// in cluster with one simple list request
pods, err := clientSet.CoreV1().Pods("").List(context.TODO(), metav1.ListOptions{})
if err != nil {
	panic(err.Error()) // your request failed 
}
fmt.Printf("There are %d pods in the cluster\n", len(pods.Items))
```

I hope the article was useful to you. Come here more often.