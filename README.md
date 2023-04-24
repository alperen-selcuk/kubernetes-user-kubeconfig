# kubernetes-user-KUBECONFIG

bu repoda belirli bir user a yetki verip kubeconfig file oluşturup, spesifik bir kubeconfig file ile kubernetes e eriştireceğiz.

ilk önce user için crt ve key yaratmamız gerekiyor bunun için openssl kullanacağız. 

```
openssl genrsa -out devopsdude.key 2048
```

key oluşturduktan sonra bu key ile csr oluşturacağız.
```
openssl req -new -key devopsdude.key -out devopsdude.csr -subj "/CN=user"
```

en sonunda kubernetes clusterımızın ca.crt ve ca.key ile birlikte bu oluşturdugumuz csr i kullanarak crt oluşturacapız. kubernetes ca sertifikaları /etc/kubernetes/pki altında olur.

```
openssl x509 -req -in devopsdude.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out devopsdude.crt
````

yeni oluşturdugumuz sertifika ile userı configfile üzerine ekleyeceğiz.
```
kubectl config set-credentials devopsdude --client-certificate=devopsdude.crt --client-key=devopsdude.key
````

ardından contex üzerinde mevcut clusterımızla user ı setleyeceğiz. clustername ini bulmak için "kubectl config get-contexts" derseniz görebilirsiniz. benim isim kind-kind

```
kubectl config set-context user-context --cluster=kind-kind --user=devopsdude
````

en son bu file ı ayıracağız. 

```
kubectl config view --minify --flatten > devopsdude.kubeconfig
```

# kubernetes-user-RBAC
