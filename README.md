# s3-mounter

Mount s3 buckets into pods in k8s.
- Orgin
```
[Here](https://blog.meain.io/2020/mounting-s3-bucket-kube/) is a blog post which explains it in detail.
```

## Changlog

- V1.0 
```
update to mount with custom s3 URL
```

## GUIDE

- Create and apply configmap that store s3 credential
```
vi config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: s3-config
data:
  S3_REGION: ""
  S3_URL: ""
  S3_BUCKET: ""
  AWS_KEY: ""
  AWS_SECRET_KEY: ""
```

- Create and apply daemon-set that mount s3 bucket to host path
```
vi daemonset.yaml

apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  labels:
    app: s3-provider
  name: s3-provider
spec:
  template:
    metadata:
      labels:
        app: s3-provider
    spec:
      containers:
      - name: s3fuse
        image: hungnt99/s3-mounter
        securityContext:
          privileged: true
        envFrom:
        - configMapRef:
            name: s3-config
        volumeMounts:
        - name: devfuse
          mountPath: /dev/fuse
        - name: mntdatas3fs
          mountPath: /var/s3fs:shared
      volumes:
      - name: devfuse
        hostPath:
          path: /dev/fuse
      - name: mntdatas3fs
        hostPath:
          path: /mnt/s3data
```

- Get all ds-set, will show s3-provider ds state
```
kubectl get ds | grep s3-provider


```
- By now, you should have the host system with s3 mounted on /mnt/s3data. You can check that by running the command 
```
kubectl exec -it s3-provider-psp9v -- ls /var/s3fs
```

