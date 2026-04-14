### 3.2 Mirror Images 'oc mirror --v2' 

#1. Source:
https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/disconnected_environments/index#about-installing-oc-mirror-v2


> **Note:** Login to Images Node

#1. Install Packages

```
sudo yum install -y podman
sudo yum install -y wget
#sudo yum module install -y container-tools
sudo yum install -y container-tools
sudo yum install -y firewalld
sudo systemctl enable firewalld --now
```


#1. Create Directories

```
WORKDIR=~/mirror
mkdir -p $WORKDIR/downloads
mkdir -p $WORKDIR/files
mkdir -p $WORKDIR/images-v4-20-10
```

#1. Download and extract oc and oc mirror

```
cd $WORKDIR/downloads
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.20.10/openshift-client-linux-4.20.10.tar.gz

wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/oc-mirror.tar.gz

tar -zxvf openshift-client-linux-4.20.10.tar.gz
tar -zxvf oc-mirror.tar.gz 

chmod +x oc-mirror
sudo mv oc kubectl oc-mirror /usr/local/bin/
rm -rf README.md

cd $WORKDIR
```

#1. Paste your **PULL-SECRET**

```
#Paste your PULL-SECRET
mkdir -v $HOME/.docker
vi $HOME/.docker/config.json
```

#1. Copy pull-secret

```
mkdir -p $XDG_RUNTIME_DIR/containers
cp $HOME/.docker/config.json $XDG_RUNTIME_DIR/containers/auth.json
```
#1. Create imageset-config yaml

```
vi $WORKDIR/files/imageset-config-v4-20-10.yaml
```

```yaml
---
kind: ImageSetConfiguration
apiVersion: mirror.openshift.io/v2alpha1
mirror:
  platform:
    channels:
    - name: stable-4.20 
      minVersion: 4.20.10
      maxVersion: 4.20.10
    graph: true 
  operators:
    - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.20
      packages:
        - name: kubernetes-nmstate-operator
        - name: kubevirt-hyperconverged   
        - name: local-storage-operator    
        - name: node-healthcheck-operator 
        - name: node-maintenance-operator 
        - name: self-node-remediation
        - name: fence-agents-remediation
        - name: web-terminal      
        - name: devworkspace-operator         
        - name: node-observability-operator
        - name: mtv-operator
          channels:
            - name: release-v2.10          
        - name: ocs-client-operator
          channels:
            - name: stable-4.20          
        - name: ocs-operator
          channels:
            - name: stable-4.20                 
        - name: odf-csi-addons-operator
          channels:
            - name: stable-4.20      
        - name: odf-dependencies
          channels:
            - name: stable-4.20             
        - name: odf-multicluster-orchestrator
          channels:
            - name: stable-4.20
        - name: odf-operator
          channels:
            - name: stable-4.20                 
        - name: odf-prometheus-operator
          channels:
            - name: stable-4.20      
        - name: rook-ceph-operator
          channels:
            - name: stable-4.20      
        - name: recipe
          channels:
            - name: stable-4.20      
        - name: mcg-operator
          channels:
            - name: stable-4.20      
        - name: cephcsi-operator
          channels:
            - name: stable-4.20      
        - name: odr-hub-operator
          channels:
            - name: stable-4.20      
        - name: odr-cluster-operator
          channels:
            - name: stable-4.20
  additionalImages: 
   - name: registry.redhat.io/ubi8/ubi:latest
   - name: registry.redhat.io/ubi9/ubi:latest
```

#1. You can directly mirror to quay or mirror to disk then disk to quay.

> **2 options:**
> 1. Mirror2Disk > DiskTransfer > Disk2Registry
> 2. Directly Mirror2Registry

```bash

#1a. Mirror to disk
oc-mirror --v2 --config $WORKDIR/files/imageset-config-v4-20-10.yaml file://$WORKDIR/images-v4-20-10  --parallel-images 2 --parallel-layers 2 --image-timeout 30m

#1b. Disk transfer
rsync -avP $WORKDIR/mirror 10.131.0.224:/home/admin

#1c. Disk to registry
oc mirror -c $WORKDIR/files/imageset-config-v4-20-10.yaml --from file:///$WORKDIR/images-v4-20-10 docker://ec2-3-140-253-230.us-east-2.compute.amazonaws.com:5002 --v2
```

```
#2. Mirror to registry directly
oc-mirror --v2 --config $WORKDIR/files/imageset-config-v4-20-10.yaml --workspace file://$WORKDIR/images-v4-20-10  docker://ec2-3-140-253-230.us-east-2.compute.amazonaws.com:5002 --parallel-images 2 --parallel-layers 2 --image-timeout 30m
