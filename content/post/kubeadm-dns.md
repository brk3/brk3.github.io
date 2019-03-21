+++
categories = ["notes", "kubernetes"]
date = "2019-03-21T12:01:43Z"
title = "How to replace coredns with kube-dns"

+++


CoreDNS is the default DNS addon in Kubernetes since 1.13. I recently needed to switch back to
kube-dns for some testing, and it wasn't obvious from the documentation how to do that. Here's how I
went about it for reference.

These steps assume kubeadm 1.13.

Steps
-----
First, initialise the master node, skipping the coredns addon phase:

```
kubeadm init --pod-network-cidr=10.244.0.0/16 --skip-phases=addon/coredns
```

Then prepare a kubeadm override config to specify kube-dns is to be used instead of the default:

```
$ cat > kubeadm-kube-dns.yaml << EOF
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
dns:
    type: "kube-dns"
    imageRepository: myregistry:5000
EOF
```

This example also shows how to use a custom repository for the images.

Now init the addon:

```
kubeadm init phase addon coredns --config=kubeadm-kube-dns.yaml
```

Finally, add your CNI addon of choice to finish setting up the master:

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

That's it. I also found the following steps from [IBM][0] useful to verify the DNS is sane (remember to
add a k8s worker before proceeding with this):

```
kubectl create -f https://k8s.io/examples/admin/dns/busybox.yaml
kubectl exec -ti busybox -- nslookup kubernetes.default
```

[0]: https://www.ibm.com/support/knoawledgecenter/en/SSYGQH_6.0.0/admin/install/cp_prereq_kubernetes_dns.html
