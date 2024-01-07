# Create the configuration files

1. In Cloud Shell, create your Terraform configuration files and a directory structure that resembles the following:
   
   Create file `main.tf`
   ```
   touch main.tf
   ```
   Create file `variables.tf`
   ```
   touch variables.tf
   ```
   Create directory `modules`
   ```
   mkdir modules
   ```
   Go to directory `modules`
   ```
   cd modules
   ```
   Create directory `instances`
   ```
   mkdir instances
   ```
   Go to directory `instances`
   ```
   cd instances
   ```
   Create file `instances.tf`
   ```
   touch instances.tf
   ```
   Create file `outputs.tf`
   ```
   touch outputs.tf
   ```
   Create file `variables.tf`
   ```
   touch variables.tf
   ```
   Go back to previous directory (`modules`)
   ```
   cd ..
   ```
   Create directory `storage`
   ```
   mkdir storage
   ```
   Go to directory `storage`
   ```
   cd storage
   ```
   Create file `storage.tf`
   ```
   touch storage.tf
   ```
   Create file `outputs.tf`
   ```
   touch outputs.tf
   ```
   Create file `variables.tf`
   ```
   touch variables.tf
   ```
   Go back to previous directory (`modules`)
   ```
   cd
   ```

   
2. Fill out the `variables.tf` files in the root directory and within the modules. Add three variables to each file: `region`, `zone`, and `project_id`. For their default values, use , `<filled in at lab start>`, and your Google Cloud Project ID.
   
   ```
   variable "region" {
     default = "us-central1"
   }

   variable "zone" {
     default = "us-central1-a"
   }

   variable "project_id" {
     default = "<FILL IN PROJECT ID>"
   }
   ```

   
3. Add the Terraform block and the [Google Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs) to the `main.tf` file. Verify the **zone** argument is added along with the **project** and **region** arguments in the Google Provider block.

   ```
   terraform {
     required_providers {
       google = {
         source   = "hashicorp/google"
         version  = "4.47.0"
       }
     }
   }

   provider "google" {
     project = var.project_id
     region  = var.region
     zone    = var.zone
   }
   ```

   
4. Initialize Terraform
   
   Run
   
   ```
   terraform init
   ```

   In Cloud Shell in the root directory to initialize terraform
   

# Import infrastructure

1. In the Google Cloud Console, on the **Navigation menu**, click **Compute Engine > VM Instances**. Two instances named **tf-instance-1** and **tf-instance-2** have already been created for you.

   Go to Compute Engine > VM Instances. Click on **tf-instance-1**. Copy the Instance ID down somewhere to use later.

   Go to Compute Engine > VM Instances. Click on **tf-instance-2**. Copy the instance ID down somewhere to use later.

   
