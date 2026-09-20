```bash
kubectl run my-pod -l app=nginx --image=nginx
kubectl get pods --show-labels
kubectl get pods -l app=nginx

kubectl scale rc nginx-rc --replicas=4
kubectl get rc -o wide
kubectl get rc nginx-rc -o yaml
```

- you can see last updated replicas after scaling replicas

<img width="1845" height="517" alt="image" src="https://github.com/user-attachments/assets/c2c72f75-33ab-4c48-ab15-6f0459b2d269" />

```bash
kubectl set image rc nginx-rc nginx-container=nginx:1.22
```
<img width="1091" height="451" alt="image" src="https://github.com/user-attachments/assets/c3838a44-ea1b-4e3a-8f1d-b60253425f60" />


```bash
kubectl edit rc nginx-rc
```

<img width="842" height="110" alt="image" src="https://github.com/user-attachments/assets/4d642ffa-14a5-4eea-b807-29c4f3975e56" />

