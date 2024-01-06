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
   

# Import Infrastructure

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

   Copy the followinf configuration into the file **modules/instances/instances.tf**
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
