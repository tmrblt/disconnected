## METHOD-1

### 1. Create a helper pod that mounts your RWX PVC
```
oc run file-transfer --image=registry.redhat.io/rhel8/support-tools --overrides='
{
  "spec": {
    "containers": [{
      "name": "uploader",
      "image": "registry.redhat.io/rhel8/support-tools",
      "command": ["/bin/sh", "-c", "sleep infinity"],
      "volumeMounts": [{"name": "data", "mountPath": "/mnt"}]
    }],
    "volumes": [{"name": "data", "persistentVolumeClaim": {"claimName": "YOUR_RWX_PVC_NAME"}}]
  }
}'
```

### 2. Wait for the pod to be in "Running" state
```
oc wait --for=condition=Ready pod/file-transfer --timeout=60s
```

### 3. Copy the tar file from your local machine/node to the pod
```
oc cp your-file.tar file-transfer:/mnt/
```

### 4. Extract the file inside the PV and set permissions
```
oc exec file-transfer -- sh -c "cd /mnt && tar -xvf your-file.tar && chmod -R 777 ."
```

### 5. (Optional) Verify the files and clean up the tar file to save space
```
oc exec file-transfer -- ls -lh /mnt
oc exec file-transfer -- rm /mnt/your-file.tar
```

## METHOD-2

### 1. Create a helper pod to act as the sync target
```
oc run file-syncer --image=registry.redhat.io/rhel8/support-tools --overrides='
{
  "spec": {
    "containers": [{
      "name": "syncer",
      "image": "registry.redhat.io/rhel8/support-tools",
      "command": ["/bin/sh", "-c", "sleep infinity"],
      "volumeMounts": [{"name": "data", "mountPath": "/mnt"}]
    }],
    "volumes": [{"name": "data", "persistentVolumeClaim": {"claimName": "YOUR_RWX_PVC_NAME"}}]
  }
}'
```

### 2. Wait for the pod to be Ready
```
oc wait --for=condition=Ready pod/file-syncer --timeout=60s
```

### 3. Use oc rsync to transfer the extracted file folder
```
### Ensure your local file folder is already extracted
oc rsync ./local_file_folder/ file-syncer:/mnt/file_destination/ --progress
```

### 4. Set permissions to ensure RHOAI/OpenShift pods can read the weights
```
oc exec file-syncer -- chmod -R 777 /mnt/file_destination/
```

### 5. Verify the transfer
```
oc exec file-syncer -- du -sh /mnt/file_destination/
```
