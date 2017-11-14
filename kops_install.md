## Install K8S cluster on AWS using `kops` and Deploy the Voting App

Useful Links:

* Install needed CLI tools (kops, kubectl, aws) on local dev VM:  https://github.com/kubernetes/kops/blob/master/docs/install.md
* Create cluster:
    * kubernetes-made instructions:  https://github.com/kubernetes/kops/blob/master/docs/aws.md 
    * amazon-made instructions:  https://aws.amazon.com/blogs/compute/kubernetes-clusters-aws-kops/ 

Notes:

* The kops IAM user is already created:  see PN for this user’s aws secret access key.
* Use a gossip based cluster to avoid setting up DNS.  This requires the cluster name to end in `.k8s.local`. Cluster name must also be a fully-qualified DNS name, e.g.:  `k8s-sq.k8s.local`
* The s3 bucket for cluster state storage is already created in us-east-1:  `skopos-io-k8s-state-store` 
* The `kops create cluster` arguments `--master-count` and `--node-count` are happily eaten, but at least the node count does not work.  You will need to edit the node instance group (from s3) after creating the cluster configuration if you don’t like the default 1-master and 2-nodes.  An example of command to use, as seen below, is:  `kops edit ig --name=k8s-sq.k8s.local nodes`.  This command is shown as part of the output of the cluster config creation.

### Quick Start

Install kops:
```
wget -O kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/
```

Install kubctl:
```
wget -O kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

Install aws cli:  `pip install awscli`

Configure aws client:

* requires kops iam user aws access key id and secret access key (see PN)
* sq used:  default region `us-west-2` (oregon) and output format `json`
* command to configure the aws client:  `aws configure`  (verify with `aws iam list-users`)

Prepare for cluster config creation - export env vars:
```
export AWS_ACCESS_KEY_ID=<val>
export AWS_SECRET_ACCESS_KEY=<val>
```

Create cluster config (in s3 bucket):
```
kops create cluster \
     --cloud aws --zones us-west-2a \
     --name k8s-sq.k8s.local --state s3://skopos-io-k8s-state-store \
     --master-count 1 --node-count 0 --master-size t2.medium > create.log 2>&1
```
**note**: the `--node-count` arg is apprently non-effective

Examine/edit the resulting configs:

* list clusters with: `kops get cluster`
* edit this cluster with: `kops edit cluster k8s-sq.k8s.local`
* edit node instance group (**e.g., to change node count**): `kops edit ig --name=k8s-sq.k8s.local nodes`
* edit master instance group: `kops edit ig --name=k8s-sq.k8s.local master-us-west-2`

Build (create/deploy) the cluster:  `kops update cluster k8s-sq.k8s.local --yes`

Verify the cluster:

* `kubectl get nodes` (verify K8S simple api call)
* `kubectl cluster-info`
* `kops validate cluster --state s3://skopos-io-k8s-state-store` (verify cluster working)
* `kubectl -n kube-system get po` (list system components)

Untaint the master node (get the node name and view the existing taint using `kubectl describe nodes`) so that objects may be deployed to it (if desired, or if creating a single node K8S cluster):
```
kubectl taint node ip-172-20-32-20.us-west-2.compute.internal node-role.kubernetes.io/master:NoSchedule-
```

Deploy the voting app:

* `wget https://raw.githubusercontent.com/opsani/k8s-demo/master/app.yaml`
* Edit app.yaml and change `v1beta2` to `v1beta1`
* Create the deployment:  `kubectl create -f app.yaml`

On AWS this will create a classic LB for each externally exposed K8S service (vote/result).  Note:

* LBs must have K8S master/node instance(s) added manually (before then, you get an empty result from the LB ingress)
* K8S cluster must have a security group which provides access to EACH LB on the service external port -- whatever is assigned by K8S, e.g. `8080:32216/TCP`

### Update K8S Cluster using `kops`

See:  https://github.com/kubernetes/kops/blob/master/docs/upgrade.md

Tested:
```
# edit the kops cluster spec and set "kubernetesVersion: 1.8.3"
kops edit cluster k8s-sq.k8s.local --state s3://skopos-io-k8s-state-store

# preview upgrade
kops update cluster k8s-sq.k8s.local --state s3://skopos-io-k8s-state-store

# perform upgrade
kops update cluster k8s-sq.k8s.local --state s3://skopos-io-k8s-state-store --yes

kops rolling-update cluster k8s-sq.k8s.local --state s3://skopos-io-k8s-state-store

kops rolling-update cluster k8s-sq.k8s.local --state s3://skopos-io-k8s-state-store --yes

# As desired, untaint the (new) master node
# note: rolling-update replaces this VM - you may need to add any custom
# security group from the previous node(s) to the new node(s)
kubectl taint node ip-172-20-51-0.us-west-2.compute.internal node-role.kubernetes.io/master:NoSchedule-
```
