# Lab 13 Kubernetes P2 Test

#  Scenario
Your team wants to deploy a simple Notes App (any lightweight web app, e.g., Nginx or small Node.js server) that:

1. Runs inside its own namespace
2. Uses a ConfigMap for basic settings
3. Uses a Secret for storing a fake database password
4. Stores notes into a persistent volume
5. Is managed through a Deployment (not a bare Pod)

You will build the deployment configuration step by step.

#  Tasks

## **1. Create a Namespace**

Create a new namespace named:

```
notes-app
```


## **2. Create a Persistent Volume (PV)**

Create a **hostPath-based PV** (for Minikube or local cluster) with:

* Name: `notes-pv`
* Capacity: `1Gi`
* Access Mode: `ReadWriteOnce`
* Host path: `/data/notes`

---

## **3. Create a Persistent Volume Claim (PVC)**

Create a PVC:

* Name: `notes-pvc`
* Requests: `1Gi`
* Access Mode: `ReadWriteOnce`
* Should bind to the PV you created above

---

## **4. Create a ConfigMap**

ConfigMap name: `notes-config`

It should contain:

```yaml
APP_TITLE: "Student Notes App"
APP_ENV: "development"
```

---

## **5. Create a Secret**

Secret name: `db-secret`

It should contain a single key:

```
DB_PASSWORD = "supersecretpassword"
```

(Base64-encode it)

---

## **6. Create a Deployment**

Deployment name: **notes-deployment**
Replicas: **2**
Container image: **nginx:alpine** (or any tiny server)

Requirements inside the Pod template:

- Mount the PVC at:

```
/var/notes
```

- Inject ConfigMap values as environment variables

- Inject Secret value as an environment variable called:

```
DB_PASSWORD
```

- Add resource labels (e.g. `app: notes`)

---

Verify that:

- The app is reachable (from curl or browser)
- Both replicas are running
- The PV is bound to the PVC
- The ConfigMap and Secret are visible in the pod (`kubectl exec`)
- The app is using the ConfigMap and Secret values

## Mutlicontainer

Now your Deployment should now run **two containers** inside the same Pod:

### Container 1

* Name: `notes-app`
* Image: `nginx:alpine`
* Mounts the shared volume at: `/var/notes`

### Container 2

* Name: `container2`
* Image: `busybox`
* Mounts **the same volume**
* Periodically prints out the content of `/var/notes` to verify changes

# **Testing the Shared File Synchronization**

Once pods are running:

```bash
kubectl get pods -n notes-app
```

Pick a pod name, then exec into the **main container**:

```bash
kubectl exec -it <pod-name> -n notes-app -c notes-app -- sh
```

Inside the main container, create or modify a file:

```bash
echo "hello from main container" > /var/notes/test.txt
```

Exit the container, then:

### 1️ Check logs of  container2:

```bash
kubectl logs <pod-name> -n notes-app -c container2
```

You should an output similar to this:

```
XXXXX Listing files:
-rw-r--r-- 1 root root 26  test.txt
```

### 2️ Also exec into the container2

```bash
kubectl exec -it <pod-name> -n notes-app -c container2 -- sh
cat /var/notes/test.txt
```

It should display:

```
hello from main container
```

---

#  **Try Updating the File Again**

From the container2:

```bash
echo "edited from container2" >> /var/notes/test.txt
```

Now go back to the **main container**:

```bash
kubectl exec -it <pod-name> -n notes-app -c notes-app -- cat /var/notes/test.txt
```

## Questions

1. How would you rollback to a previous version of the deployment?
2. How would you scale the deployment to 3 replicas?
3. How would you apply a Resource Quota to the namespace? Write the yaml file and the command to apply it.


## Submission
Submit your work on Github with :
- All the yaml files
- A readme file containing the answers to the questions
