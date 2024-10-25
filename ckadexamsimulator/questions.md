## Question 1

### Context 

Solve this question on instance: ssh ckad5601

The DevOps team would like to get the list of all Namespaces in the cluster. Get the list and save it to /opt/course/1/namespaces on ckad5601.


### Solution 
-  SSH into the instance
ssh ckad5601

-  Get the list of all Namespaces and save it to the file
```bash 
kubectl get namespaces -o jsonpath='{.items[*].metadata.name}' > /opt/course/1/namespaces
```

## Question 2

### Context 

Solve this question on instance: ssh ckad5601

Create a single Pod of image httpd:2.4.41-alpine in Namespace default. The Pod should be named pod1 and the container should be named pod1-container.

Your manager would like to run a command manually on occasion to output the status of that exact Pod. Please write a command that does this into /opt/course/2/pod1-status-command.sh on ckad5601. The command should use kubectl.

### Solution 
- SSH into the instance
```sh
ssh ckad5601

kubectl run pod1 --image=httpd:2.4.41-alpine --namespace=default --restart=Never --overrides='
{
  "apiVersion": "v1",
  "spec": {
    "containers": [
      {
        "name": "pod1-container",
        "image": "httpd:2.4.41-alpine"
      }
    ]
  }
}'

echo "kubectl get pod pod1 --namespace=default --output=jsonpath='{.status.phase}'" > /opt/course/2/pod1-status-command.sh
```


## Question 3

### Context 

Team Neptune needs a Job template located at /opt/course/3/job.yaml. This Job should run image busybox:1.31.0 and execute `sleep 2 && echo done`. It should be in namespace neptune, run a total of 3 times and should execute 2 runs in parallel.

Start the Job and check its history. Each pod created by the Job should have the label `id: awesome-job`. The job should be named `neb-new-job` and the container `neb-new-job-container`.

### Solution 
- SSH into the instance
```sh
ssh ckad5601
k -n netune create job neb-new-job --image=busybox:1.31.0 --dry-run=clinet -o yaml > /opt/course/3/job.yaml
vi opt/course/3/job.yaml
```

- update job.yaml
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: neb-new-job
  namespace: neptune
spec:
  completions: 3
  parallelism: 2
  template:
    metadata:
      labels:
        id: awesome-job
    spec:
      containers:
      - name: neb-new-job-container
        image: busybox:1.31.0
        command: ["sh", "-c", "sleep 2 && echo done"]
      restartPolicy: Never
```

```bash 
kubectl apply -f /opt/course/3/job.yaml
kubectl get jobs -n neptune
kubectl get pods -n neptune --selector=job-name=neb-new-job
```

## Question 4

### Context 

Team Mercury asked you to perform some operations using Helm, all in Namespace mercury:

- Delete release `internal-issue-report-apiv1`
- Upgrade release `internal-issue-report-apiv2` to any newer version of chart `bitnami/nginx` available
- Install a new release `internal-issue-report-apache` of chart `bitnami/apache`. The Deployment should have two replicas, set these via Helm-values during install
- There seems to be a broken release, stuck in pending-install state. Find it and delete it

### Solution 

```bash
ssh ckad5601
# Delete release `internal-issue-report-apiv1`
helm delete internal-issue-report-apiv1 -n mercury
# Upgrade release `internal-issue-report-apiv2` to any newer version of chart `bitnami/nginx` available
helm repo update
helm upgrade internal-issue-report-apiv2 bitnami/nginx -n mercury
# Install a new release `internal-issue-report-apache` of chart `bitnami/apache` with two replicas
helm install internal-issue-report-apache bitnami/apache -n mercury --set replicaCount=2
# Find and delete the broken release stuck in pending-install state
helm list -n mercury --filter '.*pending-install' -o json | jq -r '.[].name' | xargs -I {} helm delete {} -n mercury
bash

helm list -n mercury --filter '.*' --pending
helm delete <broken-release-name> -n mercury

```
## Question 5

### Context 

Team Neptune has its own ServiceAccount named neptune-sa-v2 in Namespace neptune. A coworker needs the token from the Secret that belongs to that ServiceAccount. Write the base64 decoded token to file /opt/course/5/token on ckad7326.

### Solution 
```bash
# Get the Secret name associated with the ServiceAccount
kubectl get serviceaccount neptune-sa-v2 -n neptune
kubectl describe serviceaccount neptune-sa-v2 -n neptune

