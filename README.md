# kubernetes-user-KUBECONFIG

bu repoda belirli bir user a yetki verip kubeconfig file oluşturup, spesifik bir kubeconfig file ile kubernetes e eriştireceğiz.

ilk önce user için crt ve key yaratmamız gerekiyor bunun için openssl kullanacağız. 

```
openssl genrsa -out devopsdude.key 2048
```

key oluşturduktan sonra bu key ile csr oluşturacağız.
```
openssl req -new -key devopsdude.key -out devopsdude.csr -subj "/CN=devopsdude"
```

en sonunda kubernetes clusterımızın ca.crt ve ca.key ile birlikte bu oluşturdugumuz csr i kullanarak crt oluşturacapız. kubernetes ca sertifikaları /etc/kubernetes/pki altında olur.

```
openssl x509 -req -in devopsdude.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out devopsdude.crt
````

yeni oluşturdugumuz sertifika ile userı configfile üzerine ekleyeceğiz.
```
kubectl config set-credentials devopsdude --client-certificate=devopsdude.crt --client-key=devopsdude.key
````

ardından yeni bir contex üzerinde mevcut clusterımızla user ı setleyeceğiz. clustername ini bulmak için "kubectl config get-contexts" derseniz görebilirsiniz. benim isim kind-kind

```
kubectl config set-context devopsdude-context --cluster=kind-kind --user=devopsdude
````

en son yeni açtığımız context e geçip export edeceğiz. 

```
kubectl config use-context devopsdude-context
kubectl config view --minify --flatten > devopsdude.kubeconfig
```

# kubernetes-user-RBAC

cluster-role

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: devopsdude-role
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["nodes", "pods", "replicationcontrollers", "deployments", "statefulsets", "services", "configmaps", "secrets"]
  verbs: ["get", "watch", "list", "create", "update", "patch"]

```

cluster-rolebinding

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: devopsdude-clusterbinding
subjects:
- kind: User
  name: devopsdude
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: devopsdude-role
  apiGroup: rbac.authorization.k8s.io
```
