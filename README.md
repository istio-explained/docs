# Documentation for the project

## [Deploy the DB2 service](DB2.md)

## Deploy the portfolio service:
1. create required secret

```
$ kubectl create secret generic jwt -n stock-trader --from-literal=audience=stock-trader --from-literal=issuer=http://stock-trader.ibm.com
```

2. deploy

```
$ git clone https://github.com/thevoyagerproject/portfolio.git
$ kubectl apply -f portfolio/manifests/deploy.yaml -n stock-trader

# validate pod has reached running status
$ kubectl get pods -l app=portfolio -n stock-trader

# view logs to validate no errors
$ kubectl logs -c portfolio --namespace=stock-trader --selector="app=portfolio,solution=stock-trader"
```

## Deploy the trader service:

```
$ git clone https://github.com/thevoyagerproject/trader.git
$ kubectl apply -f trader/manifests/deploy.yaml -n stock-trader
```

## Deploy the stock-quote service:

The stock-quote service reaches out to [IEX Cloud](https://iexcloud.io/) to obtain stock prices.  

Optional: If you want to use your own API token, you may [register](https://iexcloud.io/cloud-login#/register) with IEX Cloud.  After you register and [login to IEX cloud](https://iexcloud.io/cloud-login), you can view your API token.

```
# replace $YOUR_IEX_API_TOKEN with the valye of your API token
kubectl create secret generic iex -n stock-trader --from-literal=iex-api-key=$YOUR_IEX_API_TOKEN
```

Deploy the stock-quote service.

```
$ git clone https://github.com/thevoyagerproject/stock-quote.git
$ kubectl apply -f stock-quote/manifests/deploy.yaml -n stock-trader
```

## Visit the trader application:
To visit the trader-service via NodePort, find the worker public ip and visit ```https://$WORKER_PUBLIC_IP:32389/trader/``` and login using stock/trader.
