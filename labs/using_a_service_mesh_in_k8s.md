![image](https://github.com/user-attachments/assets/1fc49e4b-db08-4664-a0e9-563d39bb9836)

# Using a Service Mesh in Kubernetes

## Introduction

Service meshes can automate the process of providing additional security, reliability, and functionality around your containers. In this lab, you will be able to see how a service mesh works up close by exploring how the service mesh interacts with a simple application.

## Solution

### Explore the Linkerd Dashboard

1.  Copy the public IP address of the Kubernetes server from the lab.
2.  Open a new browser window or tab.
3.  Insert the public IP of your lab's Kubernetes server in place of `<k8s Server Public IP>` in the web address below:

![image](https://github.com/user-attachments/assets/7b0f329b-4ded-44d9-9197-63a0769a7f73)

![image](https://github.com/user-attachments/assets/e79e2710-aeb3-4e66-8ba0-3e7ebb87c040)

![image](https://github.com/user-attachments/assets/ae734a42-c6be-405d-a10f-0f7fc3695ed9)

![image](https://github.com/user-attachments/assets/08055c46-0cf0-4c5b-9b2a-69710071f842)

![image](https://github.com/user-attachments/assets/0562628f-1db1-4470-8a2b-082ccd9e2ffb)


    ```
    http://<k8s Server Public IP>:30080
    ```

4.  Copy and paste the resulting address into your browser's address bar. This should bring you to the Linkerd dashboard for the Kubernetes cluster.
5.  Under **HTTP Metrics**, click **default**. This should let you see our two deployments, which have not been meshed with Linkerd yet.

### Mesh the Application with Linkerd

1.  In a new browser window or tab, launch the `Cloud Server k8s Server` from the lab page to access the Kubernetes control plane node.
2.  Log in to the server using the credentials provided:

    ```
    ssh cloud_user@<PUBLIC_IP_ADDRESS>
    ```

3.  Get a list of the deployments in the `default` Namespace in YAML format, inject Linkerd's configuration into the YAML, and apply those changes using `kubectl apply`:

    ```
    kubectl get -n default deploy -o yaml | \
      linkerd inject - | \
      kubectl apply -f -
    ```

    This should reconfigure the deployments so that Linkerd can inject the sidecar proxies.

4.  Return to the browser window or tab with the Linkerd dashboard for the Kubernetes cluster. You should see the deployments changing as the Pods are being meshed.

### Explore the Changes to Application Components Made by Linkerd

1.  Return to the browser tab or window with the Kubernetes control plane node.
2.  View the list of Pods:

    ```
    kubectl get pods
    ```

3.  Copy the entire Pod name of the first client Pod starting with `terrapin-client-dep`.
4.  View detailed information about the chosen Pod to see the changes that were made when the application was meshed with Linkerd:

    ```
    kubectl describe pod <Pod name>
    ```

5.  Scroll up through the output to see the volumes added by Linkerd and the list of containers, particularly the Linkerd service proxy sidecar that should have been added to the configuration of our deployment.

