# Canary release

Częściowe wgrywanie nowej wersji i udostępnienie jej tylko części użytkowników (po testowaniu na lokalnych środowiskach).

```bash
IMAGE_PROJECT=debian-cloud
VM_TYPE=f1-micro
INSTANCE_GROUP_NAME=instance-vm-group
IMAGE1=debian-9
INSTANCE_TEMPLATE_NAME1=test-vm-template-wd1
INSTANCE_TEMPLATE_NAME2=test-vm-template-wd2

# crate an instance temaplate with debian 9 with version 1 of our applcation

gcloud compute instance-templates create $INSTANCE_TEMPLATE_NAME1 \
--image-family $IMAGE1 \
--image-project $IMAGE_PROJECT \
--tags=http-server \
--machine-type=$VM_TYPE \
--metadata=startup-script=\#\!/bin/bash$'\n'sudo\ apt-get\ update\ $'\n'sudo\ apt-get\ install\ -y\ nginx\ $'\n'sudo\ service\ nginx\ start\ $'\n'sudo\ sed\ -i\ --\ \"s/Welcome\ to\ nginx/Version:1\ -\ Welcome\ to\ \$HOSTNAME/g\"\ /var/www/html/index.nginx-debian.html

# crate an instance temaplate with debian 9 with version 1 of our applcation

gcloud compute instance-templates create $INSTANCE_TEMPLATE_NAME2 \
--image-family $IMAGE1 \
--image-project $IMAGE_PROJECT \
--tags=http-server \
--machine-type=$VM_TYPE \
--metadata=startup-script=\#\!/bin/bash$'\n'sudo\ apt-get\ update\ $'\n'sudo\ apt-get\ install\ -y\ nginx\ $'\n'sudo\ service\ nginx\ start\ $'\n'sudo\ sed\ -i\ --\ \"s/Welcome\ to\ nginx/Version:2\ -\ Welcome\ to\ \$HOSTNAME/g\"\ /var/www/html/index.nginx-debian.html

# verify instance templates
gcloud compute instance-templates list

# create an instance group based on template 1 
gcloud compute instance-groups managed create $INSTANCE_GROUP_NAME --base-instance-name=$BASE_INSTANCE_NAME --template $INSTANCE_TEMPLATE_NAME1 --size 4 --zone $ZONE

# check if all instances were created
gcloud compute instances list

# perform canary update (50% of instances will be updated)
gcloud compute instance-groups managed rolling-action start-update $INSTANCE_GROUP_NAME --version template=$INSTANCE_TEMPLATE_NAME1 --canary-version template=$INSTANCE_TEMPLATE_NAME2,target-size=50% --zone $ZONE

# check if the new version of the application is OK

# if the test was failed then return to temaplate 1 (*max-unavailable - all the instances need to be replaced with new ones at once)
gcloud compute instance-groups managed rolling-action start-update $INSTANCE_GROUP_NAME --version template=$INSTANCE_TEMPLATE_NAME1 --zone $ZONE --max-unavailable 100%

# if the test was passed then update the rest of instances to temaplate 2 
# update instances group with template 2
gcloud compute instance-groups managed rolling-action start-update $INSTANCE_GROUP_NAME --version template=$INSTANCE_TEMPLATE_NAME2 --zone $ZONE --max-unavailable 100%



# delete regources
gcloud compute instance-groups managed delete $INSTANCE_GROUP_NAME --zone $ZONE
```
