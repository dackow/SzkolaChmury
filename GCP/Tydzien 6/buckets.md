# Cloud storage

```bash
BUCKET_NAME=wd-bucket-no-1
BUCKET=gs://$BUCKET_NAME

# transfer to bucket
# -m - pararell transfer of many files
gsutil -m cp * $BUCKET 

# check size of data inside of bucket
# IF the size of data in bucket is huge, the below operation is long.
# Then better is to use StackDriver monitoring where once a day there is bucket size determined and saves to logs.
gsutil du -s $BUCKET

# Returns metadata connected with the bucket.
gsutil ls -L -b $BUCKET

# create a bucket
STORAGE_CLASS=standard
LOCAL=us-east1

BUCKET2=gs://wd-bucket-no-2

gsutil mb -c $STORAGE_CLASS -l $LOCAL $BUCKET2

# list buckets
gsutil ls

# copy files from one bucket to another
gsutil cp $BUCKET/** $BUCKET2

# display files inside of bucket
gsutil ls -r $BUCKET2

# change storageclass to a bucket
NEW_STORAGE_CLASS=nearline

gsutil defstorageclass set $NEW_STORAGE_CLASS $BUCKET2

# upload a file 
touch testfile.txt
vi testfile.txt

gsutil cp testfile.txt $BUCKET2
```

