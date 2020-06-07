### Build base image
```
buildah bud -t idaas/openemrbase .
podman run --rm -dit -p 8080:8080 idaas/openemrbase /bin/sh
podman run  -it -p  8080:8080 idaas/openemrbase /bin/bash
```
### push to remote registry
```
buildah push --tls-verify=false idaas/openemrbase registry.13.67.138.157.nip.io/idaas/openemrbase
```
### Configure secure registries
```
sudo vi /etc/containers/registries.conf
podman search registry.13.67.138.157.nip.io/

```
### Run openemr on k8s
```
kubectl run --image=localhost:32000/idaas/openemrbase openemr-app --port=8080
kubectl exec -it openemr-app -- /bin/bash
```
### Build main image and run 
```
buildah bud -t idaas/openemr:5.0.2 .
podman run --name openemr-app --rm -it -p  8080:8080 idaas/openemr:5.0.2
```

### Start mariadb database 
```
podman run --name openemrdb -p 3306:3306 -e MYSQL_ROOT_PASSWORD=openemr -e MYSQL_DATABASE=openemr -e MYSQL_USER=openemr -e MYSQL_PASSWORD=openemr  -d mariadb:10.5.3-bionic --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

```