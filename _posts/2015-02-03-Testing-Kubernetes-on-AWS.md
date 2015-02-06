---
layout: post
title: Testing Kubernetes on AWS
---

After a lot of digging and figuring things out, I was finally able to get a working Kubernetes cluster running in AWS. This post walks through what I did.

**Disclaimer:** These steps should only be used for kicking the Kubernetes tires and aren't intended for a Production deployment.

We're going to use an EC2 instance as a bastion to provision a 5 node cluster in us-west-2 (Oregon).

###Step 0
Create your bastion instance (preferably with an Amazon Linux AMI) and attach an IAM role that has a Power User policy.

All subsequent steps will be assumed to be run on the bastion you just created.

###Step 1
**Download Kubernetes.** I tested this with release 0.10.0 which was the latest as of this writing but you can change the version to a newer one and it _should_ work.

```
wget https://github.com/GoogleCloudPlatform/kubernetes/releases/download/v0.10.0/kubernetes.tar.gz
tar -xzf kubernetes.tar.gz; cd kubernetes
export PATH=$PATH:$PWD/platforms/linux/amd64
```

###Step 2
Create an EC2 IAM role called "kubernetes" and assign it the Power User policy.

###Step 3
**Turn up the cluster.**

```
export KUBERNETES_PROVIDER=aws
cluster/kube-up.sh
```
The _kube-up.sh_ script will provision a new VPC and a 5 node k8s cluster in us-west-2 (Oregon). It'll also try to create a keypair called "kubernetes" so make sure one doesn't already exist prior to running the script in order to elminate a potential conflict.

Once the cluster is up, it will print the IP address of your cluster, this process takes ~5 minutes.

Create the following 2 variables in order to make running kubecfg and kubectl commands that much easier:

```
export KUBERNETES_MASTER=https://<ip-address>
export PATH=$PATH:$PWD:/cluster
```

The output below is a sample of what you should see:

```
Starting cluster using provider: aws
... calling verify-prereqs
... calling kube-up
Uploading to Amazon S3
+++ Staging server tars to S3 Storage: kubernetes-staging-e893a25f099bxo0eba5465bacc20180f/devel
upload: server/kubernetes-server-linux-amd64.tar.gz to s3://kubernetes-staging-e893a25f099bxo0eba5465bacc20180f/devel/kubernetes-server-linux-amd64.tar.gz
upload: server/kubernetes-salt.tar.gz to s3://kubernetes-staging-e893a25f099bxo0eba5465bacc20180f/devel/kubernetes-salt.tar.gz
{
    "InstanceProfile": {
        "InstanceProfileId": "AIXAI5L9TNVCA8WAYXEV4",
        "Roles": [
            {
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Action": "sts:AssumeRole",
                            "Principal": {
                                "Service": "ec2.amazonaws.com"
                            },
                            "Effect": "Allow",
                            "Sid": ""
                        }
                    ]
                },
                "RoleId": "AR0RJGHJ3NQNCG2X6BGCK",
                "CreateDate": "2015-01-29T00:49:14Z",
                "RoleName": "kubernetes",
                "Path": "/",
                "Arn": "arn:aws:iam::054329697092:role/kubernetes"
            }
        ],
        "CreateDate": "2015-01-29T00:49:14Z",
        "InstanceProfileName": "kubernetes",
        "Path": "/",
        "Arn": "arn:aws:iam::054329697092:instance-profile/kubernetes"
    }
}
Creating vpc.
Adding tag to vpc-8c69c6e9: Name=kubernetes-vpc
Using VPC vpc-8c69c6e9
Creating subnet.
Using subnet subnet-a83df8f1
Creating Internet Gateway.
Using Internet Gateway igw-18ab7e7d
Associating route table.
Configuring route table.
Adding route to route table.
Using Route Table rtb-1606ac73
Creating security group.
Starting Master
Adding tag to i-a21aa7ac: Name=ip-172-20-0-9.us-west-2.compute.internal
Adding tag to i-a21aa7ac: Role=kubernetes-master
Waiting 1 minute for master to be ready
Starting Minion (ip-172-20-0-10.us-west-2.compute.internal)
Adding tag to i-0e1ba600: Name=ip-172-20-0-10.us-west-2.compute.internal
Adding tag to i-0e1ba600: Role=kubernetes-minion
Waiting for minion ip-172-20-0-10.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-10.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-10.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-10.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-10.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-10.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-10.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Minion ip-172-20-0-10.us-west-2.compute.internal running
Starting Minion (ip-172-20-0-11.us-west-2.compute.internal)
Adding tag to i-271ba629: Name=ip-172-20-0-11.us-west-2.compute.internal
Adding tag to i-271ba629: Role=kubernetes-minion
Waiting for minion ip-172-20-0-11.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-11.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-11.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-11.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Minion ip-172-20-0-11.us-west-2.compute.internal running
Starting Minion (ip-172-20-0-12.us-west-2.compute.internal)
Adding tag to i-331ba63d: Name=ip-172-20-0-12.us-west-2.compute.internal
Adding tag to i-331ba63d: Role=kubernetes-minion
Waiting for minion ip-172-20-0-12.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-12.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-12.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-12.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Minion ip-172-20-0-12.us-west-2.compute.internal running
Starting Minion (ip-172-20-0-13.us-west-2.compute.internal)
Adding tag to i-3b1ba635: Name=ip-172-20-0-13.us-west-2.compute.internal
Adding tag to i-3b1ba635: Role=kubernetes-minion
Waiting for minion ip-172-20-0-13.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-13.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-13.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Waiting for minion ip-172-20-0-13.us-west-2.compute.internal to spawn
Sleeping for 3 seconds...
Minion ip-172-20-0-13.us-west-2.compute.internal running
Waiting 3 minutes for cluster to settle
..................Re-running salt highstate
Warning: Permanently added '54.149.145.10' (ECDSA) to the list of known hosts.
Waiting for cluster initialization.

  This will continually check to see if the API for kubernetes is reachable.
  This might loop forever if there was some uncaught error during start
  up.

Kubernetes cluster created.
Sanity checking cluster...

Kubernetes cluster is running.  Access the master at:

  https://admin:tDbAYm9NMqo2gJRe@54.149.145.10

... calling validate-cluster
Running: cluster/../cluster/../cluster/../platforms/linux/amd64/kubectl get minions -o template -t {{range.items}}{{.id}}
{{end}}
Found 4 nodes.
... calling setup-monitoring-firewall
... calling setup-logging-firewall
TODO: setup logging
Done
```
###Cluster Verification
This command will verify that things are working well. It outputs the list of nodes (minions) in the cluster:

