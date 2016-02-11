---
layout: post
title: Adding Resiliency to DTR 1.4.2
---

For anyone using Docker Trusted Registry 1.4.x you know that resiliency isn't built in. There's no native HA so it's up to you to define and manage that capability. I was eager to delivery an internal registry but there's no way I could do that without some level of resiliency.

## Resiliency Areas of Concern
DTR's data persistence are spread across a few areas, namely:
1) Image store
2) Metadata store
3) Config store

### Image store
DTR supports S3, Azure, Swift and Local storage backends for the container image store. In my case I use S3 which is already durable to eleven 9s but additionally I'm relying on cross region replication to increase my recoverability at the local bucket and region level.

### Metadata store
DTR generates some metadata like user and team membership as well as repository and organization names. This is all stored in a Postgres container that's part of the DTR stack. If you lose the host and consequently the DTR containers then you've lost the Postgres metadata and that can't be easily recreated.

### Config store
When you install DTR there's some configuration that's typically done from the GUI on first launch:
* Adding the license
* Configuring the hostname settings
* Storage configuration
* Authentication configuration
* etc

...all this would need to be reconfigured during an instance failure however this information is all stored in /var/local/dtr and /usr/local/etc/dtr

## Resiliency Solution
To address all these concerns I've used AWS CloudFormation and Auto Scaling Groups to autonomously instantiate a new DTR node on failure and configure it as well as restore from a backup taken within 15 minutes of the failure.

The caveat is that there's still downtime, but that downtime is the time it takes for a new EC2 instance to bootstrap which is ~7 minutes.

The CFN template is [here](https://github.com/alanwill/cfn-dtr).

### Prerequisites
1) Manually provision a DTR stack and configure it completely. Once configured tar up the /var/local/dtr and /usr/local/etc/dtr directories then upload them to S3 so they can be pull during the EC2 bootstrap process

2) Upload the dtr-pg-backup.sh and dtr-pg-restore.sh files to S3 as well. These will also be downloaded during bootstrap and used to initiate the initial Postgres restore as well as load the cronjob which will initiate a backup every 15 minutes that's stored in S3. NOTE: For this to work successfully you'll need to have at least 1 backup already accessible from S3.

More details are in the GitHub repo. This solution has worked great for us as a completely handsfree DTR 1.4.x service.
