### Kibana & Kubernetes Monitoring with EFK

    kubectl Cheat Sheet -> https://kubernetes.io/docs/reference/kubectl/cheatsheet/


### 시스템 구성도

![efk-cluster-logging-setup](https://user-images.githubusercontent.com/54794815/189737919-83db5f4e-8ef9-4312-a09a-b2a0886671d8.png)


### Kubernetes cluster 구성

    contain 1 (single-node cluster): 구성

    $ kubectl get nodes
    ```
    To get information about our cluster:
    ```shell
    $ kubectl describe node docker-desktop
    ```
    Or
    ```shell
    $ kubectl cluster-info

### Creating the cluster namespace


    To view the current active namespaces: 
    ```shell
    $ kubectl get namespaces
    ```
    Now that we have seen the list, let us start by creating our first own namespace:
    ```shell
    $ kubectl apply -f 1-efk-logging-ns.yaml
    ```
    Let us confirm the namespace is created:
    ```shell
    $ kubectl get namespaces
    
### Creating the Elasticsearch cluster


    We will be creating a 3-node Elasticsearch cluster in this section. Why a 3-node cluster you may ask? It's a best practice, 
    and prevents the "split-brain" situation in a highly available multi-node cluster. In short, "split-brain" arises when one or more
    nodes can't communicate with each other (i.e. node failure, network disconnect, etc) and a new master can't be elected. 
    With a 3-node cluster if one of the nodes get disconnected then by election a new master can be assigned from the
    remaining nodes.

### Headless Service

    Create the headless service:
    ```shell
    $ kubectl apply -f 2-elasticsearch-svc.yaml
    ```
    Read [Headless Services & Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) for more information.


### Elasticsearch StatefulSet
Rolling out the 3-node Elasticsearch cluster in Kubernetes requires a different type of resource also known as the
StatefulSet. It provides pods with a stable identity and grants them stable persistent storage. Elasticsearch requires 
stable persistent storage to persist data between Pod rescheduling and restarts. To learn more about 
[StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) visit the kubernetes.io website.

![Kubernetes StatefulSet](images/kubernetes-statefulset.png)

To create the Elasticsearch StatefulSet:
```shell
$ kubectl apply -f 3-elasticsearch-sts.yaml
```
We can follow the creation by running the following command: 
```shell
$ kubectl rollout status sts/es-cluster -n efk-logging
```
We should also be able to see all elasticsearch pods running in the provided namespace:
```shell
$ kubectl get pods -n efk-logging
```

#### Init Containers 
Using the `initContainers` section in the StatefulSet we can apply 
[Important System Configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html) 
settings before launching the ElasticSearch containers. We need to do this according to the documentation to prevent issues
when running the Elasticsearch cluster.

#### Volume Claim Templates
Last but not least the StatefulSet needs some form of storage which is defined under `volumeClaimTemplates`. Under the hood
Kubernetes creates the PersistentVolumeClaim and PersistentVolume resources. In our example we omitted the `storageClassName`
field under `volumeClaimTemplates.spec` so the default of `hostPath` is chosen. Kubernetes supports `hostPath` for 
development and testing on a single-node cluster. A hostPath PersistentVolume uses a file or directory on the Node to 
emulate network-attached storage. Read [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/) 
to learn about supported storage types.

To get information about the PersistentVolumeClaim pvc, run:
```shell
$ kubectl get pvc -n efk-logging
```
The following will display the PersistentVolume pv:
```shell
$ kubectl get pv -n efk-logging
```
As a result we should have ended up with 3 pvc's total, one for each node in the Elasticsearch cluster.

#### Resolving DNS
Prior to rolling out the StatefulSet we talked about a headless service not returning a single IP address. We can verify
this by running a special Pod with DNS utilities, and run a command to perform a service lookup to obtain all the Pod IP 
addresses backing the headless service:
```shell
# install & run the DNSUtils Pod in the correct namespace, so that it has access to the headless service!
$ kubectl run dnsutils --image=tutum/dnsutils -n efk-logging -- sleep infinity
# run nslookup command against the elasticsearch service, should return multiple IP's
$ kubectl exec dnsutils -n efk-logging -- nslookup elasticsearch
```
The endpoint resource can also provide us with IP address information:
```shell
# checking the service endpoints
$ kubectl get endpoints elasticsearch -n efk-logging
# or running describe to get the complete output of the service endpoints
$ kubectl describe endpoints elasticsearch -n efk-logging
```
Not only will Kubernetes assign each Service/Pod with its own IP address it also adds DNS records. This makes it easier
for clients to find those services instead of using IP addresses. Go to kubernetes.io to learn more about
[Kubernetes DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/).

Using the same DNSUtils Pod we can extract DNS information by running a nslookup on one of the Elasticsearch Pods:
```shell
# returns the FQDN for the pod, remember to run nslookup first on the headless service
$ kubectl exec dnsutils -n efk-logging -- nslookup <pod-ip-address>
```
The result should follow a pattern that looks like this: `{pod-name}.{service-name}.{namespace}.svc.cluster.local`. In our
case it should resemble the following values: `es-cluster-[0,1,2].elasticsearch.efk-logging.svc.cluster.local`. Running
the StatefulSet with multiple replicas creates Pods with the supplied name followed by an ordinal number, as can be seen
in the DNS entries. Kubernetes also uses this number when doing RollingUpdates as it will start with the highest number 
and ending with 0.

### Elasticsearch cluster REST-API
Now that our Elasticsearch cluster is running we can access its REST API and verify everything is running as expected. Doing
so allows us to see how our running cluster configuration looks like and what the health state is. To make this REST API 
available we made `port 9200` available while creating the headless service. Since we cannot access the REST API service
directly we will `port-forward` it and make it available outside the cluster.

Port-forwarding is as easy as running one single command:
```shell
$ kubectl port-forward es-cluster-0 -n efk-logging 9200:9200
```
Open a new Terminal window so that we can query the REST API interface:
```shell
$ curl -XGET 'localhost:9200/_cluster/health?pretty'
# we can also ask information about the running nodes on our cluster
$ curl -X GET "localhost:9200/_cluster/state/nodes?pretty"
```
To get some cluster topology in view, run:
```shell
# the result contains IP addresses, roles and metrics
curl -X GET "localhost:9200/_cat/nodes?v"
```
We should see node- IP addresses, metrics, roles and current master node. See
`cat nodes` from [CAT API](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cat-nodes.html) and customize the 
output by adding headers to the request.

The above examples only touch the `Cluster API/CAT API` but there are more, 
go to [REST API](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/rest-apis.html) for more information.

## Creating the Service & Kibana Deployment
In this section we will create the Kibana Service and Deployment resources and execute them to have a running Kibana 
instance. Kibana will then provide us with the centralized interface where we can analyze application logs. 

We will rollout a single Kibana instance:
```shell
$ kubectl apply -f 4-kibana.yaml
```
Verify rollout is succesful:
```shell
$ kubectl rollout status deployment/kibana -n efk-logging
```
Kibana should now be running on the Kubernetes cluster, and to be able to access the dashboard we will also need to 
apply port-forwarding.

```shell
# first we must retrieve the node name
$ kubectl get nodes -n efk-logging
# copy-past the kibana-{id} node name in the following command
$ kubectl port-forward <kibana-node> -n efk-logging 5601:5601
```
Open a browser and go to <http://localhost:5601>, you should see Kibana loading up. Don't do anything further at
this point. We will configure an index later.

## Creating the FluentD DaemonSet
In this final step we will start rolling out the FluentD log collector. We will do this by introducing, yet again, a new 
resource type, the DaemonSet. Kubernetes will roll out a copy of this Pod(node-logging agent) on each node in the Kubernetes 
cluster. The idea behind this solution is that the logging-agent running as a container in a Pod will have access to
a directory with log files from all the application containers on that Kubernetes node. The logging-agent will collect and
parse (supports formats like MySQL, Apache and many more) them and ship them to a new destination like Elasticsearch, 
Amazone S3 or a third-party log management solution.

Read [Cluster-level logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/#cluster-level-logging-architectures)
for other approaches.

![DaemonSet](images/kubernetes-daemonset.png)

To roll out the FluentD DaemonSet, run the following command:
```shell
$ kubectl apply -f 5-fluentd.yaml
```
Looking at the FluentD DaemonSet configuration we just rolled out, you'll see that other resource types were needed, like 
the ServiceAccount, ClusterRole and ClusterRoleBinding. All these resources are needed to cover the following:
* ServiceAccount: Provides FluentD with access to the Kubernetes-API
* ClusterRole: Grants permissions (get, list and watch) on the Pods and Namespaces. In other words grants access to other
cluster-scoped Kubernetes resources like Nodes
* ClusterRoleBinding: Binds ServiceAccount to the ClusterRole. Grants ServiceAccount permissions listed in the ClusterRole

Let's verify the DaemonSet was rolled out successfully:
```shell
# should return 1 FluentD instance
$ kubectl get ds -n efk-logging
```

## Analyze logs with Kibana
Now it's time to verify log data is collected properly and shipped to Elasticsearch. Before doing so we must
first define an index pattern we would like to explore in Kibana. Simply put an index in Elasticsearch is a collection
of related JSON documents. FluentD provides us with those JSON documents and enriches it with Kubernetes fields we can
use.

In Kibana we need to click on the `Discover` icon on the left-hand navigation, this starts the Index creation process.
![Create Index](images/kibana-create-index1.png)
If you take closer a look at the image above (taken days later running the cluster), you will see some available indices. 
You can then match your index pattern based on that, and in our case that would be `logstash-*`.

In the second and last step we need to define the filter time field Kibana will use to filter log data. Choose `@timestamp`
from the drop-down and click on the `Create index pattern` button.
![Create Index Time Filter](images/kibana-create-index2.png)

Click the `Discover` button again, the output should resemble something like this after running some time.
![Discover Index](images/kibana-index-discover.png)

Read the [Kibana Guide](https://www.elastic.co/guide/en/kibana/current/index.html) to learn more about Kibana, includes 
a Quick Start section and other useful information like, how to create dashboards to visualize your log data.

Final step complete! The 3-node Elasticsearch cluster is up and running and integrated with FluentD to capture log data.
We've also configured Kibana with Elasticsearch, so that we can start analyzing/visualizing log data.

## Launch Counter Pods & analyze log data
The real final piece in this setup is capturing our application data. Therefore, we will deploy a small application to 
generate some log data for us to see.

Deploy the counter application to Kubernetes:
```shell
$ kubectl apply -f 6-counter-en.yaml
```
The application should now be generating log data every 5 seconds. To view that data you must click on the `Discover` menu
item and enter the following `kubernetes.pod_name:counter` in the filter field.
![Filtered Application Log Data](images/kibana-application-counter-es-logs.png)

We can now launch multiple applications or scale them and FluentD will be responsible for gathering all the log data
for us. Let's launch one more counter Pod that sends information in another language at another time frequency.
```shell
$ kubectl apply -f 7-counter-en.yaml
```
Go to Kinbana and click on the `Refresh` button to reload the data. To make the information a bit more readable we select
fields: `kubernetes.pod_name` and `log` from the available fields. Modify the filter query to `kubernetes.pod_name:counter*`
to retrieve log data from both running pods. The result should look something like this:
![Multi Counter Pod Log Data](images/kibana-multi-pod-logging.png)




