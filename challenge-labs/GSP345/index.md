# [GSP345] Automating Infrastructure on Google Cloud with Terraform: Challenge Lab


### [GSP345](https://www.cloudskillsboost.google/focuses/42740?parent=catalog)

![](https://cdn.qwiklabs.com/GMOHykaqmlTHiqEeQXTySaMXYPHeIvaqa2qHEzw6Occ%3D)

---

Time: 1 hour 30 minutes<br>
Difficulty: Introductory<br>
Price: 1 Credit

Quest: [Automating Infrastructure on Google Cloud with Terraform](https://www.cloudskillsboost.google/quests/159)<br>

Last updated: May 19, 2023

---

## Challenge scenario

You are a cloud engineer intern for a new startup. For your first project, your new boss has tasked you with creating infrastructure in a quick and efficient manner and generating a mechanism to keep track of it for future reference and changes. You have been directed to use [Terraform](https://www.terraform.io/) to complete the project.

For this project, you will use Terraform to create, deploy, and keep track of infrastructure on the startup's preferred provider, Google Cloud. You will also need to import some mismanaged instances into your configuration and fix them.

In this lab, you will use Terraform to import and create multiple VM instances, a VPC network with two subnetworks, and a firewall rule for the VPC to allow connections between the two instances. You will also create a Cloud Storage bucket to host your remote backend.


## Task 1. Create the configuration files

1. Make the empty files and directories in Cloud Shell or the Cloud Shell Editor.

    ```bash
    touch main.tf
    touch variables.tf
    mkdir modules
    cd modules
    mkdir instances
    cd instances
    touch instances.tf
    touch outputs.tf
    touch variables.tf
    cd ..
    mkdir storage
    cd storage
    touch storage.tf
    touch outputs.tf
    touch variables.tf
    cd
    ```

    Folder structure should look like this:

    ```bash
    main.tf
    variables.tf
    modules/
    └── instances
        ├── instances.tf
        ├── outputs.tf
        └── variables.tf
    └── storage
        ├── storage.tf
        ├── outputs.tf
        └── variables.tf
    ```

2. Add the following to the each variables.tf file, and replace `PROJECT_ID` with your GCP Project ID, also change the `REGION` and the `ZONE` based on the lab instructions.

    ```terraform
    variable "region" {
        default = "<****us-central1****>"
    }

    variable "zone" {
        default = "<****us-central1-a****>"
    }

    variable "project_id" {
        default = "<****PROJECT_ID****>"
    }
    ```

3. Add the following to the `main.tf` file.

    ```terraform
    terraform {
        required_providers {
            google = {
                source = "hashicorp/google"
                version = "4.53.0"
            }
        }
    }

    provider "google" {
        project     = var.project_id
        region      = var.region
        zone        = var.zone
    }

    module "instances" {
        source     = "./modules/instances"
    }
    ```

4. Run the following commands in Cloud Shell in the root directory to initialize terraform.

    ```bash
    terraform init
    ```

## Task 2. Import infrastructure

1. In the Cloud Console, go to the **Navigation menu** and select **Compute Engine**.
2. Click the `tf-instance-1`, then copy the **Instance ID** down somewhere to use later.
   ![Instance ID](/challenge-labs/GSP345/images/Instance%20ID.png)
3. In the Cloud Console, go to the **Navigation menu** and select **Compute Engine**.
4. Do the same thing on previous step, click the `tf-instance-2`, then copy the **Instance ID** down somewhere to use later.
5. Next, navigate to `modules/instances/instances.tf`. Copy the following configuration into the file.

    ```terraform
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
            network = "default"
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
            network = "default"
        }

        metadata_startup_script = <<-EOT
                #!/bin/bash
            EOT
        allow_stopping_for_update = true
    }
    ```

6. Run the following commands in Cloud Shell to import the first instance. Replace `INSTANCE_ID_1` with **Instance ID** for `tf-instance-1` you copied down earlier.

    ```bash
    terraform import module.instances.google_compute_instance.tf-instance-1 <****INSTANCE_ID_1****>
    ```

7. Run the following commands in Cloud Shell to import the first instance. Replace `INSTANCE_ID_2` with **Instance ID** for `tf-instance-2` you copied down earlier.

    ```bash
    terraform import module.instances.google_compute_instance.tf-instance-2 <****INSTANCE_ID_2****>
    ```

8. Run the following commands to apply your changes.

    ```bash
    terraform plan

    terraform apply
    ```

## Task 3. Configure a remote backend

1. Add the following code to the `modules/storage/storage.tf` file. Replace `BUCKET_NAME` with bucket name given in lab instructions.

    ```terraform
    resource "google_storage_bucket" "storage-bucket" {
        name          = "<****BUCKET_NAME****>"
        location      = "US"
        force_destroy = true
        uniform_bucket_level_access = true
    }
    ```

2. Next, add the following to the `main.tf` file.

    ```terraform
    module "storage" {
        source     = "./modules/storage"
    }
    ```

3. Run the following commands to initialize the module and create the storage bucket resource. Type `yes` at the dialogue after you run the apply command to accept the state changes.

    ```bash
    terraform init

    terraform apply
    ```

4. Next, update the `main.tf` file so that the terraform block looks like the following. Fill in your GCP Project ID for the bucket argument definition. Replace `BUCKET_NAME` with Bucket Name given in lab instructions.

    ```terraform
    terraform {
        backend "gcs" {
            bucket  = "<****BUCKET_NAME****>"
            prefix  = "terraform/state"
        }

        required_providers {
            google = {
                source = "hashicorp/google"
                version = "4.53.0"
            }
        }
    }
    ```

5. Run the following commands to initialize the remote backend. Type `yes` at the prompt.

    ```bash
    terraform init
    ```

## Task 4. Modify and update infrastructure

1. Navigate to `modules/instances/instance.tf`. Replace the entire contents of the file with the following, then replace `INSTANCE_NAME` with instance name given in lab instructions.

    ```terraform
    resource "google_compute_instance" "tf-instance-1" {
        name         = "tf-instance-1"
        machine_type = "n1-standard-2"
        zone         = var.zone

        boot_disk {
            initialize_params {
                image = "debian-cloud/debian-10"
            }
        }

        network_interface {
            network = "default"
        }

        metadata_startup_script = <<-EOT
                #!/bin/bash
            EOT
        allow_stopping_for_update = true
    }

    resource "google_compute_instance" "tf-instance-2" {
        name         = "tf-instance-2"
        machine_type = "n1-standard-2"
        zone         = var.zone

        boot_disk {
            initialize_params {
                image = "debian-cloud/debian-10"
            }
        }

        network_interface {
            network = "default"
        }

        metadata_startup_script = <<-EOT
                #!/bin/bash
            EOT
        allow_stopping_for_update = true
    }

    resource "google_compute_instance" "<****INSTANCE_NAME****>" {
        name         = "<****INSTANCE_NAME****>"
        machine_type = "n1-standard-2"
        zone         = var.zone

        boot_disk {
            initialize_params {
                image = "debian-cloud/debian-10"
            }
        }

        network_interface {
            network = "default"
        }

        metadata_startup_script = <<-EOT
                #!/bin/bash
            EOT
        allow_stopping_for_update = true
    }
    ```

2. Run the following commands to initialize the module and create/update the instance resources. Type `yes` at the dialogue after you run the apply command to accept the state changes.

    ```bash
    terraform init

    terraform apply
    ```

## Task 5. Destroy resources

1. Taint the `INSTANCE_NAME` resource by running the following command.

    ```bash
    terraform taint module.instances.google_compute_instance.<****INSTANCE_NAME****>
    ```

2. Run the following commands to apply the changes.

    ```bash
    terraform init

    terraform apply
    ```

3. Remove the `INSTANCE_NAME` (instance 3) resource from the `instances.tf` file. Delete the following code chunk from the file.

    ```terraform
    resource "google_compute_instance" "<****INSTANCE_NAME****>" {
        name         = "<****INSTANCE_NAME****>"
        machine_type = "n1-standard-2"
        zone         = var.zone

        boot_disk {
            initialize_params {
                image = "debian-cloud/debian-10"
            }
        }

        network_interface {
            network = "default"
        }

        metadata_startup_script = <<-EOT
                #!/bin/bash
            EOT
        allow_stopping_for_update = true
    }
    ```

4. Run the following commands to apply the changes. Type yes at the prompt.

    ```bash
    terraform apply
    ```

## Task 6. Use a module from the Registry

1. Copy and paste the following into the `main.tf` file. Replace `VPC_NAME` with VPC Name given in lab instructions.

    ```terraform
    module "vpc" {
        source  = "terraform-google-modules/network/google"
        version = "~> 6.0.0"

        project_id   = var.project_id
        network_name = "<****VPC_NAME****>"
        routing_mode = "GLOBAL"

        subnets = [
            {
                subnet_name           = "subnet-01"
                subnet_ip             = "10.10.10.0/24"
                subnet_region         = var.region
            },
            {
                subnet_name           = "subnet-02"
                subnet_ip             = "10.10.20.0/24"
                subnet_region         = var.region
                subnet_private_access = "true"
                subnet_flow_logs      = "true"
                description           = "This subnet has a description"
            }
        ]
    }
    ```

2. Run the following commands to initialize the module and create the VPC. Type `yes` at the prompt.

    ```bash
    terraform init

    terraform apply
    ```

3. Navigate to `modules/instances/instances.tf`. Replace the entire contents of the file with the following. Replace `VPC_NAME` with VPC Name given in lab instructions.

    ```terraform
    resource "google_compute_instance" "tf-instance-1" {
        name         = "tf-instance-1"
        machine_type = "n1-standard-2"
        zone         = var.zone

        boot_disk {
            initialize_params {
                image = "debian-cloud/debian-10"
            }
        }

        network_interface {
            network = "<****VPC_NAME****>"
            subnetwork = "subnet-01"
        }

        metadata_startup_script = <<-EOT
                #!/bin/bash
            EOT
        allow_stopping_for_update = true
    }

    resource "google_compute_instance" "tf-instance-2" {
        name         = "tf-instance-2"
        machine_type = "n1-standard-2"
        zone         = var.zone

        boot_disk {
            initialize_params {
                image = "debian-cloud/debian-10"
            }
        }

        network_interface {
            network = "<****VPC_NAME****>"
            subnetwork = "subnet-02"
        }

        metadata_startup_script = <<-EOT
                #!/bin/bash
            EOT
        allow_stopping_for_update = true
    }

        module "vpc" {
        source  = "terraform-google-modules/network/google"
        version = "~> 6.0.0"

        project_id   = "*****PROJECT_ID****"
        network_name = "****VPC_NAME*****"
        routing_mode = "GLOBAL"

        subnets = [
            {
                subnet_name           = "subnet-01"
                subnet_ip             = "10.10.10.0/24"
                subnet_region         = "us-central1"
            },
            {
                subnet_name           = "subnet-02"
                subnet_ip             = "10.10.20.0/24"
                subnet_region         = "us-central1"
                subnet_private_access = "true"
                subnet_flow_logs      = "true"
                description           = "This subnet has a description"
            },
        ]
    }
    ```

4. Run the following commands to initialize the module and update the instances. Type `yes` at the prompt.

    ```bash
    terraform init

    terraform apply
    ```

## Task 7. Configure a firewall

1. Add the following resource to the `main.tf` file and replace `PROJECT_ID` and `VPC_NAME` with your GCP Project ID and VPC Name given in lab instructions.

    ```terraform
    resource "google_compute_firewall" "tf-firewall" {
        name    = "tf-firewall"
        network = "projects/<****PROJECT_ID****>/global/networks/<****VPC_NAME****>"

        allow {
            protocol = "tcp"
            ports    = ["80"]
        }

        source_tags = ["web"]
        source_ranges = ["0.0.0.0/0"]
    }
    ```

2. Run the following commands to configure the firewall. Type `yes` at the prompt.

    ```bash
    terraform init

    terraform apply
    ```

## Congratulations!

![Badge](https://cdn.qwiklabs.com/RGaT7KirRAjGDJTaOTOuax2BzYId0zvvGTs%2BPpGlcQI%3D)