# Retrieve the token from the Secret, decode it from base64, and write it to the file

kubectl get secret $SECRET_NAME -n neptune -o jsonpath='{.data.token}' | base64 --decode > /opt/course/5/token
# or below command will provide decodedbase64 as well
kubectl describe  secret $SECRET_NAME -n neptune 
```

## Question 6

### Context 

Create a single Pod named pod6 in Namespace default of image busybox:1.31.0. The Pod should have a readiness-probe executing `cat /tmp/ready`. It should initially wait 5 and periodically wait 10 seconds. This will set the container ready only if the file `/tmp/ready` exists.

The Pod should run the command `touch /tmp/ready && sleep 1d`, which will create the necessary file to be ready and then idles. Create the Pod and confirm it starts.

### Solution 
```bash
# Create the Pod manifest

vi /opt/course/6/pod6.yaml
```

```yaml
# Add the following content to `pod6.yaml`

apiVersion: v1
kind: Pod
metadata:
  name: pod6
  namespace: default
spec:
  containers:
  - name: pod6-container
    image: busybox:1.31.0
    command: ["sh", "-c", "touch /tmp/ready && sleep 1d"]
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/ready
      initialDelaySeconds: 5
      periodSeconds: 10
```

```bash
# Apply the Pod manifest
kubectl apply -f /opt/course/6/pod6.yaml
# Confirm the Pod starts
kubectl get pod pod6 -n default
```


## Question 7

### Context 

The board of Team Neptune decided to take over control of one e-commerce webserver from Team Saturn. The administrator who once setup this webserver is not part of the organisation any longer. All information you could get was that the e-commerce system is called my-happy-shop.

Search for the correct Pod in Namespace saturn and move it to Namespace neptune. It doesn't matter if you shut it down and spin it up again, it probably hasn't any customers anyways.

### Solution 

```bash
# SSH into the instance
ssh ckad5601

# Find the Pod in Namespace saturn
kubectl get pods -n saturn | grep my-happy-shop

# Save the Pod definition to a file
kubectl get pod my-happy-shop -n saturn -o yaml > /opt/course/7/my-happy-shop.yaml

# Update the Namespace in the Pod definition file
vi /opt/course/7/my-happy-shop.yaml

# Change the namespace from saturn to neptune in the file
# Delete the Pod from Namespace saturn
kubectl delete pod my-happy-shop -n saturn

# Create the Pod in Namespace neptune
kubectl apply -f /opt/course/7/my-happy-shop.yaml -n neptune

# Confirm the Pod is running in Namespace neptune
kubectl get pods -n neptune | grep my-happy-shop
```

## Question 8  IMP

### Context 

There is an existing Deployment named api-new-c32 in Namespace neptune. A developer did make an update to the Deployment but the updated version never came online. Check the Deployment history and find a revision that works, then rollback to it. Could you tell Team Neptune what the error was so it doesn't happen again?

### Solution 

```bash
# SSH into the instance

ssh ckad5601

# Check the Deployment history

kubectl rollout history deployment/api-new-c32 -n neptune

# Identify a revision that works (e.g., revision 2)

# Rollback to the identified revision

kubectl rollout undo deployment/api-new-c32 --to-revision=2 -n neptune

# Describe the Deployment to find the error

kubectl describe deployment api-new-c32 -n neptune

# Check the events section for any errors or issues

kubectl get events -n neptune --field-selector involvedObject.name=api-new-c32

# Inform Team Neptune about the error (e.g., image pull error, incorrect configuration)
```


## Question 9

### Context 

In Namespace pluto there is a single Pod named holy-api. It has been working okay for a while now but Team Pluto needs it to be more reliable.

Convert the Pod into a Deployment named holy-api with 3 replicas and delete the single Pod once done. The raw Pod template file is available at /opt/course/9/holy-api-pod.yaml.

In addition, the new Deployment should set `allowPrivilegeEscalation: false` and `privileged: false` for the security context on container level.

Please create the Deployment and save its yaml under /opt/course/9/holy-api-deployment.yaml.

### Solution 

```bash
# SSH into the instance

ssh ckad5601

# Create the Deployment manifest based on the existing Pod template

