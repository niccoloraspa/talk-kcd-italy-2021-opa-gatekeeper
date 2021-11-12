# Demo

## Setup

1. Create minikube cluster

```bash
cd demo/
make minikube
```

2. Deploy OPA Gatekeeper and Gatekeeper Policy Manager

```bash
cd manifests
kubectl apply -k .
```

or

```bash
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.5/deploy/gatekeeper.yaml
```

3. Port forward Gatekeeper Policy Manager

```bash
kubectl port-forward svc/gatekeeper-policy-manager -n gatekeeper-system 8080:80 &
```

## OPA Demo

1. Create our first ContraintTemplate

```bash
kubectl apply -f rules/templates/k8srequiredlabels_template.yaml
```

> A `ConstraintTemplate` describes both the Rego that enforces the constraint and the schema of the constraint.

2. Check creation

```bash
kubectl get constrainttemplates.templates.gatekeeper.sh
```

Expected Output:

```bash
NAME                AGE
k8srequiredlabels   15m
```

3. Create a constraint

```bash
kubectl apply -f rules/constraints/all_ns_must_have_kcd_italy_label.yaml
```

> `Constraints` are used to actually enforce a `ConstraintTemplate`

4. Check creation:

```bash
kubectl get constraints -A
```

Expected output:

```bash
NAME                           AGE
ns-must-have-kcd-italy-label   9s
```

5. Test creation of a namespace `opa` without the `kcd-italy` label

```bash
kubectl create ns opa                                     
```

Expected output:

```bash
Error from server ([ns-must-have-kcd-italy-label] you must provide labels: {"kcd-italy"}): admission webhook "validation.gatekeeper.sh" denied the request: [ns-must-have-kcd-italy-label] you must provide labels: {"kcd-italy"}
```

6. Test creation of a namespace `opa` **with** the `kcd-italy` label

```bash
kubectl apply -f rules/tests/ns_opa_good.yaml 
```

Expected output:

```bash
namespace/opa created
```

## Cleanup

1. Delete minikube cluster;

```bash
make delete
```
