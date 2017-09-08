# Learning Kubernetes
Kubernetes is an open-source platform for automating deployment, scaling, and operations of application containers across clusters of hosts, providing container-centric infrastructure.
With Kubernetes, you are able to quickly and efficiently respond to customer demand:
- Deploy your applications quickly and predictably.
- Scale your applications on the fly.
- Roll out new features seamlessly.
- Limit hardware usage to required resources only.

For more information see: https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/

This course provides a first introduction in Kubernetes. A set of examples are given which will provide hands on experience in using kubernetes.

Prerequisites with this course:
- a pre-configured kubernetes cluster
- a pre-configured computer to build up a connection to the cluster

The only thing installed on your local computer is a browser to do this workshop, Chrome is advised.

For all notes are handed out with the correct URL, userid and passwd.

The slides with this workshop can be found at: https://www.slideshare.net/secret/2NQIgW0xrw4b1q

## 1 Getting information on the cluster

Browse to the provided URL and login to the shell with the provided credentials. 
You are now on the shell of a container running in the cluster.
You can retrieve information on the cluster by querying the kubernetes API. 

You can query the API directly with curl but to make life easy we will use the program kubectl to talk to the api. You can for instance collect data about the cluster using:

```
$ kubectl cluster-info
Kubernetes master is running at https://kubernetes
KubeDNS is running at https://kubernetes/api/v1/namespaces/kube-system/services/kube-dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

To find out at what version the cluster is running use:

```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.2", GitCommit:"08e099554f3c31f6e6f07b448ab3ed78d0520507", GitTreeState:"clean", BuildDate:"2017-01-12T04:57:25Z", GoVersion:"go1.7.4", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.2", GitCommit:"08e099554f3c31f6e6f07b448ab3ed78d0520507", GitTreeState:"clean", BuildDate:"2017-01-12T04:52:34Z", GoVersion:"go1.7.4", Compiler:"gc", Platform:"linux/amd64"}
```

To see the nodes the cluster contains use:

```
$ kubectl get nodes
NAME         STATUS                     AGE
gluster0     Ready,SchedulingDisabled   1d
gluster1     Ready,SchedulingDisabled   1d
gluster2     Ready,SchedulingDisabled   1d
k8smaster0   Ready,SchedulingDisabled   1d
k8snode0     Ready                      1d
k8snode1     Ready                      1d
```

In above example only two nodes are available to schedule worloads on. The rest is tasked with other jobs (master processes and guster for persistent storage).

## 2 namespaces

On the cluster different namespaces have been deployed in preparation of this course. A namespace separates resources. You can view the namespaces deployed on this cluster using:

```
$ kubectl get namespaces
NAME             STATUS    AGE
default          Active    1d
jenkins-slaves   Active    1d
kube-system      Active    1d
platform         Active    1d
student0         Active    1d
student1         Active    1d
student2         Active    1d
student3         Active    1d
student4         Active    1d
student5         Active    1d
student6         Active    1d
student7         Active    1d
student8         Active    1d
student9         Active    1d
```

For this workshop each student has its own namespace to prevent you from getting in the way of each other. Your computer had been preconfigured to use a specific namespace. To view the configuration specific to you computer use:

```
$ kubectl config view
```

## 3 creating your first pod

A pod is set of containers running together and sharing a network. The containers are deployed together on the same host within a certain namespace. A pod can be launched using the command line tool kubectl. 

In this example you will deploy a pod with a single container using the image: `kpnappfactory/testcontainer`.

```
$ kubectl run example --image=kpnappfactory/testcontainer:1.0.0 --port 80 --restart=Never
pod "example" created
```

Once you have deployed the pod you should be able to see it using:

```
$ kubectl get pods
```

You can view the details of the pod using:

```
$ kubectl describe pod <podname>
Name:		example
Namespace:	platform
Node:		k8snode1/10.0.0.14
Start Time:	Tue, 13 Jun 2017 14:37:59 +0200
Labels:		<none>
Status:		Running
IP:		10.47.0.14
Controllers:	<none>
...
```

You should be able to ping the example pod from you shell. Use the IP address you just found:

```
ping 10.47.0.14
64 bytes from 10.47.0.14: seq=1 ttl=64 time=0.540 ms
64 bytes from 10.47.0.14: seq=2 ttl=64 time=1.187 ms
^C
--- 10.47.0.14 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.540/1.112/1.825 ms
```

The example pod is running a webserver you should be able to query. We should be able to query the webserver from the shell conatiner on the same pod network. For example by using a command line web browser (replace the IP address with the address you have found yourself):

```
lynx http://10.47.0.14/
```
After the pod is started you should press `enter` before you see the lynx browser in your terminal. Press `q` followed by `enter` to quit.
Note! This webserver is running on the Weave network whithin the cluster and is only accessible from inside the cluster. The shell container is in the cluster.

## 4 service & labels
To expose a pod you should create a service. With the service you will create a clusterIP to forward traffic to the different pods. To create a service use:

```
kubectl create service clusterip example --tcp=80:80
service "example" created
```

Now you have a service with a cluster IP created. The service will direct all traffic send to its cluster IP to the pod with a label `app=example`.

To view the details of this service use:

```
$ kubectl describe service example
Name:			example
Namespace:		platform
Labels:			app=example
Selector:		app=example
Type:			ClusterIP
IP:			    10.0.19.127
Port:			80-80	80/TCP
Endpoints:		<none>
Session Affinity:	None
No events.
```
As you can see no Endpoints are present yet. This is because no pods exist with the label `app=example`. To direct traffic from this service to the correct pod you should add this label to the pod `example`. Do this the following command:

```
$ kubectl label pod example app=example
pod "example" labeled
```

Now when you do a describe of the service `example` you should see an endpoint belonging to this service:

```
$ kubectl describe service example
Name:			example
Namespace:		platform
Labels:			app=example
Selector:		app=example
Type:			ClusterIP
IP:			10.0.19.127
Port:			80-80	80/TCP
Endpoints:		10.47.0.14:80
Session Affinity:	None
No events.
```

To test if we get the same result querying this service use the following command (replace the IP address with the address you have found yourself):

```
lynx http://10.0.19.127
```

You will get the same command line browser as before with still the same information showed. The pod name and address are still the same. 

A service can front more than one pods. It actually fronts every pod with matching labels. We can for example create a second pod and add the same labels to it:

```
$ kubectl run second-example-pod --image=kpnappfactory/testcontainer:1.0.0  --port 80 --restart=Never
pod "second-example-pod" created
```

When we also add the same label to this pod, our service will also note this pod as an endpoint:

```
$ kubectl label pod second-example-pod app=example
pod "second-example-pod" labeled

