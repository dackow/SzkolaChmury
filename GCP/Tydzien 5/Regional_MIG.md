# Regional Managed Instance Groups (MIG)

In zonal managed instance group all our machines are places in the same zone.
In regional MIG our app is spread across multiple zones within a single region.
What for? To inscrease availability,

In regions with more than 3 zones, our app will be spread to 3 zones (by default), but we can configure to which zones the app shall go and how many zones our app shall go to (e.g. 2 or 4 zones).


```bash

#consts
INSTANCE_TEMPLATE_NAME=test-vm-template-wd
VM_TYPE=f1-micro
IMAGE=debian-9
IMAGE_PROJECT=debian-cloud
BOOT_DISK_SIZE=10G
ZONE=europe-west3-b
REGION=europe-west3
INSTANCE_GROUP_NAME=wd-instance-group
BASE_INSTANCE_NAME=wd-single-instance

# create a template
gcloud compute instance-templates create $INSTANCE_TEMPLATE_NAME --machine-type $VM_TYPE --image-family $IMAGE --image-project $IMAGE_PROJECT

# case 1: create managed regional instance group - then GCP will distribute thje same number of instances to each available zone
gcloud compute instance-groups managed create $INSTANCE_GROUP_NAME --template $INSTANCE_TEMPLATE_NAME --base-instance-name $BASE_INSTANCE_NAME --size 3 --region $REGION

# delete created regional MIG
gcloud compute instance-groups managed delete $INSTANCE_GROUP_NAME --region $REGION

# case 2: create regional MIG in specified zones
ZONE1=europe-west3-c
ZONE2=europe-west3-b

gcloud compute instance-groups managed create $INSTANCE_GROUP_NAME --template $INSTANCE_TEMPLATE_NAME --base-instance-name $BASE_INSTANCE_NAME --size 3 --zones $ZONE1,$ZONE2

# delete created regional MIG
gcloud compute instance-groups managed delete $INSTANCE_GROUP_NAME --region $REGION

# case 3: without instance redistribution
gcloud compute instance-groups managed create $INSTANCE_GROUP_NAME
 --template $INSTANCE_TEMPLATE_NAME --base-instance-name $BASE_INSTANCE_NAME --size 3 --zones $ZONE1,$ZONE2 --instance-redistribution-type NONE
```
(To read)[https://cloud.google.com/compute/docs/instance-groups/distributing-instances-with-regional-instance-groups#proactive_instance_redistribution]
