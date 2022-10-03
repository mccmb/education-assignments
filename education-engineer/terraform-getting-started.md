# Getting Started with Terraform on Docker

HashiCorp Terraform is an infrastructure as code (IaC) tool used to provision and configure resources on-prem and in the cloud. Terraform is modular, meaning it supports virtually any infrastructure provider with an API. Terraform configurations are human-readable files that you can version, share, and reuse.

In this guide, you will write a Terraform configuration, use it to deploy [Nginx](https://www.nginx.com/) to Docker, and then destroy it.

## Prerequisites

- [Terraform v1.0.11](https://www.terraform.io/downloads) or newer
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) 

## Create the configuration

Create a directory for your configuration named `terraform-demo`.

```shell
$ mkdir terraform-demo
```

Change into the directory.

```shell
$ cd terraform-demo
```

Create a new file for your configuration named `main.tf`.

```shell
$ touch main.tf
```

Open `main.tf` in your text editor and paste the following configuration into it.

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.22.0"
    }
  }
}

provider "docker" {
  host = "unix:///var/run/docker.sock"
}

resource "docker_image" "nginx_img" {
  name = "nginx:latest"
}

resource "docker_container" "nginx_container" {
  image = docker_image.nginx_img.image_id
  name  = "training"
  ports {
    internal = 80
    external = 8080
  }
}
```

## Review the configuration

The `terraform` block tells Terraform that you will use the [Docker provider](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs) as well as the version to use. The `provider` block configures the Docker provider. In this case, Terraform will connect to the local Docker host at `unix:///var/run/docker.sock`. Finally, this configuration creates two resources.

The first resource is a `docker_image` named `nginx_img`. This resource pulls the `nginx:latest` container image from Docker Hub.

The second resource is a `docker_container` named `nginx_container` and creates the container in Docker. It references the previously defined `docker_image` with the value of `docker_image.nginx_img.image_id`. It also maps port `80` in the container to port `8080` on the Docker host.

## Prepare the configuration

Before applying your configuration, you must initialize it with the `terraform init` command. This command downloads and installs the required provider.

```shell
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 2.22.0"...
- Installing kreuzwerker/docker v2.22.0...
- Installed kreuzwerker/docker v2.22.0 (self-signed, key ID BD080C4571C6104C)

#...
```

You may also validate the syntax of your configuration by running `terraform validate`.

```shell
$ terraform validate
Success! The configuration is valid.
```

## Apply the configuration

Apply your configuration with the `terraform apply` command. Terraform will list the steps that it will take, known as an _execution plan_. We recommend you review this output for accuracy. Notice that some values aren't known until after you apply the configuration.


```shell
$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx_container will be created
  + resource "docker_container" "nginx_container" {
      # ...
      + name         = "training"

      + ports {
          + external = 8080
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx_img will be created
  + resource "docker_image" "nginx_img" {
      + id          = (known after apply)
      + image_id    = (known after apply)
      + latest      = (known after apply)
      + name        = "nginx:latest"
      + output      = (known after apply)
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.
```

After reviewing the plan, enter `yes` to continue.

```shell
Enter a value: yes

docker_image.nginx_img: Creating...
docker_image.nginx_img: Creation complete after 6s [id=sha256:2d389e545974d4a93ebdef09b650753a55f72d1ab4518d17a30c0e1b3e297444nginx:latest]
docker_container.nginx_container: Creating...
docker_container.nginx_container: Creation complete after 1s [id=361e0604a67ab66cbd9a2cd2e697fe8c8b6edc6d0ce263d160d16c034489ef82]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```


Your infrastructure is up and running! You can reach Nginx in your web browser at http://localhost:8080.

## Destroy the infrastructure

To remove the resources created by Terraform, use the `terraform destroy` command.

```shell
$ terraform destroy
docker_image.nginx_img: Refreshing state... [id=sha256:2d389e545974d4a93ebdef09b650753a55f72d1ab4518d17a30c0e1b3e297444nginx:latest]
docker_container.nginx_container: Refreshing state... [id=361e0604a67ab66cbd9a2cd2e697fe8c8b6edc6d0ce263d160d16c034489ef82]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx_container will be destroyed
  - resource "docker_container" "nginx_container" {
      # ...
      - name         = "training" -> null

      - ports {
          - external = 8080 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx_img will be destroyed
  - resource "docker_image" "nginx_img" {
      - id          = "sha256:2d389e545974d4a93ebdef09b650753a55f72d1ab4518d17a30c0e1b3e297444nginx:latest" -> null
      - image_id    = "sha256:2d389e545974d4a93ebdef09b650753a55f72d1ab4518d17a30c0e1b3e297444" -> null
      - latest      = "sha256:2d389e545974d4a93ebdef09b650753a55f72d1ab4518d17a30c0e1b3e297444" -> null
      - name        = "nginx:latest" -> null
      - repo_digest = "nginx@sha256:0b970013351304af46f322da1263516b188318682b2ab1091862497591189ff1" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value:
```

Terraform once again displays the plan that it will execute. Here, Terraform shows it will destroy both the `docker_container` and `docker_image`. Enter `yes` to continue.

```shell
Enter a value: yes

docker_container.nginx_container: Destroying... [id=361e0604a67ab66cbd9a2cd2e697fe8c8b6edc6d0ce263d160d16c034489ef82]
docker_container.nginx_container: Destruction complete after 0s
docker_image.nginx_img: Destroying... [id=sha256:2d389e545974d4a93ebdef09b650753a55f72d1ab4518d17a30c0e1b3e297444nginx:latest]
docker_image.nginx_img: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

## Next steps

In this guide, you learned how to write a Terraform configuration, use it to provision your infrastructure, and how to destroy it. You also learned how to validate that the syntax of your configuration is correct.

For more information on Terraform:

- Reference the [Terraform documentation](https://www.terraform.io/docs)
- Learn more about [Terraform's configuration language](https://www.terraform.io/language)
- Read the [Docker provider documentation](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs)
- See more [Terraform tutorials](https://learn.hashicorp.com/terraform)