$ kubectl describe service example
Name:			example
Namespace:		platform
Labels:			app=example
Selector:		app=example
Type:			ClusterIP
IP:			10.0.19.127
Port:			80-80	80/TCP
Endpoints:		10.35.0.2:8000,10.47.0.14:80
Session Affinity:	None
No events.
```
When you now query the service mulitple times you will see both pod names displayed alternating (replace the IP address with the address you have found yourself):

```
lynx http://10.0.19.127
```

We can clean up the pods and services by deleting them:

```
$ kubectl delete pod example
pod "example" deleted

$ kubectl delete pod second-example-pod
pod "second-example-pod" deleted

$ kubectl delete service example
service "example" deleted
```

## 5 Deployments

As you can see when a pod is deleted it won't be recreated. To automatically (re)create pods you will need a `replicationcontroller` or a `replicaset`. You can automatically create replicaset's using a deployment. With deployments will create a replicaset with the number of pods desired. When a pod is deleted, the replicaset will create a new pod. When you want to upgrade or reconfigure your deployment, simply apply the new configuration to the deployment and a new replicaset is created for you with the new configurations. New pods are created and old pods are deleted.

Using the previous example, you can create your own deployment using:

```
$ kubectl run example --image=kpnappfactory/testcontainer:1.0.0  --port 80
deployment "example" created
```
The only difference here is that the option `--restart=Never` is missing. Now you can see the deployment you just created with:

```
$ kubectl get deployments
NAME                           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
example                        1         1         1            1           1m
```

And you can see the replicasets created by the deployments using:

```
$ kubectl get replicasets
NAME                                     DESIRED   CURRENT   READY     AGE
example-1745274054                       1         1         1         3m
```

To see the details of the deployment use:

```
$ kubectl describe deployment example
Name:			example
Namespace:		platform
CreationTimestamp:	Tue, 13 Jun 2017 23:58:46 +0200
Labels:			run=example
Selector:		run=example
Replicas:		1 updated | 1 total | 1 available | 0 unavailable
StrategyType:		RollingUpdate
MinReadySeconds:	0
RollingUpdateStrategy:	1 max unavailable, 1 max surge
Conditions:
  Type		Status	Reason
  ----		------	------
  Available 	True	MinimumReplicasAvailable
OldReplicaSets:	<none>
NewReplicaSet:	example-1745274054 (1/1 replicas created)
Events:
  FirstSeen	LastSeen	Count	From				SubObjectPath	Type		Reason			Message
  ---------	--------	-----	----				-------------	--------	------			-------
  4m		4m		1	{deployment-controller }			Normal		ScalingReplicaSet	Scaled up replica set example-1745274054 to 1
