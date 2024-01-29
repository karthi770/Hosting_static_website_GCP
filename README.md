## Setting up the GCP account
### Enable APIs
1. Cloud DNS API
![image](https://github.com/karthi770/Hosting-Wordpress-AWS/assets/102706119/f8ff9657-89bd-42c0-b4d1-1f56183e7023)
2. Compute Engine API
![image](https://github.com/karthi770/Hosting-Wordpress-AWS/assets/102706119/a5bf0113-9ec3-4693-9e3c-b2676335f655)
#### Create a Service Account
![image](https://github.com/karthi770/Hosting-Wordpress-AWS/assets/102706119/29f5c4fd-2707-4c2f-92f2-79921b54bb72)
![image](https://github.com/karthi770/Hosting-Wordpress-AWS/assets/102706119/51bbb21b-e59e-4079-b652-ebce45d7d377)
![image](https://github.com/karthi770/Hosting-Wordpress-AWS/assets/102706119/717e5f67-8320-4d65-96dd-ba010cbbbc49)
![image](https://github.com/karthi770/Hosting-Wordpress-AWS/assets/102706119/966b4ef9-b458-4e44-81b3-6daf763709b4)
![image](https://github.com/karthi770/Hosting-Wordpress-AWS/assets/102706119/ff2fc228-6480-4854-9f44-8501981fdc9d)
>[!note]
>Select the service account that we created and click on the API key tab and create the API key in JSON format.

Create a folder named 'infra' and add the files main.tf and provider.tf.

```python 
#provider.tf
provider "google" {
	credentials = file(var.gcp_svc_key)
	project = var.gcp_project
	region = var.gcp_region
}
```

```python
#main.tf
	resource "google_storage_bucket" "website" {
    name = "ex-kar-web"
    location = "US"
}
```

![image](https://github.com/karthi770/Hosting-Wordpress-AWS/assets/102706119/755f491a-2585-4dfb-ac86-59ff2b647b51)
```python
#variables.tf
variable "gcp_svc_key"{

}
variable "gcp_project"{

}
variable "gcp_region" {
}
```

```python
#terraform.tfvars
gcp_svc_key = "../genuine-ether-412514-d49a65bc8faa.json"

gcp_project = "genuine-ether-412514"

gcp_region = "us-east1"
```

```python
terraform init
terraform plan
terraform apply
```
>[!note]
>Run the above commands the bucket shall be created with an object in it.

![image](https://github.com/karthi770/Hosting-Wordpress-AWS/assets/102706119/b6ff0377-7f18-4c6d-8958-4cd5347a9efa)
## Cloud DNS

![image](https://github.com/karthi770/Hosting-Wordpress-AWS/assets/102706119/5de1eb64-07ed-4c49-9897-ee07c6dd633a)
>[!note]
>Before creating a Zone, create a Domain name with Google Cloud domains which can be used here to in the domain name.
>Load balancer -> Cloud DNS -> Create Zone

![image](https://github.com/karthi770/Hosting_static_website_GCP/assets/102706119/898e15f7-2024-4cda-8f72-78b0b233f7a7)