```kubecfg list minions```

```
Minion identifier                           Labels
----------                                  ----------
ip-172-20-0-10.us-west-2.compute.internal
ip-172-20-0-11.us-west-2.compute.internal
ip-172-20-0-12.us-west-2.compute.internal
ip-172-20-0-13.us-west-2.compute.internal
```


###Clean Up
Once you're done with the cluster run the command below to clean up all the AWS resources provisioned with the exception of the IAM role and the EC2 keypair, those you'll need to deprovision manually.

``` cluster/kube-down.sh ```

The output will look something like this:

```
Bringing down cluster using provider: aws
TODO: teardown logging
Waiting for instances deleted
Instances not yet terminated: i-0e1ba600 i-3b1ba635 i-331ba63d i-a21aa7ac i-271ba629
Sleeping for 3 seconds...
Instances not yet terminated: i-0e1ba600 i-3b1ba635 i-331ba63d i-a21aa7ac i-271ba629
Sleeping for 3 seconds...
Instances not yet terminated: i-0e1ba600 i-3b1ba635 i-331ba63d i-a21aa7ac i-271ba629
Sleeping for 3 seconds...
Instances not yet terminated: i-0e1ba600 i-3b1ba635 i-331ba63d i-a21aa7ac i-271ba629
Sleeping for 3 seconds...
Instances not yet terminated: i-0e1ba600 i-3b1ba635 i-331ba63d i-a21aa7ac i-271ba629
Sleeping for 3 seconds...
Instances not yet terminated: i-0e1ba600 i-3b1ba635 i-331ba63d i-a21aa7ac i-271ba629
Sleeping for 3 seconds...
Instances not yet terminated: i-0e1ba600 i-3b1ba635 i-331ba63d
Sleeping for 3 seconds...
All instances terminated
Deleting VPC
Done
```
Run the commands below to clean up remnant files on the filesystem.

```
rm -rf ~/.kubecfg.crt
rm -rf ~/.kubecfg.key
rm -rf ~/.kubernetes_auth
rm -rf ~/.kubernetes.ca.crt
rm -rf ~/.kubernetes_ns
rm -rf ~/.ssh/kube_aws_rsa
rm -rf ~/.ssh/kube_aws_rsa.pub
```
