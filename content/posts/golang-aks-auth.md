---
categories: ["howto", "kubernetes", "azure", "golang"]
title: "AKS Authentication with Golang"
date: 2023-04-25T12:32:30+01:00

---

# Overview
Recently I've needed to interact with Azure Kubernetes Service (AKS) using golang. This isn't hard,
but what complicates matters somewhat is when the cluster is AAD enabled.

In this case the general advice is to use [kubelogin](https://github.com/Azure/kubelogin). In the
past I've shelled out to `kubelogin` from my golang code, and this works fine. However, we should be
able to do this in native golang without the dependencies on external tools.

The requirements I came up with are as follows:

* Allow use of various auth methods, in particular azcli and system assigned managed identity.
* Doesn't require a kubeconfig to be present on the host.
* Doesn't require persisting the kubeconfig to the host.
* Uses native and non deprecated sdks.

# Even too hard for ChatGPT?!
Surprisingly, I wasn't able to find any good stock examples of this online. I assume they have to be
out there, but either my Google-fu is coming up short or I'm missing something obvious! Even ChatGPT
was spewing non working examples and insisted on using the deprecated
[go-autorest](https://github.com/Azure/go-autorest). After splunking through the docs I finally
managed to get a working example, which I'm posting here for future use.

# Process
From a high level, the process to obtain an authorized k8s client is as follows:

* Create an authenticated Azure client
* Fetch a client for the container service
* Fetch the kubeconfig
* Fetch an AAD bearer token
* Replace the exec plugin details with the token in the kubeconfig
* Finally, create a k8s client using the kubeconfig

The full code for this can be found
[here](https://gist.github.com/brk3/0f263edeb0eb842a7d1ace45fb50b150).

