# Kubernetes Hizmetlerini DNS ile Kullanma

## Introduction

Kubernetes Services can be located with the Kubernetes DNS just like Pods can. In this lab, you will work with the Kubernetes DNS to discover Services from within a Pod. This will test your knowledge of how to interact with Services using DNS.

<img width="722" alt="image" src="https://github.com/user-attachments/assets/d35bf63e-d814-465c-8665-26d51290d086">

## Çözüm

<img width="633" alt="image" src="https://github.com/user-attachments/assets/f0e2b9fc-facc-4dad-9b20-7b54ff8b3308">

### Deployment ile Front-End Ayaklandıralım

Namespace ile tüm yapımızı taşıyacak isim alanımızı tanımlayalım `ns-web.yaml`
```yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2024-08-08T23:04:52Z"
  labels:
    kubernetes.io/metadata.name: web
  name: web
  resourceVersion: "571"
  uid: cfcddac3-04d5-49a6-a0f8-620c1aec2576
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```

Deployment oluşturulacak `deployment-web-frontend.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"web-frontend","namespace":"web"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"web-frontend"}},"template":{"metadata":{"labels":{"app":"web-frontend"}},"spec":{"containers":[{"image":"nginx:1.19.1","name":"nginx","ports":[{"containerPort":80}]}]}}}}
  creationTimestamp: "2024-08-08T23:05:02Z"
  generation: 1
  name: web-frontend
  namespace: web
  resourceVersion: "733"
  uid: 15c65a4e-3c8c-4f6f-9762-37ed13f81d36
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: web-frontend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web-frontend
    spec:
      containers:
      - image: nginx:1.19.1
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2024-08-08T23:05:11Z"
    lastUpdateTime: "2024-08-08T23:05:11Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2024-08-08T23:05:02Z"
    lastUpdateTime: "2024-08-08T23:05:11Z"
    message: ReplicaSet "web-frontend-74b976bfb5" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
```

Servisi `svc-web-frontend.yaml` oluşturulacak:
```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"web-frontend","namespace":"web"},"spec":{"ports":[{"port":80,"protocol":"TCP","targetPort":80}],"selector":{"app":"web-frontend"}}}
  creationTimestamp: "2024-08-08T23:05:03Z"
  name: web-frontend
  namespace: web
  resourceVersion: "657"
  uid: be2f9e57-1e23-45ec-82f7-20726cb3b437
spec:
  clusterIP: 10.109.217.9
  clusterIPs:
  - 10.109.217.9
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: web-frontend
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

Replicaset dosyamız `replica-web-frontend.yaml` 
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  annotations:
    deployment.kubernetes.io/desired-replicas: "1"
    deployment.kubernetes.io/max-replicas: "2"
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2024-08-08T23:05:02Z"
  generation: 1
  labels:
    app: web-frontend
    pod-template-hash: 74b976bfb5
  name: web-frontend-74b976bfb5
  namespace: web
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: Deployment
    name: web-frontend
    uid: 15c65a4e-3c8c-4f6f-9762-37ed13f81d36
  resourceVersion: "729"
  uid: 84120aa2-7275-4b53-a13e-620d8fe9072e
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-frontend
      pod-template-hash: 74b976bfb5
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web-frontend
        pod-template-hash: 74b976bfb5
    spec:
      containers:
      - image: nginx:1.19.1
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  fullyLabeledReplicas: 1
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
```

`` İsim alanının bileşenleri: 


`ns-data.yaml` ile user-data isim alanı:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2024-08-08T23:04:52Z"
  labels:
    kubernetes.io/metadata.name: data
  name: data
  resourceVersion: "575"
  uid: 662ae35a-b241-4ae2-a289-b36d00285b2b
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```

`svc-user-data.yaml` İle user-data'nın servis kısmı:
```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"user-db","namespace":"data"},"spec":{"ports":[{"port":80,"protocol":"TCP","targetPort":80}],"selector":{"app":"user-db"}}}
  creationTimestamp: "2024-08-08T23:04:58Z"
  name: user-db
  namespace: data
  resourceVersion: "627"
  uid: dfdeed70-6021-4de6-a19d-fff4275f34b4
spec:
  clusterIP: 10.109.170.132
  clusterIPs:
  - 10.109.170.132
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: user-db
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

Replicaset'in dosyası `replica-user-data.yaml`:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  annotations:
    deployment.kubernetes.io/desired-replicas: "1"
    deployment.kubernetes.io/max-replicas: "2"
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2024-08-08T23:04:56Z"
  generation: 1
  labels:
    app: user-db
    pod-template-hash: 7bd9cb55d4
  name: user-db-7bd9cb55d4
  namespace: data
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: Deployment
    name: user-db
    uid: d594c4c3-8311-4464-a33c-7d0f5bf8f552
  resourceVersion: "764"
  uid: c6f72b8f-5acc-4e64-8cca-7ff5a2872b61
