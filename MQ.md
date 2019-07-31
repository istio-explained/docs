# Setup MQ

## Clone the repo

```
git clone https://github.com/thevoyagerproject/ch04.git
cd ch04/mq
```

## Preparation

```
$ kubectl create namespace stock-trader-mq

# deploy cluster-level pod security policy
$ kubectl apply -f ibm-mq-dev-psp.yaml 

# deploy cluster-level cluster role
$ kubectl apply -f ibm-mq-dev-cr.yaml 

$ kubectl create secret generic mq --from-literal=id=app --from-literal=pwd=mqpassword --from-literal=adminPassword=mqpassword -n stock-trader-mq
```

## Deploy

```
# install MQ chart
$ helm install --name appmsg ibm-charts/ibm-mqadvanced-server-dev \
  --set license=accept \
  --set persistence.enabled=false \
  --set queueManager.name=STQMGR \
  --set queueManager.dev.secret.name=mq \
  --set queueManager.dev.secret.adminPasswordKey=adminPassword \
  --set queueManager.dev.appPassword=ThisIsMyPassword \
  --set nameOverride=stmq --set service.type=NodePort --namespace stock-trader-mq
```

## Setup MQ

```
# exec into MQ pod
$ kubectl exec -it appmsg-stmq-0 /bin/bash -n stock-trader-mq
$ runmqsc
DEFINE QLOCAL (NotificationQ)
SET AUTHREC PROFILE('NotificationQ') OBJTYPE(QUEUE) PRINCIPAL('app') AUTHADD(PUT,GET,INQ,BROWSE)
end

# exit out of the MQ pod
$ exit
```

## Validate MQ is up running, login using the admin pwd using user admin.

The MQ service is exposed using service type NodePort by default.  You can find out the port number that port 9443 maps to.   Example:

```
$ kubectl get services -n stock-trader-mq
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
appmsg-stmq           NodePort    172.21.70.53    <none>        9443:30356/TCP,1414:31181/TCP   2m38s
appmsg-stmq-metrics   ClusterIP   172.21.97.221   <none>        9157/TCP

```

Visit MQ service using NodePort, example: ```https://169.1.1.1:30356/ibmmq/console/login.html```.

Set MQ_ADMIN_PASSWORD:

```
MQ_ADMIN_PASSWORD=$(kubectl get secret --namespace stock-trader-mq mq -o jsonpath="{.data.adminPassword}" | base64 --decode; echo)
```
## Create the secret so that stock trader app can use MQ

```
$ kubectl create secret generic mq --from-literal=id=app \
--from-literal=pwd=$MQ_ADMIN_PASSWORD --from-literal=host=appmsg-stmq  \
--from-literal=port=1414   --from-literal=channel=DEV.APP.SVRCONN \
--from-literal=queue-manager=STQMGR  --from-literal=queue=NotificationQ -n stock-trader

```
