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
kind create cluster -n seata-at-test --config=cluster.conf
---
kind delete cluster -n seata-at-test

```

## Create one namespace under the testing cluster
```
kubectl create namespace seata-at-test-ns
kubectl config set-context --current --namespace=seata-at-test-ns
```

## Build Seata-App Docker Image
we need build four APPs image, include `order`, `storage`, `account` and `business`

### 1、First git clone seata example project from github
```
cd <Your project directory>
--- Because the apache offical example('git clone git@github.com:apache/incubator-seata-samples.git') not support in K8S, so just clone my private repository
git clone git@github.com:InverseLina/incubator-seata-samples.git
```

### 2、Maven package dubbo-samples-seata
```
cd at-sample/springboot-dubbo-seata
mvn clean package -DskipTests
```

### 3、Pull seata-server image and load image to kind node
```
docker pull seataio/seata-server:2.0.0
kind load docker-image --name seata-at-test seataio/seata-server:2.0.0
```

### 4、Copy App jar to helm project directory
``` 
-- account
cp <Your project directory>/incubator-seata-samples/at-sample/dubbo-samples-seata/dubbo-samples-seata-account/target/*.jar ./seata-at/apps/account/app.jar 
cd seata-at/apps/account
docker build -t hinsteny.com/seata-test/seata-account:latest .
kind load docker-image --name seata-at-test hinsteny.com/seata-test/seata-account:latest

-- order
cp <Your project directory>/incubator-seata-samples/at-sample/dubbo-samples-seata/dubbo-samples-seata-order/target/*.jar ./seata-at/apps/order/app.jar 
cd seata-at/apps/order
docker build -t hinsteny.com/seata-test/seata-order:latest .
kind load docker-image --name seata-at-test hinsteny.com/seata-test/seata-order:latest

-- stock
cp <Your project directory>/incubator-seata-samples/at-sample/dubbo-samples-seata/dubbo-samples-seata-stock/target/*.jar ./seata-at/apps/stock/app.jar 
cd seata-at/apps/stock
docker build -t hinsteny.com/seata-test/seata-stock:latest .
kind load docker-image --name seata-at-test hinsteny.com/seata-test/seata-stock:latest

-- business
cp <Your project directory>/incubator-seata-samples/at-sample/dubbo-samples-seata/dubbo-samples-seata-business/target/*.jar ./seata-at/apps/business/app.jar 
cd seata-at/apps/business
docker build -t hinsteny.com/seata-test/seata-business:latest .
kind load docker-image --name seata-at-test hinsteny.com/seata-test/seata-business:latest

```

## Install Hingress for visit http service from the cluster outside
```
--- refer to https://higress.io/zh-cn/docs/ops/deploy-by-helm
helm repo add higress.io https://higress.io/helm-charts
helm install higress -n higress-system higress.io/higress --create-namespace --render-subchart-notes --set global.local=true --set higress-console.o11y.enabled=false
```

## Install seata example
```
helm install seata-at ./seata-at -n seata-at-test-ns
--- upgrade
helm upgrade seata-at ./seata-at
--- local install test
helm install seata-at ./seata-at --dry-run --debug
--- delete
helm uninstall seata-at
```

## Test seata-at
```
--- commit success
curl "http://127.0.0.1:80/test/commit?userId=1&commodityCode=1&orderCount=1" -H 'host: seata-at.local'
--- commit rollback when occured an exception
curl "http://127.0.0.1:80/test/rollback?userId=1&commodityCode=1&orderCount=1" -H 'host: seata-at.local'
```