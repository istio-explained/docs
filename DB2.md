# Deploy DB2 service

## Deploy the DB2 chart

1. Subscribe to [Db2 Developer-C Editon](https://hub.docker.com/_/db2-developer-c-edition) on Docker Store to access the DB2 Developer-C Edition image.

2. Init Helm if needed.

```
$ cat <<EOF | kubectl create -f -
# Create a service account for Helm and grant the cluster admin role.
# It is assumed that helm should be installed with this service account
# (tiller).
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: tiller
  namespace: kube-system
EOF

$ helm init --service-account tille
```

3. Install the IBM DB2 helm chart.

```
# create the stock-trader-data namespace
$ kubectl create namespace stock-trader-data

# clone the repo
$ git clone https://github.com/IBM/charts
$ cd charts/stable/ibm-db2oltp-dev/ibm_cloud_pak/pak_extensions/prereqs

# Setup required security context
$ ./createNSandSecurity.sh --namespace stock-trader-data
$ kubectl create secret docker-registry mydb2secret --docker-username=linsun --docker-password=<pwd> --docker-email=<email> --namespace=stock-trader-data

# Update serviceaccount with correct secret
$ kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "mydb2secret"}]}' --namespace=stock-trader-data

# install DB2.  Use ibmc-block-gold storage class for IKS
$ helm install --name stocktrader-db2 ibm-charts/ibm-db2oltp-dev \
--set db2inst.instname=db2inst1 \
--set db2inst.password=ThisIsMyPassword \
--set options.databaseName=trader \
--set persistence.useDynamicProvisioning=true \
--set dataVolume.storageClassName=ibmc-block-gold \
--set etcdVolume.storageClassName=ibmc-block-gold \
--set dataVolume.size=2Gi  --namespace=stock-trader-data
```

4. Check DB2 services, statefulset and pod's readiness in stock-trader-data namespace.  Examples:
```
$ kubectl get all -n stock-trader-data
NAME                               READY   STATUS    RESTARTS   AGE
pod/stocktrade-ibm-db2oltp-dev-0   1/1     Running   0          35d

NAME                                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                   AGE
service/stocktrade-ibm-db2oltp-dev       ClusterIP   None            <none>        50000/TCP,55000/TCP,60006/TCP,60007/TCP   35d
service/stocktrade-ibm-db2oltp-dev-db2   NodePort    172.21.153.68   <none>        50000:30882/TCP,55000:30926/TCP           35d

NAME                                          READY   AGE
statefulset.apps/stocktrade-ibm-db2oltp-dev   1/1     35d
```