cp /opt/course/9/holy-api-pod.yaml /opt/course/9/holy-api-deployment.yaml

# Edit the Deployment manifest

vi /opt/course/9/holy-api-deployment.yaml
```

```yaml
# Update the content to convert it into a Deployment with 3 replicas and the required security context

apiVersion: apps/v1
kind: Deployment
metadata:
  name: holy-api
  namespace: pluto
spec:
  replicas: 3
  selector:
    matchLabels:
      app: holy-api
  template:
    metadata:
      labels:
        app: holy-api
    spec:
      containers:
      - name: holy-api-container
        image: <image-name> # Use the appropriate image name from the Pod template
        securityContext:
          allowPrivilegeEscalation: false
          privileged: false
```
```bash
# Apply the Deployment manifest

kubectl apply -f /opt/course/9/holy-api-deployment.yaml

# Delete the original Pod

kubectl delete pod holy-api -n pluto

# Confirm the Deployment is running with 3 replicas

kubectl get deployment holy-api -n pluto
kubectl get pods -n pluto -l app=holy-api
```

## Question 10 Imp

### Context 

Team Pluto needs a new cluster internal Service. Create a ClusterIP Service named project-plt-6cc-svc in Namespace pluto. This Service should expose a single Pod named project-plt-6cc-api of image nginx:1.17.3-alpine, create that Pod as well. The Pod should be identified by label project: plt-6cc-api. The Service should use tcp port redirection of 3333:80.

Finally use for example curl from a temporary nginx:alpine Pod to get the response from the Service. Write the response into /opt/course/10/service_test.html on ckad9043. Also check if the logs of Pod project-plt-6cc-api show the request and write those into /opt/course/10/service_test.log on ckad9043.

### Solution 

```bash
# SSH into the instance

ssh ckad9043

# Create the Pod

kubectl run project-plt-6cc-api --image=nginx:1.17.3-alpine --namespace=pluto --labels="project=plt-6cc-api"

# Create the Service

kubectl expose pod project-plt-6cc-api --port=3333 --target-port=80 --name=project-plt-6cc-svc --namespace=pluto --type=ClusterIP

# Use a temporary Pod to test the Service and save the response

kubectl run tmp-nginx --rm -i --tty --image=nginx:alpine -- /bin/sh -c "apk add curl && curl project-plt-6cc-svc.pluto.svc.cluster.local:3333 > /opt/course/10/service_test.html"

# Check the logs of the Pod and save them

kubectl logs project-plt-6cc-api -n pluto > /opt/course/10/service_test.log
```

## Question 11 IMP

### Context 

There are files to build a container image located at /opt/course/11/image on ckad9043. The container will run a Golang application which outputs information to stdout. You're asked to perform the following tasks:

ℹ️ Run all Docker and Podman commands as user root. Use sudo docker and sudo podman or become root with sudo -i

- Change the Dockerfile: set ENV variable `SUN_CIPHER_ID` to hardcoded value `5b9c1065-e39d-4a43-a04a-e59bcea3e03f`
- Build the image using `sudo docker`, tag it `registry.killer.sh:5000/sun-cipher:v1-docker` and push it to the registry
- Build the image using `sudo podman`, tag it `registry.killer.sh:5000/sun-cipher:v1-podman` and push it to the registry
- Run a container using `sudo podman`, which keeps running detached in the background, named `sun-cipher` using image `registry.killer.sh:5000/sun-cipher:v1-podman`
- Write the logs your container `sun-cipher` produces into `/opt/course/11/logs` on ckad9043

### Solution 

```bash
# SSH into the instance

ssh ckad9043

# Become root

sudo -i

# Change the Dockerfile to set the ENV variable

vi /opt/course/11/image/Dockerfile

# Add the following line to the Dockerfile

ENV SUN_CIPHER_ID=5b9c1065-e39d-4a43-a04a-e59bcea3e03f

# Build the image using Docker

cd /opt/course/11/image
sudo docker build -t registry.killer.sh:5000/sun-cipher:v1-docker .

# Push the image to the registry using Docker

sudo docker push registry.killer.sh:5000/sun-cipher:v1-docker

# Build the image using Podman

sudo podman build -t registry.killer.sh:5000/sun-cipher:v1-podman .

# Push the image to the registry using Podman

