# seata-AT-samples
测试一下seata的AT模式

# Steps
## precondition
- Docker
- kubectl
- kind
- helm

## Create one cluster for testing
```
kind create cluster -n seata-at-test

```
## Create one namespace under the testing cluster
```
kubectl create namespace seata-at-test-ns
```

## Install seata example
```
helm install seata-at-test ./seata-at-test -n seata-at-test-ns
```