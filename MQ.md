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

$ kubectl create secret generic mq --from-literal=id=app --from-literal=pwd=mqpassword --from-literal=adminPassword=mqpw4me -n stock-trader-mq
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

```
# visit MQ service using NodePort, example:
https://169.60.120.59:32460/ibmmq/console/login.html
```

## Consume this from stock-trader

```
# create mq secret
$ kubectl create secret generic mq --from-literal=id=app --from-literal=pwd=mqpw4me --from-literal=adminPassword=<pwd> -n stock-trader
```
