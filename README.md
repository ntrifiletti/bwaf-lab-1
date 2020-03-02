# bwaf-byol-autolicense
These are the steps to use ARM templates hosted in this repo to create a resouce group, and then create a BWAF BYOL instance which automatically grabs a license from a license blob.

go to the azure portal.

open the azure shell.

az group create --name sko2020bwaf --location eastus

az storage account create --name sabwaf --resource-group sko2020bwaf --location eastus --sku Standard_ZRS

az storage container create --account-name sabwaf --name contbwaf3 --auth-mode key --account-key changeme

( now in Azure gui, click on the storage account, on the left menu is "keys", copy the first key )

wget from this repo the file named barracuda-byol-license-list.json 

az storage blob upload --account-name sabwaf --container-name contbwaf3 --name barracuda-byol-license-list.json --file barracuda-byol-license-list.json --auth-mode key --account-key <right click, paste, put your azure storage account key here>

az group deployment create --rollback-on-error --parameters '{"saKey": {"value": "changeme"}}' --resource-group sko2020bwaf --template-uri https://raw.githubusercontent.com/bwolmarans/bwaf-lab/master/sko_bwaf_deployment.json


# bwaf-lab #

This is the Terraform/Postman/Ansible lab.

## Diagram: ##

![Test Image 3](https://github.com/bwolmarans/bwaf-lab/blob/master/rrrr.png)

## End state: ##

![Test Image 4](https://github.com/bwolmarans/bwaf-lab/blob/master/resources_list.png)

### Requirements: ###

* Azure Subscription  
* Azure Shell containing, among other things:
** Git  
** Terraform 0.12+  
** Ansible 2.9.2+  
  
* Optional: PC/Mac that can run Postman  

### Want something running quick, without learning anything? Here's the short version ###
* az vm image terms accept --urn barracudanetworks:waf:hourly:latest  
* git clone https://github.com/bwolmarans/bwaf-lab.git  
* cd bwaf-lab  
* ssh-keygen -m PEM -t rsa -b 2048 ( be careful about existing SSH keys don't over-write )  
* terraform init
* terraform apply
* Edit myazure_rm.yaml, change the string change_me to your Azure resource group name
* ansible-playbook -i ./myazure_rm.yml ./bwaf-playbook.yaml --key-file ~/.ssh/id_rsa --u azureuser
* Find your BWAF public IP in the Azure portal  
* Web browse to your BWAF http://<bwaf public ip>:8000, login admin/Hello123456! and verify in the WAF configuration look good  
* Browse to the DVWA protected service http://<bwaf public ip>:80
  
### Full Instructions if you want to Learn Something Properly ###
* az vm image accept-terms --urn barracudanetworks:waf:hourly:latest  
* git clone https://github.com/bwolmarans/bwaf-lab.git  
* cd bwaf-lab  
* ssh-keygen -m PEM -t rsa -b 2048 ( be careful about existing SSH keys don't over-write )  
* terraform init  
* examine the terraform configuration file
* terraform plan ( You need a unique ID for the Resource Group. Enter your first name + your phone number when asked. For example, my name is Brett and my phone number is (818) 292-7981, so I will enter brett8182925555 )  
* terraform apply ( enter unique ID as above )  
* #CURL COMMANDS and SSH
* curl http://<your BWAF public IP>:8000/restapi/v3.1/login -X POST -H Content-Type:application/json -d '{"username": "admin", "password": "Hello123456!"}' 
* curl -X POST "http://<your BWAF public IP>:8000/restapi/v3.1/services " -H "accept: application/json" -u "<your token>:" -H "Content-Type: application/json" -d '{ "address-version": "IPv4", "app-id": "curl_app_id", "ip-address": "10.0.1.5", "name": "curl_service", "port": 80, "status": "On", "type": "HTTP"}'
* curl -X POST "http://<your BWAF public IP>:8000/restapi/v3.1/services/curl_service/servers " -H "accept: application/json" -u "<your token>:" -H "Content-Type: application/json" -d '{ "ip-address": "10.0.1.4", "status": "In Service", "comments": "string", "port": 8080, "address-version": "IPv4", "identifier": "IP Address", "name": "curl_server"}'
* ssh azureuser@<your ubuntu public IP>
* sudo su
* apt-get --yes --force-yes update
* apt install docker.io --yes --force-yes
* docker run -d --rm -it -p 8080:80 vulnerables/web-dvwa
* manually test. then delete server and service.
* examine the ansible playbook
* Edit myazure_rm.yaml, change the string change_me to your Azure resource group name  
* ansible-playbook -i ./myazure_rm.yml ./bwaf-playbook.yaml --key-file ~/.ssh/id_rsa --u azureuser
* Find your BWAF public IP in the Azure portal  
* Use your web browser, browse to the BWAF, login, and verify in the BWAF GUI the WAF configuration look good  
* How did you know what password to use?  
* Verify access to the DVWA app by browsing to the Ubuntu public IP, port 8080, i.e. http://<Ubuntu public ip>:8080  
* Login as user admin, password is password  
* Click “Create / Reset Database”  
* Click login at the very bottom of the screen  
* Log back in to DVWA as user admin, password is password  
* Choose SQL Injection from the left menu  
* Enter 1 for the user, it will return a single user  
* Perform a simple SQL Injection  
* Enter for the user ' or '1'='1  
* Do you get a list of all users? ( hint: yes )  
* This is because this web application is Dxxx vulnerable, and is not protected.  
* Now, in a new browser tab, browse to the BWAF service http://<bwaf public ip>:80 , login, and repeat step 1 above.  
* Did the BWAF successfully block the SQL injection attack?  
* Did the BWAF log the attack?  
* What do you need to change to get a successful block?  
* terraform destroy  