```

We can scale the deployment to a larger number of pods using:

```
$ kubectl scale deployment example --replicas=2
deployment "example" scaled
```

As you can see by viewing the pods deployed:
```
kubectl get pods
NAME                                           READY     STATUS    RESTARTS   AGE
example-1745274054-5d8r7                       1/1       Running   0          51s
example-1745274054-qm9h0                       1/1       Running   0          7m
```

Now we want to upgrade our deployment to a new version. First we check the strategy for this deployment. This can be either: `RollingUpdate` or `Recreate`. Default this is set to `RollingUpdate`. The controller will than always keep a minimum amount of pods available. With `Recreate` all pods are first destroyed before new ones are created.

```
$ kubectl get deployment example -oyaml
...
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
...
```

We want to update our testcontainer to version `1.0.1`. To upgrade our example deployment we can either edit our deployment using:

method 1: edit the deployment and change the version.
```
$ kubectl edit deployment example
```

method 2: specify the new image using the command line
```
$ kubectl set image deployment/example example=kpnappfactory/testcontainer:1.0.1
deployment "example" image updated
```

After this you can view the status of the rollout using:
```
$ kubectl rollout status deployment/example
deployment "example" successfully rolled out
```

You can now also see two replicasets (old and new) using:
```
$ kubectl get replicasets
NAME                                     DESIRED   CURRENT   READY     AGE
example-1745274054                       0         0         0         35m
example-824231406                        2         2         2         2m
```

You can undo a deployment using:
```
$ kubectl rollout undo deployment/example
deployment "example" rolled back
```

## 6 Using yaml

Simple resources can be created using the commandline only but for more complex resources using a yaml specification is a must. When you create a resource via kubectl as done before you can view the yaml by using:

```
kubectl get deployment/example -o yaml
```

Lets first clean up the deployed resources and than create a deployment using a yaml.
```
$ kubectl delete deployment example
deployment "example" deleted
```

Now create a file named `example.yml` with the configuration below. You van use vim, emacs or nano as your editor.
```
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: example
  name: example
spec:
  replicas: 2
  selector:
    matchLabels:
      app: example
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - image: kpnappfactory/testcontainer:1.0.1
        imagePullPolicy: IfNotPresent
        name: example
        ports:
        - containerPort: 80
          protocol: TCP
```

We can create a new deployment with this yaml file using:
```
$ kubectl create -f example.yml
deployment "example" created
```

You can see the pods created with:
```
kubectl get pods
NAME                                           READY     STATUS              RESTARTS   AGE
example-3852110269-39rk9                       1/1       Running             0          3m
example-3852110269-znqx6                       1/1       Running             0          3s
```

When you kill one of the pods it will be recreated by the controller.


For more information on deployments see: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

## 7 ingresses
To make a service available outside the cluster we can either use a `nodeport` or an `ingress`. Nodeports have the disadvantage that you need to keep record of the ports you use to expose a service. Each nodeport is exposed on every node on the cluster. A better way is using ingresses. An ingress uses name based or path based http routing to send the traffic to the correct service within the cluster.

To create an ingress we first need to create a service. Create a yaml file named example-service.yml containing:
```
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: example
  name: example
spec:
  ports:
  - name: 80-80
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: example
  type: ClusterIP
```

And create the service using:
```
$ kubectl create -f example-service.yml
service "example" created
```

After that create a file named example-ingress.yml containing:
```
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: example
spec:
  rules:
  - host: user11.workshop-ws01.kpnappfactory.nl
    http:
      paths:
      - backend:
          serviceName: example
          servicePort: 80
        path: /
```

And create this ingress using:
```
$ kubectl create -f example-ingress.yml
ingress "example" created
```
You can test the ingress by going to the url specified in the ingress, e.g.: `http://user11.workshop-ws01.kpnappfactory.nl`.

Don't forget to change the URL to the one belonging to your user account!!!

## 8 persistent storage

Not all your applicaties will probably be stateless. To enable applications which need a form of state stored on disk you can use `persistent volume claims`. Kubernetes doen't come out of the box with a storage solution. There are several ways to enalbe the persistent storage and hand out `persistent volumes` to the applications. In this environment a gluster setup will provide the persistent storage and with heketi the provisioning of persistent storage is automated.

We will create our own persistent volume claim and link a deployment to it.