sudo podman push registry.killer.sh:5000/sun-cipher:v1-podman

# Run a container using Podman

sudo podman run -d --name sun-cipher registry.killer.sh:5000/sun-cipher:v1-podman

# Write the logs your container produces into the specified file

sudo podman logs sun-cipher > /opt/course/11/logs
```

## Question 12

### Context 

Create a new PersistentVolume named earth-project-earthflower-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /Volumes/Data and no storageClassName defined.

Next create a new PersistentVolumeClaim in Namespace earth named earth-project-earthflower-pvc. It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

Finally create a new Deployment project-earthflower in Namespace earth which mounts that volume at /tmp/project-data. The Pods of that Deployment should be of image httpd:2.4.41-alpine.

### Solution 

```bash
# SSH into the instance

ssh ckad5601

# Create the PersistentVolume manifest

vi /opt/course/12/earth-project-earthflower-pv.yaml
```
```yaml
# Add the following content to `earth-project-earthflower-pv.yaml`

apiVersion: v1
kind: PersistentVolume
metadata:
  name: earth-project-earthflower-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /Volumes/Data
```
```bash
# Apply the PersistentVolume manifest

kubectl apply -f /opt/course/12/earth-project-earthflower-pv.yaml

# Create the PersistentVolumeClaim manifest

vi /opt/course/12/earth-project-earthflower-pvc.yaml
```
```yaml
# Add the following content to `earth-project-earthflower-pvc.yaml`

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: earth-project-earthflower-pvc
  namespace: earth
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi

```
```bash
# Apply the PersistentVolumeClaim manifest

kubectl apply -f /opt/course/12/earth-project-earthflower-pvc.yaml

# Create the Deployment manifest

vi /opt/course/12/project-earthflower-deployment.yaml
```
```yaml
# Add the following content to `project-earthflower-deployment.yaml`

apiVersion: apps/v1
kind: Deployment
metadata:
  name: project-earthflower
  namespace: earth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: project-earthflower
  template:
    metadata:
      labels:
        app: project-earthflower
    spec:
      containers:
      - name: project-earthflower-container
        image: httpd:2.4.41-alpine
        volumeMounts:
        - mountPath: /tmp/project-data
          name: project-data
      volumes:
      - name: project-data
        persistentVolumeClaim:
          claimName: earth-project-earthflower-pvc
```
```bash
# Apply the Deployment manifest

kubectl apply -f /opt/course/12/project-earthflower-deployment.yaml

# Confirm the PersistentVolume, PersistentVolumeClaim, and Deployment are created and bound correctly

kubectl get pv earth-project-earthflower-pv
kubectl get pvc earth-project-earthflower-pvc -n earth
kubectl get deployment project-earthflower -n earth
kubectl get pods -n earth -l app=project-earthflower
```

## Question 13

### Context 

Team Moonpie, which has the Namespace moon, needs more storage. Create a new PersistentVolumeClaim named moon-pvc-126 in that namespace. This claim should use a new StorageClass moon-retain with the provisioner set to moon-retainer and the reclaimPolicy set to Retain. The claim should request storage of 3Gi, an accessMode of ReadWriteOnce and should use the new StorageClass.

The provisioner moon-retainer will be created by another team, so it's expected that the PVC will not boot yet. Confirm this by writing the event message from the PVC into file /opt/course/13/pvc-126-reason on ckad9043.

### Solution 
```bash
# SSH into the instance

ssh ckad9043

# Create the StorageClass manifest

vi /opt/course/13/moon-retain-sc.yaml
```
```yaml
# Add the following content to `moon-retain-sc.yaml`

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: moon-retain
provisioner: moon-retainer
reclaimPolicy: Retain
```
```bash
# Apply the StorageClass manifest

kubectl apply -f /opt/course/13/moon-retain-sc.yaml

# Create the PersistentVolumeClaim manifest

vi /opt/course/13/moon-pvc-126.yaml
```
```yaml
# Add the following content to `moon-pvc-126.yaml`

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: moon-pvc-126
  namespace: moon
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
  storageClassName: moon-retain
```
```bash
# Apply the PersistentVolumeClaim manifest

kubectl apply -f /opt/course/13/moon-pvc-126.yaml

# Confirm the PVC is not bound and write the event message to the file

