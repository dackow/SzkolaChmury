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


# Zadanie 2
# Zadanie 3
