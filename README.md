Pointing to the management cluster apply CRDs, addon-controller and the sveltoscluster-controller

```
kubectl apply -f resources
kubectl apply -f resources/addon-controller/
kubectl apply -f resources/sveltoscluster-manager
```

To register a cluster (in either pull or push mode) sveltosctl is of help.

```
git clone git@github.com:projectsveltos/sveltosctl.git
cd sveltoctl
make build
```

The binary will be in bin.

To register a cluster in pull mode, from the sveltosctl directory

```
./bin/sveltosctl register cluster --namespace <CLUSTER NAMESPACE> --cluster <CLUSTER NAME> --pullmode --labels=env=fv > /tmp/applier
```

This will create a SveltosCluster instance and set it to pull mode.
The output of this command is what you need to apply to you **managed** cluster.

You can verify all is good by running following command while pointing to your **management** cluster

```
kubectl get sveltoscluster -A
```

Then while pointing to your **managed** cluster

```
KUBECONFIG=<YOUR MANAGED CLUSTER KUBECONFIG> kubectl apply -f /tmp/applier
```

As soon as the agent is up and running into your managed cluster, while pointing to your **management** cluster

```
kubectl get sveltoscluster -A
```

and at this point cluster should be marked ready and the managed cluster kubernetes version deployed.

## Deploy ClusterProfiles

Following will deploy Kyverno helm chart, Kyverno `disallow_latest_tag` ClusterPolicy and nginx Helm chart

```
kubectl apply -f clusterprofiles
```

nginx is deployed with SyncMode set to **ContinuousWithDriftDetection** so if for instance while pointing to the managed cluster, the nginx deployment is deleted, Sveltos will react by putting it back

```
KUBECONFIG=<YOUR MANAGED CLUSTER KUBECONFIG> kubectl delete deployment -n nginx                nginx-latest-nginx-ingress-controller
```

Verify Sveltos puts it back

```
watch KUBECONFIG=<YOUR MANAGED CLUSTER KUBECONFIG> kubectl get deployment -n nginx                nginx-latest-nginx-ingress-controller
```


## Register a cluster in push mode

Clusters can also registered in push mode (managed cluster apiserver must be reachable). For

```
./bin/sveltosctl register cluster --namespace <CLUSTER NAMESPACE> --cluster <CLUSTER NAME> --labels=env=fv --kubeconfig=<FILE with MANAGED CLUSTER KUBECONFIG>
```