spec:
  replicas: 1
  selector:
    matchLabels:
      app: user-db
      pod-template-hash: 7bd9cb55d4
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: user-db
        pod-template-hash: 7bd9cb55d4
    spec:
      containers:
      - image: nginx:1.19.1
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  fullyLabeledReplicas: 1
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
```

Deploymentset'in dosyası `deployment-user-data.yaml` :
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"user-db","namespace":"data"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"user-db"}},"template":{"metadata":{"labels":{"app":"user-db"}},"spec":{"containers":[{"image":"nginx:1.19.1","name":"nginx","ports":[{"containerPort":80}]}]}}}}
  creationTimestamp: "2024-08-08T23:04:55Z"
  generation: 1
  name: user-db
  namespace: data
  resourceVersion: "767"
  uid: d594c4c3-8311-4464-a33c-7d0f5bf8f552
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: user-db
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: user-db
    spec:
      containers:
      - image: nginx:1.19.1
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2024-08-08T23:05:15Z"
    lastUpdateTime: "2024-08-08T23:05:15Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2024-08-08T23:04:56Z"
    lastUpdateTime: "2024-08-08T23:05:15Z"
    message: ReplicaSet "user-db-7bd9cb55d4" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
```

### Perform an Nslookup for a Service in the Same Namespace

1.  Start using the `busybox` Pod in the `web` namespace to perform an nslookup on the `web-frontend` Service by entering:

    ```
    kubectl exec -n web busybox -- nslookup web-frontend
    ```

    <img width="1053" alt="image" src="https://github.com/user-attachments/assets/627da5a0-d110-47ac-a29f-94f093f578dc">


3.  Redirect the output to save the results in a text file by using:

    ```
    kubectl exec -n web busybox -- nslookup web-frontend >> ~/dns_same_namespace_results.txt
    ```

    <img width="733" alt="image" src="https://github.com/user-attachments/assets/100c9ba9-6516-43a5-a567-513e12df8e48">


5.  Look up the same Service using the fully qualified domain name by entering:

    ```
    kubectl exec -n web busybox -- nslookup web-frontend.web.svc.cluster.local
    ```

    <img width="620" alt="image" src="https://github.com/user-attachments/assets/ecccc8aa-c631-4a75-bcdd-d7dc49d71a52">


7.  Redirect the output to save the results of the second nslookup in a text file by using:

    ```
    kubectl exec -n web busybox -- nslookup web-frontend.web.svc.cluster.local >> ~/dns_same_namespace_results.txt
    ```

8.  Check that everything looks okay in the text file by using:

    ```
    cat ~/dns_same_namespace_results.txt
    ```

9.  Use `clear` to clear the text file output.

### Perform an Nslookup For a Service in a Different Namespace

1.  Use the `busybox` Pod in the `web` namespace to perform an nslookup on the `user-db` Service in the `data` namespace, while only utilizing the short Service name, by entering:

    ```
    kubectl exec -n web busybox -- nslookup user-db
    ```

    Kendi isim alanından hem kısa hem uzun adıyla bir servisi sorgularsak sorun çıkarmadan cevabı alırız:

    <img width="722" alt="image" src="https://github.com/user-attachments/assets/05ded9aa-b6df-4b7b-a290-0c0007f81436">


This first request is supposed to result in an error message, so don't be alarmed if you see that we can't resolve `user-db`.

<img width="617" alt="image" src="https://github.com/user-attachments/assets/06895776-4726-4bb3-b9fd-976494aff0f4">


1.  Save the results of this nslookup in a text file by using:

    `web` isim alanındaki bir konteyner içinde `data` isim alanındaki bir servise `nslookup` ile **servisin kısa adıyla** bakmak istediğimizde hata alırız ancak FQDN ile sorunsuz cevap alırız:

    <img width="653" alt="image" src="https://github.com/user-attachments/assets/b7870a05-174f-47a1-bf94-76cf72d4acaa">

    
    ```
    kubectl exec -n web busybox -- nslookup user-db >> ~/dns_different_namespace_results.txt
    ```

3.  Perform the same lookup using the fully qualified domain name by entering:

    ```
    kubectl exec -n web busybox -- nslookup user-db.data.svc.cluster.local
    ```

4.  Save the results in a text file by using:

    ```
    kubectl exec -n web busybox -- nslookup user-db.data.svc.cluster.local >> ~/dns_different_namespace_results.txt
    ```

5.  Check the output in the text file by using:

    ```
    cat ~/dns_different_namespace_results.txt
    ```
