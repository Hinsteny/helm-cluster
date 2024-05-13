# helm-cluster
demo projects for helm

## Local Test
仅仅在本地进行模板文件生成，而不会安装到K8S集群中去.
```
helm install seata-at ./seata-at --dry-run --debug
```