Create a file called: `pvc.yml` with the following content:
```
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Now create the pvc using:
```
$ kubectl create -f pvc.yml 
persistentvolumeclaim "my-pvc" created
```

You can get the status and see if the persistent volume has been bound to a persistent volume.

```
$ kubectl get pvc my-pvc
NAME      STATUS    VOLUME                                     CAPACITY   ACCESSMODES   STORAGECLASS        AGE
my-pvc    Bound     pvc-c1acaf5c-93ce-11e7-a20d-000d3a2a7d16   1Gi        RWO           glusterfs-storage   1m

$ kubectl describe pvc my-pvc
Name:           my-pvc
Namespace:      namespace11
StorageClass:   glusterfs-storage
Status:         Bound
Volume:         pvc-c1acaf5c-93ce-11e7-a20d-000d3a2a7d16
Labels:         <none>
Annotations:    pv.kubernetes.io/bind-completed=yes
                pv.kubernetes.io/bound-by-controller=yes
                volume.beta.kubernetes.io/storage-class=glusterfs-storage
                volume.beta.kubernetes.io/storage-provisioner=kubernetes.io/glusterfs
Capacity:       1Gi
Access Modes:   RWO
Events:         <none>
```

We can bound this volume to a set of containers, e.g. to a deployment:

Create a file called deployment-with-pvc.yml
```
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: example
  labels:
    app: example
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
        - name: example
          image: nginx
          ports:
            - containerPort: 80
              name: "http-server"
          volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: my-volume
      volumes:
        - name: my-volume
          persistentVolumeClaim:
           claimName: my-pvc
```
and create the deployment with:
```
$ kubectl create -f deployment-with-pvc.yml
```

When you go into one of the pods you should be able to see the attached volume. 
Find the pod id with `kubectl get pods` and go into the pod with `kubectl exec`:

```
$ kubectl get pods
NAME                                           READY     STATUS              RESTARTS   AGE
example-3852110269-39rk9                       1/1       Running   0          7m
example-3852110269-znqx6                       1/1       Running   0          3m

$ kubectl exec -it example-3852110269-39rk9 bash
root@example-3852110269-39rk9:/usr/src/app# df              
Filesystem                                     1K-blocks    Used Available Use% Mounted on
overlay                                        206291640 4863240 190926360   3% /
tmpfs                                            7181608       0   7181608   0% /dev
tmpfs                                            7181608       0   7181608   0% /sys/fs/cgroup
/dev/sda9                                       28454284 1774748  25191288   7% /etc/hosts
/dev/sdb1                                      206291640 4863240 190926360   3% /etc/hostname
shm                                                65536       0     65536   0% /dev/shm
10.0.0.10:vol_0c08b3c8a4ace704cd2494c8cd9327a3   1044224   33664   1010560   4% /usr/share/nginx/html
tmpfs                                            7181608      12   7181596   1% /run/secrets/kubernetes.io/serviceaccount
```
The same volume is attached to both pods.

Since we are still in the pod lets put a simple webpage on our new volume:
```
# echo "<html><body>hello world</html></body>" > /usr/share/nginx/html/index.html
```

When we go to the same webpage as before we should see our newly created page.

## 9 Network policies

Per namespace it is possible to block all traffic toward pods and only allow certain traffic depending on the origin. To allow certain traffic you define a networkpolicy. Currently networkpolicies are already enabled for you namespace. You can view the active network policies by:

```
$ kubectl get networkpolicies
NAME                               POD-SELECTOR        AGE
allow-all                          <none>              1d
allow-everyone-to-access-ingress   app=nginx-ingress   19h
allow-ingress-to-access-pods       <none>              19h
```

To test this spin up a nginx pod and try to ping it:
```
$ kubectl run nginx --image=nginx --restart=Never
pod "nginx" created

$ kubectl get pods -owide
NAME                                           READY     STATUS    RESTARTS   AGE       IP           NODE
nginx                                          1/1       Running   0          36m       10.33.0.3    ws01node5
```

And ping try to ping it using a busybox pod:

```
$ kubectl run busybox -i --image=busybox --restart=Never --rm --tty -- ping 10.33.0.3
If you don't see a command prompt, try pressing enter.
64 bytes from 10.33.0.3: seq=1 ttl=64 time=0.751 ms
...
```

Now delete the allow-all networkpolicy and try to do the same:
```
$ kubectl delete networkpolicies allow-all
networkpolicy "allow-all" deleted
```
You should not be able to ping the nginx pod.

To allow access set the following networkpolicy:
```
---
apiVersion: extensions/v1beta1
kind: NetworkPolicy
metadata:
  name: allow-busybox-to-nginx
spec:
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: busybox
  podSelector:
    matchLabels:
      run: nginx
```

With this networkpolicy applied you should be able to repeat the pingtest as done before.

