# Demos

```bash
$ kubectl create ns demo
```

## Running containers

 security context https://sysdig.com/blog/kubernetes-security-psp-network-policy/

```bash
$ kubectl -n=demo apply res/podasroot.yaml
```

`ps` from inside and outside


```bash
$ kubectl -n=demo apply res/podasusr.yaml
```

## Authentication & Authorization

Accessing the API from pod:

```bash
$ kubectl -n=demo run -it --rm apiaccess \
          --restart=Never --image=quay.io/mhausenblas/jump:0.2
export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
APISERVERTOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Authorization: Bearer $APISERVERTOKEN"  https://kubernetes.default
```

Create SA, explore (incl. secret and jump into pod)

 ```bash
$ kubectl -n=demo create serviceaccount mysa
```

Run pod with SA:

```bash
$ kubectl -n=demo apply
$ kubectl -n=demo get secret
```

```bash
$ kubectl -n=demo  exec -it podwithsa -- sh
cd /var/run/secrets/kubernetes.io/serviceaccount/
```

Role/rolebinding:

```bash
$ kubectl -n=demo create role podreader --verb=get --verb=list --resource=pods
$ kubectl -n=demo create rolebinding podreaderbinding --role=podreader --serviceaccount=demo:mysa --dry-run=true -o=yaml
```

```bash
$ kubectl -n=demo auth can-i list pods --as=system:serviceaccount:demo:mysa 
```
