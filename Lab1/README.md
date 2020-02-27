# Lab 1. Deploy your first application

Learn how to deploy an application to a Kubernetes cluster hosted within
the IBM Container Service.

## 1. Deploy the guestbook application

In this part of the lab we will deploy an application called `guestbook`
that has already been built and uploaded to DockerHub under the name
`ibmcom/guestbook:v1`.

1. Check that you are connected to your remote Kubernetes cluster,

	```sh
	$ kubectl config current-context
	```

2. If you are not connected, reconnect by following the instructions at https://github.com/remkohdev/AppModernization/tree/master/Lab02


3. Start by running `guestbook`:

    ```sh
    $ kubectl run guestbook --image=ibmcom/guestbook:v1
    kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
    deployment.apps/guestbook created
    ```

    This action will take a bit of time. To check the status of the running application, you can use `$ kubectl get pods`.

    You should see output similar to the following:

    ```console
    $ kubectl get pods
    NAME                          READY     STATUS              RESTARTS    AGE
    guestbook-59bd679fdc-bxdg7    0/1       ContainerCreating   0          1m
    ```

   	Eventually, the status should show up as `Running`.

	```console
	$ kubectl get pods
	NAME                          READY     STATUS              RESTARTS   AGE
	guestbook-59bd679fdc-bxdg7    1/1       Running             0          1m
	```

	If you already pre-installed an Istio enabled Bookinfo example application, the complete list will show,

    ```sh
    kubectl get pods
	NAME                              READY   STATUS    RESTARTS   AGE
	details-v1-75bbcf5bb-6l45k        2/2     Running   0          5h11m
	guestbook-78cff44bf8-c5rrb        1/1     Running   0          52s
	productpage-v1-754f4b4898-rhvgs   2/2     Running   0          5h11m
	ratings-v1-59d69957fd-j4rck       2/2     Running   0          5h11m
	reviews-v1-58db679fc8-2bdb9       2/2     Running   0          5h11m
	reviews-v2-99478c5cf-4tsng        2/2     Running   0          5h11m
	reviews-v3-d457ddf76-ds6vz        2/2     Running   0          5h11m
	```

    The end result of the run command is not just the pod containing our application containers, but a Deployment resource that manages the lifecycle of those pods.

1. Once the status reads `Running`, we need to expose that deployment as a
   service so we can access it through the IP of the worker nodes.
   The `guestbook` application listens on port 3000.  Run:

   ```console
   $ kubectl expose deployment guestbook --type="NodePort" --port=3000
   service "guestbook" exposed
   ```

2. To find the port used on that worker node, examine your new service:

   ```console
   $ kubectl get service guestbook
   NAME        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
   guestbook   NodePort   172.21.132.34  <none>        3000:31208/TCP   1m
   ```

   We can see that our `<nodeport>` is `31208`. We can see in the output the port mapping from 3000 inside 
   the pod exposed to the cluster on port 31208. This port in the 31000 range is automatically chosen, 
   and could be different for you.

3. `guestbook` is now running on your cluster, and exposed to the internet.    
    We need to find out where it is accessible. The worker nodes running in the container service get external IP addresses. Get the workers for your cluster and note one (any one) of the public IPs listed on the `<public-IP>` line.

    ```console
    $ ibmcloud ks workers $CLUSTER_NAME
    OK
    ID   Public IP    Private IP    Flavor    State    Status   Zone    Version   
	kube-bmt54ghd0i8vpnd5r1a0-usbankikscl-default-0000015e   150.238.6.198   10.93.209.146   b3c.4x16.encrypted   normal   Ready    dal10   1.14.8_1537   
	kube-bmt54ghd0i8vpnd5r1a0-usbankikscl-default-000002b0   150.238.6.216   10.93.209.182   b3c.4x16.encrypted   normal   Ready    dal10   1.14.8_1537   
	kube-bmt54ghd0i8vpnd5r1a0-usbankikscl-default-000003f6   150.238.6.204   10.93.209.184   b3c.4x16.encrypted   normal   Ready    dal10   1.14.8_1537
    ```

    We can see that we have 3 `<public-IP>`, one is `150.238.6.198`.

1. Now that you have both the address and the port, you can now access the application in the web browser
   at `<public-IP>:<nodeport>`. In the example case this is `150.238.6.198:31208`.

Congratulations, you've now deployed an application to Kubernetes!

When you're all done, continue to the
[next lab of this course](../Lab2/README.md).

