# Creating Helm Charts

## Introduction

There are some instances in which a chart for your application might not exist and one must be created; in other cases, you might have a Kubernetes deployment you want to convert into a Helm chart. In this hands-on lab, we will take an existing deployment and convert it into a Helm chart.

## Solution

Log in to the Kubernetes primary server using the credentials provided for the lab. Then, open a second terminal and log in to the server again using the same credentials. Arrange the two consoles as a split screen so you can refer to file contents in one console while working in the other console's editor.

```
ssh cloud_user@<PUBLIC_IP_ADDRESS>
```

**Note:** To save and exit a Vim file, press **Escape** followed by `: x`. To exit a Vim file without saving, press **Escape** followed by `: q!`.

### Convert the Service Manifest into a Service Template in a New Helm Chart

1.  On the first console, create a `blog` directory called `blog`:

    ```
    mkdir blog
    ```

2.  Access the directory:

    ```
    cd blog
    ```

3.  Run the `touch` and `mkdir` commands to create the minimum necessary scaffolding for the new chart. This includes the `Chart.yaml` and `values.yaml` files as well as the `templates` directory:

    ```
    touch Chart.yaml
    ```

    ```
    touch values.yaml
    ```

    ```
    mkdir templates
    ```

4.  View the chart details:

    ```
    ls -l
    ```

5.  Create the `Chart.yaml` file:

    ```
    vim Chart.yaml
    ```

6.  Add the `apiVersion`, `name`, and `version` to the file (this is the minimum data required for _Chart.yaml_):

    ```
    apiVersion: v1
    name: blog
    version: 0.1.0
    ```

7.  Save and exit the file.
8.  Create the `values.yaml` file:

    ```
    vim values.yaml
    ```

9.  On the second console, view the home directory, which contains a _kubernetes_ directory:

    ```
    ls
    ```

10.  Run the `cd` and `ls` commands to open and view the _kubernetes_ directory. The directory contains an _application.yaml_ file and a _service.yaml_ file:

    ```
    $ cd kubernetes/
    ```

    ```
    ls
    ```

11.  View _service.yaml_:

    ```
    vim service.yaml
    ```

12.  On the first console, use the data from _service.yaml_ to add data to _values.yaml_. Update _nodePort_ to `30080`:

    ```
    service:
     name: blog
     type: NodePort
     port: 80
     targetPort: 2368
     nodePort: 30080
    ```

13.  On the first console, save _values.yaml_.
14.  On the second console, exit out of _service.yaml_.
15.  On the second console, run the `cd` command to open the _blog_ folder and run the `vim` command to view the _values.yaml_ file:

    ```
    cd ../blog
    ```

    ```
    vim ./values.yaml
    ```

16.  On the first console, open the _templates_ directory:

    ```
    cd templates/
    ```

17.  Copy the _service.yaml_ file into the _blog_ folder's _templates_ directory:

    ```
    cp ~/kubernetes/service.yaml ./
    ```

18.  Run the `ls` and `vim` commands to view _service.yaml_.

    ```
    ls service.yaml
    ```

    ```
    vim service.yaml
    ```

19.  Use the _values.yaml_ data on the second console to make _service.yaml_ a template on the first console. To do this, update the _service.yaml_ file values as follows:

    ```
    apiVersion: v1
    kind: Service
    metadata:
        name: {{ .Values.service.name }}
    spec:
        type: {{ .Values.service.type }}
        selector:
            app: {{ .Values.service.name }}
        ports:
        -   protocol: TCP
            port: {{ .Values.service.port }}
            targetPort: {{ .Values.service.targetPort }}
            nodePort: {{ .Values.service.nodePort }}
    ```

20.  On the first console, save the template to return to the _templates_ directory.
21.  On the second console, exit out of _values.yaml_ to return to the _blog_ folder.
22.  On the first console, run the `cd` and `helm show values` commands to view the blog details. At this point, we have a full Helm chart:

    ```
    cd ~/
    ```

    ```
    helm show values blog
    ```

23.  Verify the manifest's syntax is correct:

    ```
    helm install demo blog --dry-run
    ```

24.  On the second console, run the `cd` and `cat` commands so you can compare the two _service.yaml_ files.

    ```
    cd ../
    ```

    ```
    cat ./kubernetes/service.yaml
    ```

25.  Confirm the _service.yaml_ data matches on the first and second consoles, with the exception of the _nodePort_ value.
26.  After reviewing the _service.yaml_ data, `clear` both consoles.

    ```
    clear
    ```

### Convert the Manifest for the Application into a Deployment Template in a New Helm Chart

1.  On the second console, view the _application.yaml_ file:

    ```
    vim ./kubernetes/application.yaml
    ```

2.  On the first console, view the _blog_ folder's _values.yaml_ file:

    ```
    vim ./blog/values.yaml
    ```

3.  Below the existing file data, create a new _blog_ section by inserting the following values. You can copy these values from the second console:

    ```
    blog:
      name: blog
      replicas: 1
      image: ghost:2.6-alpine
      imagePullPolicy: Always
      containerPort: 2368
    ```

4.  On the second console, exit out of the chart.
5.  On the first console, save the _values_ file.
6.  On the second console, view the _blog_ folder's _values.yaml_ file:

    ```
    vim ./blog/values.yaml
    ```

7.  On the first console, copy the _kubernetes_ folder's _application.yaml_ file into the _blog_ folder's _templates_ directory:

    ```
    cp ./kubernetes/application.yaml ./blog/templates/
    ```

8.  View the _application.yaml_ file in the _blog_ folder's _templates_ directory:

    ```
    vim ./blog/templates/application.yaml
    ```

9.  Make _application.yaml_ a template by updating the file values as follows:

    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: {{ .Values.blog.name }}
      labels:
        app: {{ .Values.blog.name}}
    spec:
      replicas: {{ .Values.blog.replicas }}
      selector:
        matchLabels:
          app: {{ .Values.blog.name }}
      template:
        metadata:
          labels:
            app: {{ .Values.blog.name }}
        spec:
          containers:
          - name: {{ .Values.blog.name }}
            image: {{ .Values.blog.image }}
            imagePullPolicy: {{ .Values.blog.imagePullPolicy }}
            ports:
            - containerPort: {{ .Values.blog.containerPort }}
    ```

10.  After updating the file values, save the template and `clear` the console:

    ```
    clear
    ```

11.  On the second console, exit out of the _values.yaml_ file.

### Ensure the Manifests Render Correctly and Deploy the Application as a NodePort Application

1.  On the first console, run the `helm show values` command to view the _blog_ folder's details:

    ```
    helm show values blog
    ```

2.  Run the `helm install` command with the `--dry-run` directive. The manifest should display with the service set to run as a _NodePort_ on port _30080_ (in the lab video, this step produced an error message because there was a typo in the _application.yaml_ file):

    ```
    helm install demo blog --dry-run
    ```

3.  Install and deploy Helm:

    ```
    helm install demo blog
    ```

4.  View the pod details (note that the pod's status is _ContainerCreating_):

    ```
    kubectl get po
    ```

5.  While the container is being created, view the service details (note that the blog service is running on the correct _NodePort_ of _30080_):

    ```
    kubectl get svc
    ```

6.  Verify the pod's status is now _Running_:

    ```
    kubectl get po
    ```

7.  On the second console, `exit` out of session so you can view the public IP address for the Kubernetes primary server:

    ```
    exit
    ```

8.  Copy the public IP address of the Kubernetes primary server and paste it into a new browser tab along with the port number: `<PUBLIC_IP_ADDRESS>:30080`.