kubectl get events -n moon --field-selector involvedObject.name=moon-pvc-126 -o jsonpath='{.items[0].message}' > /opt/course/13/pvc-126-reason
```

## Question 14

### Context 

You need to make changes on an existing Pod in Namespace moon called secret-handler. Create a new Secret secret1 which contains user=test and pass=pwd. The Secret's content should be available in Pod secret-handler as environment variables SECRET1_USER and SECRET1_PASS. The yaml for Pod secret-handler is available at /opt/course/14/secret-handler.yaml.

There is existing yaml for another Secret at /opt/course/14/secret2.yaml, create this Secret and mount it inside the same Pod at /tmp/secret2. Your changes should be saved under /opt/course/14/secret-handler-new.yaml on ckad9043. Both Secrets should only be available in Namespace moon.

### Solution 

```bash
# SSH into the instance

ssh ckad9043

# Create the first Secret

kubectl create secret generic secret1 --from-literal=user=test --from-literal=pass=pwd -n moon

# Create the second Secret from the existing yaml

kubectl apply -f /opt/course/14/secret2.yaml -n moon

# Edit the Pod manifest to include the Secrets

cp /opt/course/14/secret-handler.yaml /opt/course/14/secret-handler-new.yaml
vi /opt/course/14/secret-handler-new.yaml
```
```yaml
# Update the content of `secret-handler-new.yaml`

apiVersion: v1
kind: Pod
metadata:
  name: secret-handler
  namespace: moon
spec:
  containers:
  - name: secret-handler-container
    image: <image-name> # Use the appropriate image name from the Pod template
    env:
    - name: SECRET1_USER
      valueFrom:
        secretKeyRef:
          name: secret1
          key: user
    - name: SECRET1_PASS
      valueFrom:
        secretKeyRef:
          name: secret1
          key: pass
    volumeMounts:
    - name: secret2-volume
      mountPath: /tmp/secret2
      readOnly: true
  volumes:
  - name: secret2-volume
    secret:
      secretName: secret2
```
```bash
# Apply the updated Pod manifest

kubectl apply -f /opt/course/14/secret-handler-new.yaml

# Confirm the Pod is running with the new Secrets

kubectl get pod secret-handler -n moon
```

## Question 15

### Context 

Team Moonpie has a nginx server Deployment called web-moon in Namespace moon. Someone started configuring it but it was never completed. To complete please create a ConfigMap called configmap-web-moon-html containing the content of file /opt/course/15/web-moon.html under the data key-name index.html.

The Deployment web-moon is already configured to work with this ConfigMap and serve its content. Test the nginx configuration for example using curl from a temporary nginx:alpine Pod.

### Solution 

```bash
# SSH into the instance

ssh ckad9043

# Create the ConfigMap from the file

kubectl create configmap configmap-web-moon-html --from-file=index.html=/opt/course/15/web-moon.html -n moon

# Confirm the ConfigMap is created

kubectl get configmap configmap-web-moon-html -n moon

# Test the nginx configuration using a temporary Pod

kubectl run tmp-nginx --rm -i --tty --image=nginx:alpine -- /bin/sh -c "apk add curl && curl web-moon.moon.svc.cluster.local > /opt/course/15/web-moon-test.html"
```


## Question 16

### Context 

The Tech Lead of Mercury2D decided it's time for more logging, to finally fight all these missing data incidents. There is an existing container named cleaner-con in Deployment cleaner in Namespace mercury. This container mounts a volume and writes logs into a file called cleaner.log.

The yaml for the existing Deployment is available at /opt/course/16/cleaner.yaml. Persist your changes at /opt/course/16/cleaner-new.yaml on ckad7326 but also make sure the Deployment is running.

Create a sidecar container named logger-con, image busybox:1.31.0, which mounts the same volume and writes the content of cleaner.log to stdout, you can use the tail -f command for this. This way it can be picked up by kubectl logs.

Check if the logs of the new container reveal something about the missing data incidents.

### Solution 
```bash
# SSH into the instance
ssh ckad7326

