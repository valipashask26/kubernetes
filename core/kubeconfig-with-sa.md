## setup kubeconfig with user as SA (token)

### create ServiceAccount

* run below command to create service account
```sh
kubectl create sa <sa-name> -n <namespace>
kubectl create sa admin -n default
```
**Note :** please remember that serviceaccounts are scoped to namespace level.

## Generate the token for SA
In Kubernetes, the automatic creation of tokens for ServiceAccounts when creating them used to be a feature; however, this functionality was removed starting from version 1.24. we have to manually generate it

```sh
kubectl create token <sa-name> -n <namespace> --duration 1h
```
define time in hours

### save the token as secret
Please use the below yaml for saving the token as secret
```sh
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: sa-secret
  namespace: <namespace>
  annotations:
    kubernetes.io/service-account.name: "<sa-name>"
data:
  token: <base64-encoded-token-value>
```
after deploying, edit the service account and add the below field to it.

```yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: admin
    namespace: <namespace>
  secrets:
  - name: <token-secret-name>
```

### Create Role/Clusterrole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
```sh
kubectl apply -f <yaml file name> -n <namespace>
```

### Create Rolebinding/Custerrolebinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: ServiceAccount
  name: admin # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```
**NOTE :** Here we have added serviceAccount in the subject.


### Creating the KubeConfig

Now, the service account got permission to get,watch and list pods in default namespace only as we created role/rolebinding

```yaml
apiVersion: v1
kind: Config
current-context: my-context
clusters:
- name: my-cluster
  cluster:
    certificate-authority-data: <base64-encoded-ca-cert>
    server: https://your-kubernetes-api-server
users:
- name: <sa-name>
  user:
    token: <base64-encoded-token>
contexts:
- name: my-context
  context:
    cluster: my-cluster
    user: <sa-name>

```
In the kubeconfig file:

* **`users`** define user information.
* **`clusters`** define cluster information.
* **`contexts`** mix user and cluster information to create context configurations.

* **`current-context`** specifies the default context to use, which refers to one of the contexts defined in the contexts section.

You can have multiple contexts for different cluster/user combinations, and you choose the context you want to use by setting the current-context field to the name of the desired context.
