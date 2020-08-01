+++
author = "Ghotfall"
title = "Authenticating to the Kubernetes API with client-go"
date = "2020-07-31"
description = "You would learn how to authenticate to the Kubernetes API from an Golang application running inside or outside the Kubernetes cluster"
tags = ["go", "golang", "k8s", "kubernetes", "client-go"]
categories = ["Programming", "DevOps", "Golang"]
images = [""]
+++

Client-go, go library for talking to a kubernetes cluster, provides few ways for authenticating to the Kubernetes API. Possible steps are quite different for apps which are running inside and outside the cluster.

Your actions would be quite simple if you planning to pack your golang app in container and use it within pod. Let's look at the example:

```go
config, err := rest.InClusterConfig()
if err != nil {
	panic(err.Error()) // oops, something went wrong, you can't make calls to k8s API :(
}
```

{{< gist spf13 7896402 >}}