# Edit the Deployment to add the sidecar container
vi /opt/course/16/cleaner-new.yaml
```
```yaml
# Add the following content to `cleaner-new.yaml`
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cleaner
  namespace: mercury
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cleaner
  template:
    metadata:
      labels:
        app: cleaner
    spec:
      containers:
      - name: cleaner-con
        image: your-cleaner-image
        volumeMounts:
        - name: log-volume
          mountPath: /var/log
      - name: logger-con
        image: busybox:1.31.0
        command: ["sh", "-c", "tail -f /var/log/cleaner.log"]
        volumeMounts:
        - name: log-volume
          mountPath: /var/log
      volumes:
      - name: log-volume
        emptyDir: {}
```
```bash
# Apply the new Deployment configuration
kubectl apply -f /opt/course/16/cleaner-new.yaml

# Confirm the Deployment is running
kubectl get pods -n mercury

# Check the logs of the logger-con container
kubectl logs -l app=cleaner -c logger-con -n mercury
```

## Question 17

### Context 

Last lunch you told your coworker from department Mars Inc how amazing InitContainers are. Now he would like to see one in action. There is a Deployment yaml at /opt/course/17/test-init-container.yaml. This Deployment spins up a single Pod of image nginx:1.17.3-alpine and serves files from a mounted volume, which is empty right now.

Create an InitContainer named init-con which also mounts that volume and creates a file index.html with content check this out! in the root of the mounted volume. For this test we ignore that it doesn't contain valid html.

The InitContainer should be using image busybox:1.31.0. Test your implementation for example using curl from a temporary nginx:alpine Pod.

### Solution 

```bash
# SSH into the instance
ssh ckad7326

# Edit the Deployment to add the InitContainer
vi /opt/course/17/test-init-container.yaml
```
```yaml
# Add the following content to `test-init-container.yaml`
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-init-container
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-init-container
  template:
    metadata:
      labels:
        app: test-init-container
    spec:
      initContainers:
      - name: init-con
        image: busybox:1.31.0
        command: ["sh", "-c", "echo 'check this out!' > /usr/share/nginx/html/index.html"]
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      containers:
      - name: nginx
        image: nginx:1.17.3-alpine
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html-volume
        emptyDir: {}
```
```bash
# Apply the new Deployment configuration
kubectl apply -f /opt/course/17/test-init-container.yaml

# Confirm the Deployment is running
kubectl get pods -n default

# Test the nginx configuration using a temporary Pod
kubectl run tmp-nginx --rm -i --tty --image=nginx:alpine -- /bin/sh -c "apk add curl && curl test-init-container.default.svc.cluster.local"
```


## Question 18

### Context 

There seems to be an issue in Namespace mars where the ClusterIP service manager-api-svc should make the Pods of Deployment manager-api-deployment available inside the cluster.

You can test this with curl manager-api-svc.mars:4444 from a temporary nginx:alpine Pod. Check for the misconfiguration and apply a fix.

### Solution 
```bash
# SSH into the instance
ssh ckad7326

# Check the Service configuration
kubectl get svc manager-api-svc -n mars -o yaml

# Check the Deployment configuration
kubectl get deployment manager-api-deployment -n mars -o yaml

# Verify the Pods are running
kubectl get pods -n mars -l app=manager-api

# Test the Service using a temporary Pod
kubectl run tmp-nginx --rm -i --tty --image=nginx:alpine -- /bin/sh -c "apk add curl && curl manager-api-svc.mars:4444"

# If there is an issue with the Service or Deployment, edit the configuration
vi /opt/course/18/manager-api-fix.yaml
```
```yaml
# Example fix: Ensure the Service selector matches the Pod labels
apiVersion: v1
kind: Service
metadata:
  name: manager-api-svc
  namespace: mars
spec:
  selector:
    app: manager-api
  ports:
  - protocol: TCP
    port: 4444
    targetPort: 4444
```
```bash
# Apply the fix
kubectl apply -f /opt/course/18/manager-api-fix.yaml

# Confirm the Service is working
kubectl run tmp-nginx --rm -i --tty --image=nginx:alpine -- /bin/sh -c "apk add curl && curl manager-api-svc.mars:4444"
```

## Question 19

### Context 

In Namespace jupiter you'll find an apache Deployment (with one replica) named jupiter-crew-deploy and a ClusterIP Service called jupiter-crew-svc which exposes it. Change this service to a NodePort one to make it available on all nodes on port 30100.

Test the NodePort Service using the internal IP of all available nodes and the port 30100 using curl, you can reach the internal node IPs directly from your main terminal. On which nodes is the Service reachable? On which node is the Pod running?

### Solution 
```bash
# SSH into the instance
ssh ckad7326

