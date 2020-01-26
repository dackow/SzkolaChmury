# Autohealing + Health check

```bash
sudo apt-get update
sudo apt-get -y install git python3-pip gunicorn3
git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git 
cd python-docs-samples/compute/managed-instances/demo
sudo pip3 install -r requirements.txt
sudo gunicorn3 --bind 0.0.0.0:80 app:app --daemon
```
