To simulate machine load

```bash
dd if=/dev/zero of=/dev/null
```

```bash
#create instance template
INSTANCE_TEMPLATE_NAME=test-vm-template-wd
VM_TYPE=f1-micro
IMAGE=debian-9 
IMAGE_PROJECT=debian-cloud
BOOT_DISK_SIZE=10G
ZONE=europe-west3-b

gcloud compute instance-templates create $INSTANCE_TEMPLATE_NAME --m
achine-type $VM_TYPE --image-family $IMAGE --image-project $IMAGE_PROJECT --boot-disk-size $BOOT_DISK_SIZE

# create managed instance group
INSTANCE_GROUP_NAME=instance-vm-group
BASE_INSTANCE_NAME=wd-instance-from-group

gcloud compute instance-groups managed create $INSTANCE_GROUP_NAME -
-base-instance-name $BASE_INSTANCE_NAME --size 1 --template $INSTANCE_TEMPLATE_NAME --zone $ZONE

# enable autoscaling 
gcloud compute instance-groups managed set-autoscaling $INSTANCE_GROUP_NAME --max-num-replicas 10 --target-cpu-utilization 0.50 --zone $ZONE

# delete instance group (with instances!!!)
gcloud compute instance-groups managed delete $INSTANCE_GROUP_NAME --zone $ZONE

# delete instance template
 gcloud compute instance-templates delete $INSTANCE_TEMPLATE_NAME
```

(Autoscaling)[https://cloud.google.com/compute/docs/autoscaler/scaling-cpu-load-balancing]