# Edit the Service to change it to NodePort
kubectl edit svc jupiter-crew-svc -n jupiter

# Modify the service type to NodePort and set the nodePort to 30100
# The edited section should look like this:
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30100

# Save and exit the editor

# Confirm the Service is updated
kubectl get svc jupiter-crew-svc -n jupiter

# Get the internal IPs of all nodes
kubectl get nodes -o wide

# Test the NodePort Service using curl from your main terminal
curl <node1-internal-ip>:30100
curl <node2-internal-ip>:30100
# Repeat for all available nodes

# Check on which node the Pod is running
kubectl get pods -o wide -n jupiter

# The output will show the node where the Pod is running
```

## Question 20

### Context 

In Namespace venus you'll find two Deployments named api and frontend. Both Deployments are exposed inside the cluster using Services. Create a NetworkPolicy named np1 which restricts outgoing tcp connections from Deployment frontend and only allows those going to Deployment api. Make sure the NetworkPolicy still allows outgoing traffic on UDP/TCP ports 53 for DNS resolution.

Test using: wget www.google.com and wget api:2222 from a Pod of Deployment frontend.

### Solution 
```bash
# SSH into the instance
ssh ckad7326

# Create the NetworkPolicy manifest
vi /opt/course/20/np1.yaml
```
```yaml
# Add the following content to `np1.yaml`
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np1
  namespace: venus
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: api
    ports:
    - protocol: TCP
      port: 2222
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

- First Rule: Ensures that the frontend Pods can communicate with the api Pods on the specified port (2222). This is crucial for the internal communication between these two components of the application.
- Second Rule: Ensures that the frontend Pods can perform DNS lookups by allowing traffic on the standard DNS ports (53 for both TCP and UDP). This is necessary for resolving domain names to IP addresses, enabling the Pods to access external and internal services by name.
```bash
# Apply the NetworkPolicy
kubectl apply -f /opt/course/20/np1.yaml

# Confirm the NetworkPolicy is created
kubectl get networkpolicy np1 -n venus

# Test the NetworkPolicy using a Pod from the frontend Deployment
kubectl exec -it $(kubectl get pod -l app=frontend -n venus -o jsonpath='{.items[0].metadata.name}') -n venus -- wget www.google.com
kubectl exec -it $(kubectl get pod -l app=frontend -n venus -o jsonpath='{.items[0].metadata.name}') -n venus -- wget api:2222
```

## Question 21

### Context 

Team Neptune needs 3 Pods of image httpd:2.4-alpine, create a Deployment named neptune-10ab for this. The containers should be named neptune-pod-10ab. Each container should have a memory request of 20Mi and a memory limit of 50Mi.

Team Neptune has its own ServiceAccount neptune-sa-v2 under which the Pods should run. The Deployment should be in Namespace neptune.

### Solution 

```bash
# SSH into the instance
ssh ckad7326

# Create the Deployment manifest
vi /opt/course/21/neptune-10ab.yaml
```
```yaml
# Add the following content to `neptune-10ab.yaml`
apiVersion: apps/v1
kind: Deployment
metadata:
  name: neptune-10ab
  namespace: neptune
spec:
  replicas: 3
  selector:
    matchLabels:
      app: neptune-10ab
  template:
    metadata:
      labels:
        app: neptune-10ab
    spec:
      serviceAccountName: neptune-sa-v2
      containers:
      - name: neptune-pod-10ab
        image: httpd:2.4-alpine
        resources:
          requests:
            memory: "20Mi"
          limits:
            memory: "50Mi"
```
```bash
# Apply the Deployment manifest
kubectl apply -f /opt/course/21/neptune-10ab.yaml

# Confirm the Deployment is running
kubectl get pods -n neptune
```

## Preview Question 1

### Context 

In Namespace pluto there is a Deployment named project-23-api. It has been working okay for a while but Team Pluto needs it to be more reliable. Implement a liveness-probe which checks the container to be reachable on port 80. Initially the probe should wait 10, periodically 15 seconds.

The original Deployment yaml is available at /opt/course/p1/project-23-api.yaml. Save your changes at /opt/course/p1/project-23-api-new.yaml and apply the changes.

