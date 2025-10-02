# cka-study


kubectl set image deployment/frontend simple-webapp=kodekloud/webapp-color:v2

## Enviroment variables

```
kubectl run webapp-color --image=kodekloud/webapp-color --labels=name=webapp-color --env=APP_COLOR=green --dry-run=client -o yaml > app.yaml
```

```
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard
```

## Enviroment variables using secrets

```
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123 --dry-run=client -o yaml > db-secret.yaml
```

```
kubectl run webapp-pod --image=kodekloud/simple-webapp-mysql  --dry-run=client -o yaml > app.yaml
```

```
kubectl autoscale deploy nginx-deployment --max=3 --cpu-percent=80
```


## Cluster maintainance

### Backup etcd

- Backup command
```
ETCDCTL_API=3 etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/snapshot-pre-boot.db
```

- check backup command

```
etcdctl snapshot status /opt/snapshot-pre-boot.db \
--write-out=table
```
### Restore
- Stop kube-apiserver (if necessary)

- Restore command
```
etcdctl snapshot restore /opt/snapshot-pre-boot.db –data-dir /var/lib/restored
```

- Change data dir etcd config

  —-data-dir=/var/lib/restored

- Change volume path on etcd config




## Certificates

### View content inside kubernetes certificates

- API server certificate
```
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```
- ETCD server certificate
```
openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text -noout
```
- CA root certificate
```
openssl x509 -in /etc/kubernetes/pki/ca.crt -text -noout
```

Creating Certificate Signing Request

1-User create a key

```
openssl genrsa -out jane.key 2048
```

2-User creates a certificate signing request

```
openssl req -new -key jane.key -sunj “/CN=jane” -o
```

3-Get certificate generated on step 2 converted to base64

```
cat myuser.csr | base64 | tr -d "\n"
```
4- Get base64 certificate date and create the certificate signing request on kubernetes file

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser # example
spec:
  # This is an encoded CSR. Change this to the base64-encoded contents of myuser.csr
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
```

5- get status of certificate signing request

```
kubectl get csr
```

5- approve or deny certificate signing request

```
kubectl certificate approve myuser
```
```
kubectl certificate deny myuser
```

