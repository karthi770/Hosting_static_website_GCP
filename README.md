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
(Option-1)
![image](https://github.com/karthi770/Hosting-Wordpress-AWS/assets/102706119/5de1eb64-07ed-4c49-9897-ee07c6dd633a)
>[!note]
>Option -1 
>Before creating a Zone, create a Domain name with Google Cloud domains which can be used here in the domain name.
>Load balancer -> Cloud DNS -> Create Zone
>The option-1 can be used if we have a domain name from another domain name provider, we can create a subdomain and use it. Since I have google cloud domain, I have my DNS created automatically.

![image](https://github.com/karthi770/Hosting_static_website_GCP/assets/102706119/898e15f7-2024-4cda-8f72-78b0b233f7a7)
![image](https://github.com/karthi770/Hosting_static_website_GCP/assets/102706119/929e972a-6b4e-46d6-aa52-fe99e0253dbd)
>[!note]
>Register the domain name

![image](https://github.com/karthi770/Hosting_static_website_GCP/assets/102706119/0f4eba98-a31d-4f07-a4c7-27ee193d33de)

![image](https://github.com/karthi770/Hosting_static_website_GCP/assets/102706119/d6d03dd3-388b-4f27-9efb-2e6c8d9fffec)

### Domain Configuration

```python
#main.tf

#Reserve a static external IP address
resource "google_compute_global_address" "website_ip" {
  name = "website-lb-ip"
}
data "google_dns_managed_zone" "dns_zone" {
    name = "karthi-club"
}

#Add the reserved external static IP to the DNS
resource "google_dns_record_set" "website"{
    name = "website.${data.google_dns_managed_zone.dns_zone.dns_name}"
    type = "A"
    ttl = 300
    managed_zone = data.google_dns_managed_zone.dns_zone.name
    rrdatas = [google_compute_global_address.website_ip.address]
}

#Add the bucket as a CDN backend
resource "google_compute_backend_bucket" "website-backend"{
    name = "website-bucket"
    bucket_name =  google_storage_bucket.website.name
    description = "Containes files needed for the website"
    enable_cdn = true
}

#URL MAP
resource "google_compute_url_map" "website"{
  name = "website-url-map"
  default_service = google_compute_backend_bucket.website-backend.self_link
  host_rule {
    hosts = ["*"]
    path_matcher = "allpaths"
    }
    path_matcher {
      name = "allpaths"
      default_service = google_compute_backend_bucket.website-backend.self_link
    }
  }
# GCP HTTP Proxy
resource "google_compute_target_http_proxy" "website"{
  name = "website-target-proxy"
  url_map = google_compute_url_map.website.self_link
}

#GCP forwarding Rule
resource "google_compute_global_forwarding_rule" "default" {
  name = "website-forwarding-rule"
  load_balancing_scheme = "External"
  ip_address = google_compute_global_address.website_ip.address
  ip_protocol = "TCP"
  port_range = "80"
  target = google_compute_target_http_proxy.website.self_link
}
```

![image](https://github.com/karthi770/Hosting_static_website_GCP/assets/102706119/de6b4523-74d2-4f78-9716-d85ff4ecccf9)
Since the gif in the website was not working properly, I had to add some additional code.
```python
#main.tf
#Make new object public
resource "google_storage_object_access_control" "public_rule" {
  for_each = var.objects
  object   = google_storage_bucket_object.static_site_src[each.key].name
  bucket   = google_storage_bucket.website.name
  role     = "READER"
  entity   = "allUsers"
}

# upload the html file to the bucket
resource "google_storage_bucket_object" "static_site_src" {
  for_each = var.objects
  name     = each.key
  source   = each.value
  bucket   = google_storage_bucket.website.name
}
```

```python
#variables.tf
variable "objects" {
  type = map(string)
}
```

```python
#terraform.tfvars
objects = {
  "index.html" = "../website/index.html"
  "fork.gif"  = "../website/fork.gif"
}
```

![image](https://github.com/karthi770/Hosting_static_website_GCP/assets/102706119/6c1a1636-3def-4bb4-ba00-44a5eb7f1f84)

