## Dynamic updates in Configmap

Let's practice this in your Kubernetes cluster. We'll create a ConfigMap, mount it as a volume in a Pod, update the ConfigMap, and verify that the new value appears inside the running container without restarting the Pod.

The key concept is: ConfigMap volume mount without `subPath` → Kubernetes can update the mounted files eventually. The update is not necessarily instantaneous, and an application may need to reread the file to use the new value.

## Step 1: Create a ConfigMap

```
kubectl create configmap dynamic-demo \
  --from-literal=APP_MODE=blue
```

Verify it:

```
kubectl get configmap dynamic-demo -o yaml
```

Initially, the value is `blue`.

## Step 2: Create a Pod with a volume mount

Run:

```
kubectl run config-demo \
  --image=busybox:1.36 \
  --restart=Never \
  --command -- sh -c 'sleep 36000'
```

Mounting a ConfigMap requires a Pod volume, so create the Pod using this YAML instead if you haven't created it yet. If the previous command already created it, delete that Pod first with `kubectl delete pod config-demo`.

```
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: config-demo
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "sleep 36000"]
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: dynamic-demo
EOF
```

Wait until the Pod is running:

```
kubectl get pod config-demo
```

## Step 3: Read the mounted value

```
kubectl exec config-demo -- cat /etc/config/APP_MODE
```

Expected output:

```
blue
```

## Step 4: Update the ConfigMap

Change `blue` to `green`:

```
kubectl create configmap dynamic-demo \
  --from-literal=APP_MODE=green \
  --dry-run=client -o yaml | kubectl apply -f -
```

Verify the ConfigMap itself:

```
kubectl get configmap dynamic-demo -o yaml
```

## Step 5: Check the running Pod again

Kubernetes needs time to propagate the update. Wait around 30–60 seconds, then run:

```
kubectl exec config-demo -- cat /etc/config/APP_MODE
```

Expected output:

```
green
```

Notice that we didn't restart the Pod!

## Step 6: Understand why it works

ConfigMap updated

APP_MODE = green

Kubernetes propagates the change

Mounted file updated

/etc/config/APP_MODE = green

Important for CKA: If you mount a ConfigMap or Secret using `subPath`, updates are not propagated to that mount. With a normal volume mount, updates are eventually reflected, but the application must reread the file to use the new value. Environment variables sourced from ConfigMaps or Secrets also do not update inside an already-running container.

## Cleanup after practice

```
kubectl delete pod config-demo
kubectl delete configmap dynamic-demo
```
