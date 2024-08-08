# Exploring Kubernetes Networking

## Introduction

Kubernetes ağı derin ve çeşitli bir konudur. Bu laboratuvarda bir ağ çözümü uygulayarak Kubernetes ağ iletişimi bilginizi test edeceksiniz. Ayrıca iki pod'un sanal konteyner ağınız aracılığıyla birbirleriyle iletişim kurabildiğini de doğrulayacaksınız.

<img width="695" alt="image" src="https://github.com/user-attachments/assets/f979f8f5-148a-4cdc-9c13-ec23563d0892">

## Çözüm

### Podların Başlatılmamasına Neden Olan Sorunu Düzeltme

1.  List the pods to check their status:

    ```
    kubectl get pods
    ```

2.  Check the node status:

    ```
    kubectl get nodes
    ```

    It looks like the nodes are `NotReady`.
    
    <img width="462" alt="image" src="https://github.com/user-attachments/assets/cfa7ca2a-b59c-4670-a2a1-ffd24e44c413">


4.  Describe a node to see if you can get more info:

    ```
    kubectl describe node k8s-worker1
    ```

    It looks like `kube-proxy`, a component that handles networking-related tasks, is stuck starting up.

    <img width="1440" alt="image" src="https://github.com/user-attachments/assets/b113a1e9-631d-468a-95e7-ff9d17f4461c">

6.  Check the status of the networking plugin pods:

    ```
    kubectl get pods -n kube-system
    ```

    <img width="587" alt="image" src="https://github.com/user-attachments/assets/de4789f6-562a-4acc-a240-0391c9def1e5">

    Buraya bakınca core-dns'in çalışmadığını görüyoruz ama birincil sorun ağ yapılandırmasının bu kubernetes kümesinde yapılandırılmadığıdır.

8.  Install the Calico networking plugin:

    Bunu calico ağını k8s kümesine uygulayarak çözebiliriz:
    
    ```
    kubectl apply -f https://docs.projectcalico.org/v3.15/manifests/calico.yaml
    ```

10.  Check the status of the Nodes and Pods again:

    ```
    kubectl get nodes
    ```

    They should both be `Ready` after about a minute.

    ```
    kubectl get pods
    ```

    <img width="852" alt="image" src="https://github.com/user-attachments/assets/304e7fdb-570d-427e-ad93-8a335299acd7">
    
    <img width="451" alt="image" src="https://github.com/user-attachments/assets/2507ef7b-b9b7-4648-8358-151d6bfba9e0">

### Verify You Can Communicate between Pods Using the Cluster Network

1.  Verify the two pods can communicate over the network:

    ```
    kubectl get pods -o wide
    ```

2.  Run `curl` on the IP address of the `cyberdyne-frontend` Pod (which will be listed in the output from the previous command):

    ```
    kubectl exec testclient -- curl <cyberdyne-frontend_POD_IP>
    ```

    The result should be HTML of an Nginx page, meaning the Pods are able to communicate.

    <img width="976" alt="image" src="https://github.com/user-attachments/assets/259f9735-4547-4509-ae26-3b965ed971a1">
