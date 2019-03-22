I am trying using Multus with Cilium and openshift-sdn. This repo includes collection of scripts/hacks around Openshift
I did to get a working deployment. Files in this repo are customised manifests for k8s resources.
 
# THIS IS A WORK IN PROGRESS.
 
Environment I used:  
OS: Centos 7  
Kernel > 4.2 needed by Cilium  
 
I started with a default openshift all-in-one cluster installed using openshift-ansible.
 
Next step is to add multus to the installation. I used playbook from dougbtv detailed at  
https://github.com/dougbtv/openshift-ansible/tree/multus-developer-preview/playbooks/openshift-multinetwork

Once this is done, edit /etc/cni/net.d/70-multus.conf to match with the file in this repo.  
And then kubectl apply -f multus.yaml from this repo.  
 
There should be 3 pods running in 'openshift-sdn'project  
$ kubectl get pods --all-namespaces  
openshift-sdn                       kube-multus-ds-amd64-tv2r5                     1/1       Running             0          1d  
openshift-sdn                       ovs-8mlkz                                      1/1       Running             1          2d  
openshift-sdn                       sdn-zjhnj                                      1/1       Running             1          2d  

 
Once multus is installed, install Cilium using 'cilium.yaml' file from this repo  
$ kubectl apply -f cilium.yaml  
 
There should be Cilium pods running now.  
$ kubectl get pods --all-namespaces  
kube-system                         cilium-etcd-4p5znd7gxn                         1/1       Running             0          1d  
kube-system                         cilium-etcd-m2w2nj6s46                         1/1       Running             0          1d  
kube-system                         cilium-etcd-operator-585dbcbb47-w9z6f          1/1       Running             0          1d  
kube-system                         cilium-etcd-sftjgjrgwh                         1/1       Running             0          1d  
kube-system                         cilium-operator-7d4bff7fbf-pxp5x               1/1       Running             1          1d  
kube-system                         cilium-rhbk2                                   1/1       Running             2          1d  
kube-system                         coredns-55f86bf584-6z875                       1/1       Running             4          1d  
kube-system                         coredns-55f86bf584-l9lbw                       1/1       Running             4          1d  

If you don't have coredns pods, start them by running  
$ kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml  

Create a network attachement definition for Cilium as defined in cilium_nad.yaml  
$ kubectl apply -f cilium_nad.yaml  

By this time, you should have a setup where openshift-sdn is a primary CNI and CIlium is secondary CNI achieved using Multus.  

Lets try to spawn a pod with 2 itnerfaces now.  
$ kubectl apply -f pod.yaml  

For me, the pod is not spawned. Its stuck in containercreate state.  
default                             samplepod                                      0/1       ContainerCreating   0          13m  

journald logs show error as detailed in https://paste.fedoraproject.org/paste/zQQsjYOXjauuoH0bcmHCEg