2. [Import](https://www.terraform.io/docs/cli/commands/import.html#example-import-into-module) the existing instances into the **instances** module.

   Add the module reference to the end of `main.tf`
   ```
   module "instances" {
      source = "./modules/instances"
   }
   ```

   Copy the following configuration into the file **modules/instances/instances.tf**
   ```
   resource "google_compute_instance" "tf-instance-1" {
      name         = "tf-instance-1"
      machine_type = "n1-standard-1"
      zone         = var.zone
      boot_disk {
         initialize_params {
            image = "debian-cloud/debian-10"
         }
      }
      network_interface {
         network = "default
      }
      metadata_startup_script = <<-EOT
            #!/bin/bash
         EOT
      allow_stopping_for_update = true
   }

   resource "google_compute_instance" "tf-instance-2" {
      name         = "tf-instance-2"
      machine_type = "n1-standard-1"
      zone         = var.zone
      boot_disk {
         initialize_params {
            image = "debian-cloud/debian-10"
         }
      }
      network_interface {
         network = "default
      }
      metadata_startup_script = <<-EOT
            #!/bin/bash
         EOT
      allow_stopping_for_update = true
   }
   ```

   Use the terraform import command to import them into your instances module.

   To import the first instance, use the following command, using the Instance ID for tf-instance-1 you copied down earlier.
   ```
   terraform import module.instances.google_compute_instance.tf-instance-1 <Instance ID - 1>
   ```

   To import the second instance, use the following command, using the Instance ID for tf-instance-2 you copied down earlier.
   ```
   terraform import module.instances.google_compute_instance.tf-instance-2 <Instance ID - 2>
   ```
   
4. Apply your changes. Note that since you did not fill out all of the arguments in the entire configuration, the `apply` will **update the instances in-place**. This is fine for lab purposes, but in a production environment, you should make sure to fill out all of the arguments correctly before importing.

   ```
   terraform plan
   ```
   ```
   terraform apply
   ```


# Configure a remote backend

1. Create a [Cloud Storage bucket resource](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_bucket) inside the `storage` module. For the bucket name, use **`Bucket Name`**

   Add the following code to the **modules/storage/storage.tf** file
   ```
   resource "google_storage_bucket" "storage-bucket" {
      name            = "Bucket Name"
      location        = "US"
      force_destroy   = true
      uniform_bucket_level_access = true
   }
   ```
   
2. Add the module reference to the `main.tf` file. Initialize the module and `apply` the changes to create the bucket using Terraform.

   Add the following to end of the `main.tf` file
   ```
   module "storage" {
      source = "./modules/storage"
   }
   ```

   Run the following commands in cloud shell
   ```
   terraform init
   ```
   ```
   terraform apply
   ```
   
3. Configure this storage bucket as the [remote backend](https://www.terraform.io/docs/language/settings/backends/gcs.html) inside the `main.tf` file. Be sure to use the **prefix** `terraform/state` so it can be graded successfully.

   ```
   terraform {
      backend "gcs" {
         bucket   = "Bucket Name"
         prefix   = "terraform/state"
      }
      required_providers {
         google = {
            source   = "hashicorp/google"
            version  = "4.47.0"
         }
      }
   }
   ```
   
4. If you've written the configuration correctly, upon `init`, Terraform will ask whether you want to copy the existing state data to the new backend. Type `yes` at the prompt.

   Run the following command to initialize the remote backend
   ```
   terraform init
   ```


# Modify and update infrastructure

1. Navigate to the **instances** module and modify the **tf-instance-1** resource to use an `e2-standard-2` machine type.

   Navigate to **modules/instances/instance.tf**. Replace the entire contents of the file with the following:
   ```
   resource "google_compute_instance" "tf-instance-1" {
      name           = "tf-instance-1"
      machine_type   = "e2-standard-2"
      zone           = var.zone
      allow_stopping_for_update = true

      boot_disk {
         initialize_params {
            image = "debian-cloud/debian-10"
         }
      }

      network_interface {
         network = "default"
      }
   }
   ```
   
2. Modify the **tf-instance-2** resource to use an `e2-standard-2` machine type.

   Navigate to **modules/instances/instance.tf**. Add contents of the file with the following:
   ```
   resource "google_compute_instance" "tf-instance-2" {
      name           = "tf-instance-2"
      machine_type   = "e2-standard-2"
      zone           = var.zone
      allow_stopping_for_update = true

      boot_disk {
         initialize_params {
            image = "debian-cloud/debian-10"
         }
      }

      network_interface {
         network = "default"
      }
   }
   ```
   
3. Add a third instance resource and name it **`Instance Name`**. For this third resource, use an `e2-standard-2` machine type. Make sure to change the machine type to `e2-standard-2` **to all the three instances**.

   Navigate to **modules/instances/instance.tf**. Add contents of the file with the following:
   ```
   resource "google_compute_instance" "Instance Name" {
      name           = "tf-instance-3"
      machine_type   = "e2-standard-2"
      zone           = var.zone
      allow_stopping_for_update = true

      boot_disk {
         initialize_params {
            image = "debian-cloud/debian-10"
         }
      }

      network_interface {
         network = "default"
      }
   }
   ```
   
4. Initialize Terraform and `apply` your changes.

   Run the following commands
   ```
   terraform init
   ```
   ```
   terraform apply
   ```
   
# Destroy resources

1. Destroy the third instance **`Instance Name`** by removing the resource from the configuration file. After removing it, initialize terraform and `apply` the changes.
   
   Taint the **Instance Name** resource by running the following commmand
   ```
   terraform taint module.instances.google_compute_instance.Instance Name
   ```

   Run the following commands to apply the changes
   ```
   terraform init
   ```
   ```
   terraform apply
   ```

   Remove the following chunk of code from `instances.tf` file
   ```
   resource "google_compute_instance" "Instance Name" {
      name           = "tf-instance-3"
      machine_type   = "e2-standard-2"
      zone           = var.zone
      allow_stopping_for_update = true

      boot_disk {
         initialize_params {
            image = "debian-cloud/debian-10"
         }
      }

      network_interface {
         network = "default"
      }
   }
   ```

   Run the following command
   ```
   terraform apply
   ```

   
# Use a module from the Registry

1. In the Terraform Registry, browse to the [Network Module](https://registry.terraform.io/modules/terraform-google-modules/network/google/6.0.0).
   
2. Add this module to your `main.tf` file.
   ```
   module "vpc" {
      source   = "terraform-google-modules/network/google"
      version  = "6.0.0"

      project_id = "Enter your project id here"
      network_name = "VPC Name"
      routing_mode = "GLOBAL"

      subnets = [
         {
            subnet_name   = "subnet-01"
            subnet_ip     = "10.10.10.0/24"
            subnet_region = "us-east1"
         },
         {
            subnet_name   = "subnet-02"
            subnet_ip     = "10.10.20.0/24"
            subnet_region = "us-east1"
         }
      ]
   }
   ```
   
3. Once you've written the module configuration, initialize Terraform and run an `apply` to create the networks.

   Run the following commands
   ```
   terraform init
   ```
   ```
   terraform apply
   ```
   
4. Next, navigate to the `instances.tf` file and update the configuration resources to connect **tf-instance-1** to `subnet-01` and **tf-instance-2** to `subnet-02`.

   Navigate to **modules/instances/instances.tf**. Replace the entire contents of the file with thw following
   ```
   resource "google_compute_instance" "tf-instance-1" {
      name           = "tf-instance-1"
      machine_type   = "e2-standard-2"
      zone           = var.zone
      allow_stopping_for_update = true

      boot_disk {
         initialize_params {
            image = "debian-cloud/debian-10"
         }
      }

      network_interface {
         network    = "VPC Name"
         subnetwork = "subnet-01"
      }
   }

   resource "google_compute_instance" "tf-instance-2" {
      name           = "tf-instance-2"
      machine_type   = "e2-standard-2"
      zone           = var.zone
      allow_stopping_for_update = true

      boot_disk {
         initialize_params {
            image = "debian-cloud/debian-10"
         }
      }

      network_interface {
         network      = "VPC Name"
         subnetwork   = "subnet-02"
      }
   }
   ```

   Run the following commands
   ```
   terraform init
   ```
   ```
   terraform apply
   ```
   
# Configure a firewall

1. Create a [firewall rule](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_firewall) resource in the `main.tf` file, and name it **tf-firewall**.

   Add the following resource to the `main.tf` file and fill in the GCP Project ID
   ```
   resource "google_compute_firewall" "tf-firewall" {
      name    = "tf-firewall"
      network = "projects/PROJECT_ID/global/networks/VPC Name"

      allow {
         protocol = "tcp"
         ports    = ["80"]
      }

      source_tags   = ["web"]
      source_ranges = ["0.0.0.0/0"]
   }
   ```

   Run the following commands
   ```
   terraform init
   ```
   ```
   terraform apply
   ```
