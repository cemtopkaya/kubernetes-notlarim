# Using PersistentVolumes in Kubernetes

## Introduction

PersistentVolumes provide a way to treat storage as a dynamic resource in Kubernetes. This lab will allow you to demonstrate your knowledge of PersistentVolumes. You will mount some persistent storage to a container using a PersistentVolume and a PersistentVolumeClaim.

### Create a PersistentVolume That Allows Claim Expansion

1.  Create a custom Storage Class by using `vi localdisk.yml`.
2.  Define the Storage Class by using:

    ```
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: localdisk
    provisioner: kubernetes.io/no-provisioner
    allowVolumeExpansion: true
    ```

3.  Save and exit the file by hitting the ESC key and using `:wq`.
4.  Finish creating the Storage Class by using `kubectl create -f localdisk.yml`.
5.  Create the PersistentVolume by using `vi host-pv.yml`.
6.  Define the PersistentVolume with a size of 1Gi by using:

    ```
    kind: PersistentVolume
    apiVersion: v1
    metadata:
       name: host-pv
    spec:
       storageClassName: localdisk
       persistentVolumeReclaimPolicy: Recycle
       capacity:
          storage: 1Gi
       accessModes:
          - ReadWriteOnce
       hostPath:
          path: /var/output
    ```

7.  Save and exit the file by hitting the ESC key and using `:wq`.
8.  Finish creating the PersistentVolume by using `kubectl create -f host-pv.yml`.
9.  Check the status of the PersistenVolume by using `kubectl get pv`.

### Create a PersistentVolumeClaim

1.  Start creating a PersistentVolumeClaim for the PersistentVolume to bind to by using `vi host-pvc.yml`.
2.  Define the PersistentVolumeClaim with a size of 100Mi by using:

    ```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
       name: host-pvc
    spec:
       storageClassName: localdisk
       accessModes:
          - ReadWriteOnce
       resources:
          requests:
             storage: 100Mi
    ```

3.  Save and exit the file by hitting the ESC key and using `:wq`.
4.  Finish creating the PersistentVolumeClaim by using `kubectl create -f host-pvc.yml`.
5.  Check the status of the PersistentVolume and PersistentVolumeClaim to verify that they have been bound:

    ```
    kubectl get pv
    kubectl get pvc
    ```

### Create a Pod That Uses a PersistentVolume for Storage

1.  Create a Pod that uses the PersistentVolumeClaim by using `vi pv-pod.yml`.
2.  Define the Pod by using:

    ```
    apiVersion: v1
    kind: Pod
    metadata:
       name: pv-pod
    spec:
       containers:
          - name: busybox
            image: busybox
            command: ['sh', '-c', 'while true; do echo Success! > /output/success.txt; sleep 5; done']
    ```

3.  Mount the PersistentVolume to the /output location by adding the following, which should be level with the `containers` spec in terms of indentation:

    ```
    volumes:
      - name: pv-storage
        persistentVolumeClaim:
           claimName: host-pvc
    ```

4.  In the `containers` spec, below the `command`, set the list of volume mounts by using:

    ```
    volumeMounts:
      - name: pv-storage
        mountPath: /output
    ```

5.  Save and exit the file by hitting the ESC key and using `:wq`.
6.  Finish creating the Pod by using `kubectl create -f pv-pod.yml`.
7.  Check that the Pod is up and running by using `kubectl get pods`.
8.  If you wish, you can log in to the worker node and verify the output data by using `cat /var/output/success.txt`.
