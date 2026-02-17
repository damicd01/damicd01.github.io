**Terraform Code to Stand Up Infrastructure to Learn Concepts**

- In order to overcome the chicken and egg scenario of creating a bucket for the state file and then managing it via terraform the following was done:

```bash
gcloud storage buckets create gs://gcp-networking-cert-project-terraform-state -l eu
gsutil versioning set on gs://gcp-networking-cert-project-terraform-state
```

- Once creted the above manually we created the terraform resources to allow us to import into the state so we could manage these via terraform too (this willbe its own standlone tf state file) - ```main.tf```

```terraform
terraform {
  required_providers {
    google = {
      version = "7.19.0"
    }
  }
  backend "gcs" {}
}
  
provider "google" {
  project = var.gcp_project
}
```

- terraform backend config file

```terraform
bucket="gcp-networking-cert-project-terraform-state"
prefix="terraform/gcsbucket-state-file"
```

- terraform ```state_bucket.tf```

```terraform
resource "google_storage_bucket" "terraform_state_file" {
  name          = "gcp-networking-cert-project-terraform-state"
  location      = "EU"
   
  versioning {
    enabled = true
  }
}
output "TFSTATE_BUCKET_NAME" {
  value = google_storage_bucket.terraform_state_file.url
}
```

- A terraform resource with the same name was then created and I imported the resource using ```terraform import google_storage_bucket.terraform_state_file gcp-networking-cert-project-terraform-state```

- At this point I was able to create the following VPC with 2 subnets and a Firewall Rule

```terraform
# Configure the Google Cloud Provider
provider "google" {
  project = "<YOUR_PROJECT_ID>"
  region  = "us-central1"
}

# 1. Create a Custom VPC (Section 4 focuses on Custom over Auto)
resource "google_compute_network" "custom_vpc" {
  name                    = "mistry-course-vpc"
  auto_create_subnetworks = false # This makes it a "Custom Mode" VPC
  routing_mode            = "REGIONAL"
}

# 2. Create Subnet A (US Region)
resource "google_compute_subnetwork" "subnet_us" {
  name          = "subnet-us-central"
  ip_cidr_range = "10.0.1.0/24"
  region        = "us-central1"
  network       = google_compute_network.custom_vpc.id
}

# 3. Create Subnet B (Europe Region)
resource "google_compute_subnetwork" "subnet_eu" {
  name          = "subnet-europe-west"
  ip_cidr_range = "10.0.2.0/24"
  region        = "europe-west1"
  network       = google_compute_network.custom_vpc.id
}

# 4. Basic Firewall Rule (Allow SSH for testing)
resource "google_compute_firewall" "allow_ssh" {
  name    = "allow-ssh-inbound"
  network = google_compute_network.custom_vpc.name

  allow {
    protocol = "tcp"
    ports    = ["22"]
  }

  source_ranges = ["0.0.0.0/0"] # In a real env, restrict this to your IP
}

```

- to build run:

```bash
terraform init
terraform plan
terraform apply -auto-approve
```

- to destroy:

```bash
terraform destroy -auto-approve
```
