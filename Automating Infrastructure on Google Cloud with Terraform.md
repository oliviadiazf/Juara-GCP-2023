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
   
