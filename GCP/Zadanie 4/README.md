* [Zadanie 1](#zadanie-1)
  * [1 Przygotowanie `Cloud Storage`](#1-przygotowanie-cloud-storage)
  * [2 Utworzenie `Service Account`](#2-utworzenie-service-account)
  * [3 Utworzenie VM](#3-utworzenie-vm)
  * [4 Sprawdzenie uprawnień](#4-sprawdzenie-uprawnień)
  * [5 Usunięcie zasobów](#5-usunięcie-zasobów)
* [Zadanie 2](#zadanie-2)
* [Zadanie 3](#zadanie-3)


# Zadanie 1

> Klient poprosił cię o przygotowanie maszyny dla swoich pracowników, którzy będą mogli pobierać  faktury z przygotowanego repozytorium ( w naszym przypadku jest to pojemnik Cloud Storage )

> 1. Twoim zadaniem będzie zapewnienie, że użytkownicy korzystający z maszyny, będą mogli odczytywać obiekty z danego pojemnika. Wykreuj instancję, która ma dostęp **tylko** do możliwości odczytywania obiektów znajdujących się w wytworzonym pojemniku Cloud Storage.

> 2. Wrzuć przykładowe dokumenty do swojego pojemnika, a następnie wyświetl jego zawartość z poziomu maszyny, którą przed chwilą skonfigurowałeś. Aby w pełni potwierdzić poprawność konfiguracji spróbuj wykonać usunięcie danego obiektu.

### 1 Przygotowanie `Cloud Storage`

```bash
# Zmienne
bucketName="wd-bucket-zad4"
bucketLocation="europe-west3"

# Utworzenie bucketa
gsutil mb -c STANDARD -l $bucketLocation gs://${bucketName}/

# Wysłanie plików do bucketa
gsutil cp file0* gs://$bucketName/
```

### 2 Utworzenie 'Service Account'

>Wykreuj instancję, która ma dostęp **tylko** do możliwości odczytywania obiektów znajdujących się w wytworzonym pojemniku Cloud Storage.

Role: Storage Object viewer
+ condition: resurce type: bucket
             and
             resource name ends with: wd-bucket-zad4

Role name: wd-szkola_zad4-service-account

### 3 Utworzenie VM

```bash
# Zmienne
vmName="wd-zad4-vm"
vmType="f1-micro"
vmZone="europe-west3-b"
serviceAccountEmail="wd-szkola-zad4-service-account@burnished-mark-260111.iam.gserviceaccount.com"

```
gcloud beta compute instances create $vmName --zone=$vmZone --machine-type=$vmType --service-account=$serviceAccountEmail --tags=http-server,https-server --image=debian-9-stretch-v20191210 --image-project=debian-cloud 

### 4 Sprawdzenie uprawnień

Połączenie się do VM i sprwadzenie czy możemy tylko odczytywać z bucketa.

waldekdcloud@wd-zad4-vm:~$ echo "costam" > file.txt
waldekdcloud@wd-zad4-vm:~$ gsutil cp file.txt gs://wd-bucket-zad4/
Copying file://file.txt [Content-Type=text/plain]...
AccessDeniedException: 403 Insufficient Permission 

### 5 Usunięcie zasobów

```
gcloud compute instances delete $vmName --zone=$vmZone 
gcloud iam service-accounts delete $serviceAccountEmail
gsutil -m rm -r gs://${bucketName}/
rm test*.txt
```

# Zadanie 2
# Zadanie 3

# Zadanie 1

** Dany klient przetrzymuje bardzo ważne dokumenty. Zarząd zdecydował, że wprowadzą szyfrowanie krytycznych dokumentów, które będą mogły zostać odszyfrowane po stronie pracownika, który z danego dokumentu chce skorzystać.**

> 1)
Wykreuj instancję, której umożliwisz dostęp do pobierania dokumentów z pojemnika Cloud Storage. Oprócz uprawnień do pobierania obiektów musisz umożliwić użytkownikowi możliwość szyfrowania/odszyfrowywania danych za pomocą klucza znajdującego się w Cloud KMS.

https://cloud.netapp.com/blog/google-cloud-storage-encryption-key-management-in-google-cloud-gcp-cvo-blg


```bash
PROJECT_ID=burnished-mark-260111
--gcloud config set project $PROJECT_ID
--gcloud config set compute/zone us-central1-f

BUCKET_NAME=zad42-wd-${RANDOM}
--gs://zad42-wd-6431/

BUCKET_LOCATION="europe-west3"

gsutil mb -c STANDARD -l $BUCKET_LOCATION gs://${BUCKET_NAME} 

SERVICE_NAME=zad42-service-${RANDOM}
--zad42-service-20937

SERVICE_ACCOUNT=${SERVICE_NAME}@${PROJECT_ID}.iam.gserviceaccount.com
--zad42-service-20937@burnished-mark-260111.iam.gserviceaccount.com
echo $SERVICE_ACCOUNT

gcloud iam service-accounts create $SERVICE_NAME --display-name $SERVICE_NAME

ROLE1=storage.objectAdmin
ROLE2=storage.objectCreator
ROLE3=cloudkms.cryptoKeyEncrypterDecrypter

# add permissions to service account
gcloud projects add-iam-policy-binding  $PROJECT_ID --member serviceAccount:zad42-service-20937@burnished-mark-260111.iam.gserviceacco
unt.com --role roles/$ROLE

# add permissions to bucket as only bucket creator has default access to the bucket
gsutil iam ch serviceAccount:${SERVICE_ACCOUNT}:objectCreator gs://$BUCKET_NAME

# crate VM
vmName="zad42-vm"
vmType="f1-micro"
vmZone="europe-west3-b"

gcloud compute instances create $vmName --zone $vmZone --machine-type $vmType --image-project=debian-cloud --image=debian-9-stretch-v20191210 --service-account $SERVICE_ACCOUNT

```


> Wytwórz klucz w usłudze Cloud KMS:

>Może być to klucz symetryczny, którego użyjesz zarówno do zaszyfrowania jak i odszyfrowania dokumentu
Lub mogą być to dwa osobne klucze asymetryczne za pomocą, których zaszfrujesz oraz odszyfrujesz dokument.

```bash
# find or create a keyring
gcloud kms keyrings list --location global

# fing or create a key (symmetric this case)
gcloud kms keys list --location global --keyring mykeyring

gcloud kms keys create mykeytodelete --purpose encryption --location global --keyring mykeyring

## list keys
gcloud kms keys list --location global --keyring mykeyring
```

> Przetestuj poprawność konfiguracji za pomocą następujących kroków:

Umieść w pojemniku przykładowe dokumenty.

```bash
echo 'gżegżółka jest zajebista' > file1.txt

gsutil cp file1.txt gs://$BUCKET_NAME/
```

Na wykreowanej maszynie spróbuj pobrać przykładowy dokument.
```bash

# Trying to get
gsutil cp gs://zad42-wd-6431/file1.txt test1.txt

# Trying to save a new on the bucket
gsutil cp test1.txt gs://zad42-wd-6431/
Copying file://test1.txt [Content-Type=text/plain]...
AccessDeniedException: 403 Insufficient Permission      
```

>Zaszyfruj dokument za pomocą klucza utworzonego wcześniej w Cloud KMS

```bash
 gcloud kms encrypt --location global --keyring mykeyring --key mykey --plaintext-file test1.txt --ciphertext-file text1.enc
ERROR: (gcloud.kms.encrypt) PERMISSION_DENIED: Request had insufficient authentication scopes.

#stop instance

# modification of scope
#gcloud compute instances set-service-account $INSTANCE_NAME --scopes storage-rw --zone urope-west3-b

gcloud compute instances set-service-account $INSTANCE_NAME --zone europe-west3-b  --scopes https://www.googleapis.com/auth/cloudkms,https://www.googleapis.com/auth/devstorage.read_write

#start instance
```

>Na sam koniec spróbuj odszyfrować dany plik, aby przywrócić do postaci pierwotnej.
Dla osób mających więcej czasu na zadanie zachęcamy, aby wykonać odszyfrowanie danych na innej maszynie niż ta, na której zostało wykonane szyfrowanie. To pozwoli lepiej przećwiczyć temat uprawnień do wykonywania różnych funkcji za pomocą różnych usług - w naszym przypadku Cloud Storage oraz Cloud KMS.

```bash
gcloud kms decrypt --location global --keyring mykeyring --key mykey --ciphertext-file text1.enc --plaintext-file text1_decrypted.txt
```

# Zadanie 3
>Firma zdecydowała się już na ostatni krok ... zbudowanie niestandardowej roli za pomocą, której połączą możliwości szyfrowania oraz odszyfrowywania danych za pomocą KMS oraz dostępu do danych w Cloud Storage na poziomie READ

>Wytwórz niestandardową rolę oraz przypisz odpowiednie uprawnienia, aby spełnić powyższe wymagania.

```bash
more custom_role.yaml

title: Zad43_EncrDescRead
description: Encdyption-Decryption-Read
stage: "ALPHA"
includedPermissions:
- cloudkms.cryptoKeyVersions.useToEncrypt
- cloudkms.cryptoKeyVersions.useToDecrypt
- storage.objects.get

gcloud iam roles create WDEncrDecrRO --project burnished-
mark-260111 --file custom_role.yaml

WARNING: API is not enabled for permissions: [storage.objects.get]. Please enable the corresponding APIs to use those p
ermissions.
Created role [WDEncrDecrRO].
description: Encdyption-Decryption-Read
etag: BwWc9z3NxNs=
includedPermissions:
- cloudkms.cryptoKeyVersions.useToDecrypt
- cloudkms.cryptoKeyVersions.useToEncrypt
- storage.objects.get
name: projects/burnished-mark-260111/roles/WDEncrDecrRO
stage: ALPHA
title: Zad43_EncrDescRead

# bind role to the service account
gcloud projects add-iam-policy-binding $PROJECT_NAME --member serviceAccount:$SERVICE_ACCOUNT --role projects/burnished-mark-260111/roles/WDEncrDecrRO

```

>Zadanie wymagałoby stworzenia przykładowego użytkownika co wiąże się z założeniem nowego konta Google, dlatego w zamian za to proponujemy przejrzenie dokumentacji w jaki sposób jesteśmy w stanie wykreować niestandardową rolę: Creating custom roles - takich sposobów jest kilka.

```bash
# new service account created
gcloud iam service-accounts create zad432-service-account --display-name zad432-service-account

# new custom role binded to this service account
gcloud projects add-iam-policy-binding $PROJECT_NAME --member serviceAccount:$SERVICE_ACCOUNT2 --role projects/burnished-mark-260111/roles/WDEncrDecrRO

# change service accoutn for VM




# test on VM
waldekdcloud@zad42-vm:~$ gsutil ls gs://zad42-wd-6431/
AccessDeniedException: 403 zad432-service-account@burnished-mark-260111.iam.gserviceaccount.com does not have storage.objects.list access to zad42-wd-6431.
waldekdcloud@zad42-vm:~$ gsutil cp gs://zad42-wd-6431/file1.txt xxx.txt
Copying gs://zad42-wd-6431/file1.txt...
/ [1 files][   93.0 B/   93.0 B]                                                
Operation completed over 1 objects/93.0 B. 

waldekdcloud@zad42-vm:~$ gcloud kms encrypt --location global --keyring mykeyring --key mykey --plaintext-file xxx.txt --ciphertext-file xxx.enc
waldekdcloud@zad42-vm:~$ ls
file2.txt  test1.txt  text1_decrypted.txt  text1.enc  xxx.enc  xxx.txt  zad42-wd-6431
waldekdcloud@zad42-vm:~$ diff xxx.txt xxx.enc 
Binary files xxx.txt and xxx.enc differ
waldekdcloud@zad42-vm:~$ more xxx.enc
$
�C�Z    h�AR�E�f^L�V\^LL��)��\�D-�����z�%����%T�2��P
waldekdcloud@zad42-vm:~$ gcloud kms decrypt --location global --keyring mykeyring --key mykey --plaintext-file xxx_new.txt --ciphertext-file xxx.enc
waldekdcloud@zad42-vm:~$ diff xxx_new.txt xxx.txt 
waldekdcloud@zad42-vm:~$ more xxx_new.txt 
gżegżółka jest zajebista
a henio znalazl dużą kure i krzakach
i zjadl obiad żę hej!

```
