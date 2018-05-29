# Demos

```bash
$ kubectl create ns demo
```

## Running containers

Creating two pods, one running as root the other as user with ID `1234`:

```bash
$ kubectl -n=demo apply -f res/podasroot.yaml
$ kubectl -n=demo apply -f res/podasusr.yaml
$ minikube ssh
$ ps -faux | grep sleep
```

## Authentication & Authorization

Accessing the API from pod:

```bash
$ kubectl run -it --rm apiaccess \
          --restart=Never --image=quay.io/mhausenblas/jump:0.2
ls -al /var/run/secrets/kubernetes.io/serviceaccount/
export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
APISERVERTOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Authorization: Bearer $APISERVERTOKEN"  https://kubernetes.default
```

Create service account:

 ```bash
$ kubectl -n=demo create serviceaccount mysa
```

Run pod with service account and explore contents:

```bash
$ kubectl -n=demo apply -f res/podwithsa.yaml
$ kubectl -n=demo get secret
$ kubectl -n=demo describe secret mysa-token-d6tjw
```

Look into the pod how the service account info has been made available via a volume mount:

```bash
$ kubectl -n=demo  exec -it podwithsa -- sh
cd /var/run/secrets/kubernetes.io/serviceaccount/
cat token
```

Also, check JWT token via https://jwt.io


Next, create a role and rolebinding for being able to view and list pods:

```bash
$ kubectl -n=demo create role podreader --verb=get --verb=list --resource=pods
$ kubectl -n=demo get role/podreader -o=yaml

$ kubectl -n=demo create rolebinding podreaderbinding --role=podreader --serviceaccount=demo:mysa --dry-run=true -o=yaml
```

```bash
$ kubectl -n=demo auth can-i list pods --as=system:serviceaccount:demo:mysa
$ kubectl -n=demo auth can-i create pods --as=system:serviceaccount:demo:mysa 
```
