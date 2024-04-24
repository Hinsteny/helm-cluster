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

## Build Seata-App Docker Image
we need build four APPs image, include `order`, `storage`, `account` and `business`

### 1、First git clone seata example project from github
```
cd <Your project directory>
git clone git@github.com:apache/incubator-seata-samples.git
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
cp <Your project directory>/incubator-seata-samples/at-sample/dubbo-samples-seata/dubbo-samples-seata-account/target/*.jar ./seata-AT/apps/account/app.jar 
cd seata-AT/apps/account
docker build -t hinsteny.com/seata-test/seata-account:1.0.0 .
kind load docker-image --name seata-at-test hinsteny.com/seata-test/seata-account:1.0.0

-- order
cp <Your project directory>/incubator-seata-samples/at-sample/dubbo-samples-seata/dubbo-samples-seata-order/target/*.jar ./seata-AT/apps/order/app.jar 
cd seata-AT/apps/order
docker build -t hinsteny.com/seata-test/seata-order:1.0.0 .
kind load docker-image --name seata-at-test hinsteny.com/seata-test/seata-order:1.0.0

-- stock
cp <Your project directory>/incubator-seata-samples/at-sample/dubbo-samples-seata/dubbo-samples-seata-stock/target/*.jar ./seata-AT/apps/stock/app.jar 
cd seata-AT/apps/stock
docker build -t hinsteny.com/seata-test/seata-stock:1.0.0 .
kind load docker-image --name seata-at-test hinsteny.com/seata-test/seata-stock:1.0.0

-- business
cp <Your project directory>/incubator-seata-samples/at-sample/dubbo-samples-seata/dubbo-samples-seata-business/target/*.jar ./seata-AT/apps/business/app.jar 
cd seata-AT/apps/business
docker build -t hinsteny.com/seata-test/seata-business:1.0.0 .
kind load docker-image --name seata-at-test hinsteny.com/seata-test/seata-business:1.0.0

```

## Install Hingress for visit http service from the cluster outside
```
--- refer to https://higress.io/zh-cn/docs/ops/deploy-by-helm
helm repo add higress.io https://higress.io/helm-charts

helm install higress higress.io/higress -n higress-system --set global.local=true --create-namespace
```

## Install seata example
```
helm install seata-at-test ./seata-AT -n seata-at-test-ns
--- upgrade
helm upgrade seata-at ./seata-AT
--- delete

```

