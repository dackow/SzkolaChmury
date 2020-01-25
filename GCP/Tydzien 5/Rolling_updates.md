# Rolling updates

```bash

# crate an instance temaplate with debian 9
IMAGE1=debian-9
INSTANCE_TEMPLATE_NAME1=test-vm-template-wd1

gcloud compute instance-templates create $INSTANCE_TEMPLATE_NAME1 --machine-type $VM_TYPE --image-family $IMAGE1 --image-project $IMAGE_PROJECT

# crate an instance temaplate with debian 10
IMAGE2=debian-10
INSTANCE_TEMPLATE_NAME2=test-vm-template-wd2

gcloud compute instance-templates create $INSTANCE_TEMPLATE_NAME2 --machine-type $VM_TYPE --image-family $IMAGE2 --image-project $IMAGE_PROJECT

# verify instance templates
gcloud compute instance-templates list

# create an instance group based on template 1 
INSTANCE_GROUP_NAME=instance-vm-group

gcloud compute instance-groups managed create $INSTANCE_GROUP_NAME --base-instance-name=$BASE_INSTANCE_NAME --template $INSTANCE_TEMPLATE_NAME1 --size 3 --zone $ZONE

# update instances group with template 2
gcloud compute instance-groups managed rolling-action start-update $INSTANCE_GROUP_NAME --version template=$INSTANCE_TEMPLATE_NAME2 --max-unavailable 2 --zone $ZONE

# delete regources
gcloud compute instance-groups managed delete $INSTANCE_GROUP_NAME --zone $ZONE
```
