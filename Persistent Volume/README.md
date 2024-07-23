# Using PersistentVolumes in Kubernetes

## Introduction

PersistentVolumes provide a way to treat storage as a dynamic resource in Kubernetes. You will mount some persistent storage to a container using a PersistentVolume and a PersistentVolumeClaim.

### Kısa Yoldan: Tümünü Kustomization İle Çalıştırmak

`kustomization.yaml` Dosyası içinde istenilen sıralarda hangi manifesto dosyalarının çalıştırılacağını [kustomization](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/) ile  istenilen sırada yaratmayı sağlar.

```sh
kubectl apply -k .
```

Aynı şekilde tüm kaynakları silmek için:
```sh
kubectl delete -k .

# veya

kubectl delete --kustomize .
```

![image](https://github.com/user-attachments/assets/81c5b0b0-e2a7-46ad-9437-0c8d802cf887)

Tüm dosyaları görüntülemek için `kubectl kustomize` komutunu çalıştıralım:

![image](https://github.com/user-attachments/assets/274f00f0-2102-4628-9705-1a9972f9af16)


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

    ![image](https://github.com/user-attachments/assets/bedfb313-e6b9-430e-8a42-ab66c1216e88)

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

    ![image](https://github.com/user-attachments/assets/7e4a946c-9b9c-4cd8-8d78-2b3156e10f8d)

7.  Save and exit the file by hitting the ESC key and using `:wq`.
8.  Finish creating the PersistentVolume by using `kubectl create -f host-pv.yml`.
9.  Check the status of the PersistenVolume by using `kubectl get pv`.

![image](https://github.com/user-attachments/assets/dc2854e9-97ff-4cb4-921d-ceae11beb41a)

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

    ![image](https://github.com/user-attachments/assets/7b531bfd-b56b-479e-8dcb-944ba535017e)

3.  Save and exit the file by hitting the ESC key and using `:wq`.
4.  Finish creating the PersistentVolumeClaim by using `kubectl create -f host-pvc.yml`.
5.  Check the status of the PersistentVolume and PersistentVolumeClaim to verify that they have been bound:

    ```
    kubectl get pv
    kubectl get pvc
    ```

    ![image](https://github.com/user-attachments/assets/d1969185-1300-4fa0-b613-c4472f6ed9d2)

### Create a Pod That Uses a PersistentVolume for Storage

1.  Create a Pod that uses the PersistentVolumeClaim by using `vi pv-pod.yml`.
2.  Pod manifest file:


```
apiVersion: v1
kind: Pod
metadata:
   name: pv-pod
spec:
   volumes:
   - name: pv-storage
     persistentVolumeClaim:
       claimName: host-pvc

   containers:
   - name: busybox
     image: busybox
     command: ['sh', '-c', 'while true; do echo Success! > /output/success.txt; sleep 5; done']
     volumeMounts:
     - name: pv-storage
       mountPath: /output
```

3.  Define the Pod by using:

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

4.  Mount the PersistentVolume to the /output location by adding the following, which should be level with the `containers` spec in terms of indentation:

    ```
    volumes:
      - name: pv-storage
        persistentVolumeClaim:
           claimName: host-pvc
    ```

5.  In the `containers` spec, below the `command`, set the list of volume mounts by using:

    ```
    volumeMounts:
      - name: pv-storage
        mountPath: /output
    ```

6.  Save and exit the file by hitting the ESC key and using `:wq`.
7.  Finish creating the Pod by using `kubectl create -f pv-pod.yml`.
8.  Check that the Pod is up and running by using `kubectl get pods`.
9.  If you wish, you can log in to the worker node and verify the output data by using `cat /var/output/success.txt`.