### Solution 

```bash
# SSH into the instance
ssh ckad9043

# Edit the Deployment to add the liveness probe
vi /opt/course/p1/project-23-api-new.yaml
```
```yaml
# Add the following content to `project-23-api-new.yaml`
apiVersion: apps/v1
kind: Deployment
metadata:
  name: project-23-api
  namespace: pluto
spec:
  replicas: 1
  selector:
    matchLabels:
      app: project-23-api
  template:
    metadata:
      labels:
        app: project-23-api
    spec:
      containers:
      - name: project-23-api-container
        image: your-image
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 15
```
```bash
# Apply the new Deployment configuration
kubectl apply -f /opt/course/p1/project-23-api-new.yaml

# Confirm the Deployment is running
kubectl get pods -n pluto
```

## Preview Question 2

### Context 

Team Sun needs a new Deployment named sunny with 4 replicas of image nginx:1.17.3-alpine in Namespace sun. The Deployment and its Pods should use the existing ServiceAccount sa-sun-deploy.

Expose the Deployment internally using a ClusterIP Service named sun-srv on port 9999. The nginx containers should run as default on port 80. The management of Team Sun would like to execute a command to check that all Pods are running on occasion. Write that command into file /opt/course/p2/sunny_status_command.sh. The command should use kubectl.

### Solution 
```bash
# SSH into the instance
ssh ckad7326

# Create the Deployment manifest
vi /opt/course/p2/sunny.yaml
```
```yaml
# Add the following content to `sunny.yaml`
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sunny
  namespace: sun
spec:
  replicas: 4
  selector:
    matchLabels:
      app: sunny
  template:
    metadata:
      labels:
        app: sunny
    spec:
      serviceAccountName: sa-sun-deploy
      containers:
      - name: nginx
        image: nginx:1.17.3-alpine
        ports:
        - containerPort: 80
```
```bash
# Apply the Deployment manifest
kubectl apply -f /opt/course/p2/sunny.yaml

# Create the Service using an imperative command
kubectl expose deployment sunny --name=sun-srv --port=9999 --target-port=80 --type=ClusterIP -n sun

# Write the command to check the status of the Pods into a script
echo "kubectl get pods -n sun -l app=sunny" > /opt/course/p2/sunny_status_command.sh

# Make the script executable
chmod +x /opt/course/p2/sunny_status_command.sh

# Confirm the Deployment and Service are running
kubectl get pods -n sun
kubectl get svc -n sun
```

## Preview Question 3

### Context 

Management of EarthAG recorded that one of their Services stopped working. Dirk, the administrator, left already for the long weekend. All the information they could give you is that it was located in Namespace earth and that it stopped working after the latest rollout. All Services of EarthAG should be reachable from inside the cluster.

Find the Service, fix any issues and confirm it's working again. Write the reason of the error into file /opt/course/p3/ticket-654.txt so Dirk knows what the issue was.

### Solution 

```bash
# SSH into the instance
ssh ckad7326

# List all Services in the Namespace earth to identify the problematic one
kubectl get svc -n earth

# Check the details of each Service to find any misconfigurations
kubectl describe svc <service-name> -n earth

# Check the associated Pods and their status
kubectl get pods -n earth -l app=<app-label>

# Describe the Pods to find any issues
kubectl describe pod <pod-name> -n earth

# If a misconfiguration is found, edit the Service or Deployment as needed
vi /opt/course/p3/service-fix.yaml
```
```yaml
# Example fix: Ensure the Service selector matches the Pod labels
apiVersion: v1
kind: Service
metadata:
  name: <service-name>
  namespace: earth
spec:
  selector:
    app: <app-label>
  ports:
  - protocol: TCP
    port: <service-port>
    targetPort: <container-port>
```
```bash
# Apply the fix
kubectl apply -f /opt/course/p3/service-fix.yaml

# Confirm the Service is working
kubectl run tmp-nginx --rm -i --tty --image=nginx:alpine -- /bin/sh -c "apk add curl && curl <service-name>.earth.svc.cluster.local:<service-port>"

# Write the reason of the error into a file
echo "The issue was due to a misconfiguration in the Service selector which did not match the Pod labels." > /opt/course/p3/ticket-654.txt
```
