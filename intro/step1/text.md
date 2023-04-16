
https://kyverno.io/docs

## Install Kyverno
```plain
helm repo add kyverno https://kyverno.github.io/kyverno
helm repo update
helm install kyverno kyverno/kyverno -n kyverno --create-namespace --set replicaCount=1
kubectl wait deployment -n kyverno kyverno --for condition=Available=True
```{{exec}}

```
k get crd | grep kyverno
```{{exec}}

<br>

## Create Policy
Create a simple policy that requires certain *Pod* labels to be set:
```
cat <<EOF > pod-require-name-label.yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: pod-require-name-label
spec:
  validationFailureAction: Enforce
  background: true
  rules:
  - name: check-for-name-label
    match:
      any:
      - resources:
          kinds:
          - Pod
    exclude:
      any:
      - resources:
          namespaces:
          - kube-system
          - kyverno
    validate:
      message: "label 'app.kubernetes.io/name' is required"
      pattern:
        metadata:
          labels:
            app.kubernetes.io/name: "?*"
EOF
k -f pod-require-name-label.yaml apply
```{{exec}}


## Test
Creating a pod without required labels is not possible
```
sleep 5 # wait till policy was implemented
k run nginx --image=nginx
```{{exec}}
