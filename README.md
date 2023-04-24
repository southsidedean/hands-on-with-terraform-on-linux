# Getting Started with Terraform

Infrastructure as code (IaC) enables you to create, manage, provision and destroy infrastructure, programmatically and on demand. IaC, a key pillar in the culture of DevOps, offers several advantages to manual operations.

Benefits of IaC:

- Reduced cost, fewer human cycles needed
- Reduced deployment time
- Reduced risk through automation

An excellent introduction to IaC can be found in the HashiCorp Developer [What is Infrastructure as Code with Terraform?](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/infrastructure-as-code) tutorial.

When selecting tooling to deploy IaC, you have a number of options that can be used standalone, or together, to deliver end-to-end infrastructure as code.

Several Popular IaC Tools:

- HashiCorp Terraform
- Red Hat Ansible
- Puppet
- Chef
- SaltStack
- AWS CloudFormation

Terraform, a popular infrastructure as code tool, facilitates deployment of IaC across a variety of infrastructure providers. In this guide, you're going to take a look at how to install Terraform, create some infrastructure as code, deploy that code on Docker and destroy the infrastructure we created, using Terraform.

Learning Objectives:

- Install and test Terraform
- Define infrastructure as code (IaC) using Terraform
- Deploy infrastructure as code (IaC) using Terraform
- Destroy infrastructure as code (IaC) using Terraform

## Prerequisites

Supported Operating System:

- [Terraform.io](https://www.terraform.io/downloads.html)

Docker:

- [Install Docker Engine](https://docs.docker.com/engine/install/)

*The examples in this guide come from an Ubuntu Linux 22.04 operating system environment.*

## Install Docker (If Needed)

The Terraform code you're using in this guide uses the `kreuzwerker/docker` infrastructure, which means that you will need to have Docker installed on your system. If you don't have Docker installed, follow the instructions in the [Install Docker Engine](https://docs.docker.com/engine/install/) guide for your operating system.

After installing Docker, don't forget to add your user to the `docker` group. The following example shows how to do that on Linux.

```shell
$ sudo usermod -aG docker `whoami`
```

This will give your user elevated access to run `docker` commands. You'll need to log out and log back in for this change to take effect.

*Don't forget to test your Docker installation, per the Docker installation instructions!*

## Install Terraform

Before you can use Terraform, you'll need to install the software. In this step, we'll provide some resources to help you install Terraform in your environment.

The Terraform installation process will vary, depending on your operating system. Follow the instructions for your operating system in the HashiCorp Developer [Install Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) or the [Terraform.io](https://www.terraform.io/downloads.html) tutorials.

## Test Your Terraform Installation

Once you've installed Terraform, you should check your work. In this step, you'll test your Terraform installation.

To validate your Terraform installation, run the `terraform -version` command.

```shell
$ terraform -version

Terraform v1.4.5
on linux_amd64
```

The `terraform -version` command shows you the installed version of Terraform.

## Create Infrastructure as Code With Terraform

With Terraform installed and working, it's time to create some infrastructure as code. In this step, you'll create a file with a Terraform configuration that uses the `kreuzwerker/docker` provider to provision an `nginx` container on Docker.

Terraform requires a working directory to host your `demo` project. You should create a directory called `terraform-demo` and change your working directory to that directory.

```shell
$ mkdir terraform-demo
$ cd terraform-demo
```

More information on Terraform working directories can be found on the HashiCorp Developer site, in the Terraform CLI documentation, [here](https://developer.hashicorp.com/terraform/cli/init).

Next, create a file with your Terraform code.

```shell
$ cat <<EOF | tee main.tf
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}
provider "docker" {
  host = "unix:///var/run/docker.sock"
}
resource "docker_container" "nginx" {
  image = docker_image.nginx.name
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
resource "docker_image" "nginx" {
  name = "nginx:latest"
}
EOF
```

The previous command will give you a file, `main.tf`, with the following contents.

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}
provider "docker" {
  host = "unix:///var/run/docker.sock"
}
resource "docker_container" "nginx" {
  image = docker_image.nginx.name
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```

Terraform uses this code to provision our infrastructure.

## Deploy Infrastructure as Code Using Terraform

So, you have a Terraform configuration. What next?

In this step, you're going to use the Terraform software you installed to provision an `nginx` container, in Docker, using the Terraform configuration you created in the previous step.

First, check for `nginx` containers, using the `docker` command.

```shell
$ docker ps -a

CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
72c99ddec658   hello-world   "/hello"   9 minutes ago    Exited (0) 9 minutes ago              boring_villani
```

The preceding example shows no `nginx` containers, running or otherwise.

Before you can deploy any infrastructure as code, you need to initialize your Terraform project directory with the `terraform init` command. This will download and install the `kreuzwerker/docker` provider and set up your `terraform-demo` directory with the necessary resources to execute your `demo` project.

```shell
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of kreuzwerker/docker...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (self-signed, key ID BD080C4571C6104C)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Next, validate your Terraform configuration, and see what you're building, using the `terraform plan` command.

```shell
$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach                                      = false
      + bridge                                      = (known after apply)
      + command                                     = (known after apply)
      + container_logs                              = (known after apply)
      + container_read_refresh_timeout_milliseconds = 15000
      + entrypoint                                  = (known after apply)
      + env                                         = (known after apply)
      + exit_code                                   = (known after apply)
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = "nginx:latest"
      + init                                        = (known after apply)
      + ipc_mode                                    = (known after apply)
      + log_driver                                  = (known after apply)
      + logs                                        = false
      + must_run                                    = true
      + name                                        = "training"
      + network_data                                = (known after apply)
      + read_only                                   = false
      + remove_volumes                              = true
      + restart                                     = "no"
      + rm                                          = false
      + runtime                                     = (known after apply)
      + security_opts                               = (known after apply)
      + shm_size                                    = (known after apply)
      + start                                       = true
      + stdin_open                                  = false
      + stop_signal                                 = (known after apply)
      + stop_timeout                                = (known after apply)
      + tty                                         = false
      + wait                                        = false
      + wait_timeout                                = 60

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + image_id    = (known after apply)
      + name        = "nginx:latest"
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```

Check for, and correct any errors, if needed. Run the `terraform plan` command again to validate as necessary.

Once the `terraform plan` command runs successfully, provision your resources with the `terraform apply` command. When asked for confirmation, type `yes` and hit ENTER.

```shell
$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach                                      = false
      + bridge                                      = (known after apply)
      + command                                     = (known after apply)
      + container_logs                              = (known after apply)
      + container_read_refresh_timeout_milliseconds = 15000
      + entrypoint                                  = (known after apply)
      + env                                         = (known after apply)
      + exit_code                                   = (known after apply)
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = "nginx:latest"
      + init                                        = (known after apply)
      + ipc_mode                                    = (known after apply)
      + log_driver                                  = (known after apply)
      + logs                                        = false
      + must_run                                    = true
      + name                                        = "training"
      + network_data                                = (known after apply)
      + read_only                                   = false
      + remove_volumes                              = true
      + restart                                     = "no"
      + rm                                          = false
      + runtime                                     = (known after apply)
      + security_opts                               = (known after apply)
      + shm_size                                    = (known after apply)
      + start                                       = true
      + stdin_open                                  = false
      + stop_signal                                 = (known after apply)
      + stop_timeout                                = (known after apply)
      + tty                                         = false
      + wait                                        = false
      + wait_timeout                                = 60

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + image_id    = (known after apply)
      + name        = "nginx:latest"
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 6s [id=sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 3s [id=40afc3f5bcd2f53ddd4096c1efc52f65e9334bda4855aa4460c53f99083e273a]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

The `terraform-apply` command could take anywhere from a few seconds to a few minutes to run. When complete, you will get a message confirming the creation of the resources.

Use the `docker` command to check your results.

```shell
$ docker ps -a

CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                      PORTS                NAMES
40afc3f5bcd2   nginx:latest   "/docker-entrypoint.…"   51 seconds ago   Up 49 seconds               0.0.0.0:80->80/tcp   training
```

You should see the `nginx` container, named `training`, as shown in the previous example.

We can use `curl` to test as well, connecting to our container, via HTTP, on port 80.

```shell
$ curl http://localhost:80

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

You should get the stock "Welcome to nginx!" index page.

*Great job! You just deployed an `nginx` container, in Docker, using Terraform and IaC!*

## Destroy Infrastructure as Code Using Terraform

Terraform makes tearing down infrastructure as code *even easier* than deploying it. In this step, we're going to use Terraform to dismantle our Docker deployment.

Use `terraform destroy` to destroy the infrastructure. When asked for confirmation, type `yes` and hit ENTER. Terraform will destroy the resources it had created earlier.

```shell
$ terraform destroy
docker_image.nginx: Refreshing state... [id=sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest]
docker_container.nginx: Refreshing state... [id=40afc3f5bcd2f53ddd4096c1efc52f65e9334bda4855aa4460c53f99083e273a]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
      - attach                                      = false -> null
      - command                                     = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - container_read_refresh_timeout_milliseconds = 15000 -> null
      - cpu_shares                                  = 0 -> null
      - dns                                         = [] -> null
      - dns_opts                                    = [] -> null
      - dns_search                                  = [] -> null
      - entrypoint                                  = [
          - "/docker-entrypoint.sh",
        ] -> null
      - env                                         = [] -> null
      - group_add                                   = [] -> null
      - hostname                                    = "40afc3f5bcd2" -> null
      - id                                          = "40afc3f5bcd2f53ddd4096c1efc52f65e9334bda4855aa4460c53f99083e273a" -> null
      - image                                       = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1f" -> null
      - init                                        = false -> null
      - ipc_mode                                    = "private" -> null
      - log_driver                                  = "json-file" -> null
      - log_opts                                    = {} -> null
      - logs                                        = false -> null
      - max_retry_count                             = 0 -> null
      - memory                                      = 0 -> null
      - memory_swap                                 = 0 -> null
      - must_run                                    = true -> null
      - name                                        = "training" -> null
      - network_data                                = [
          - {
              - gateway                   = "172.17.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.17.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - mac_address               = "02:42:ac:11:00:02"
              - network_name              = "bridge"
            },
        ] -> null
      - network_mode                                = "default" -> null
      - privileged                                  = false -> null
      - publish_all_ports                           = false -> null
      - read_only                                   = false -> null
      - remove_volumes                              = true -> null
      - restart                                     = "no" -> null
      - rm                                          = false -> null
      - runtime                                     = "runc" -> null
      - security_opts                               = [] -> null
      - shm_size                                    = 64 -> null
      - start                                       = true -> null
      - stdin_open                                  = false -> null
      - stop_signal                                 = "SIGQUIT" -> null
      - stop_timeout                                = 0 -> null
      - storage_opts                                = {} -> null
      - sysctls                                     = {} -> null
      - tmpfs                                       = {} -> null
      - tty                                         = false -> null
      - wait                                        = false -> null
      - wait_timeout                                = 60 -> null

      - ports {
          - external = 80 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id          = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest" -> null
      - image_id    = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1f" -> null
      - name        = "nginx:latest" -> null
      - repo_digest = "nginx@sha256:63b44e8ddb83d5dd8020327c1f40436e37a6fffd3ef2498a6204df23be6e7e94" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

docker_container.nginx: Destroying... [id=40afc3f5bcd2f53ddd4096c1efc52f65e9334bda4855aa4460c53f99083e273a]
docker_container.nginx: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest]
docker_image.nginx: Destruction complete after 1s

Destroy complete! Resources: 2 destroyed.
```

Upon completion of the `terraform destroy` process, you'll see the `nginx` container and the `nginx` container image destroyed.

Use the `curl` command to confirm that the `nginx` container has been deleted.

```shell
$ curl http://localhost:80

curl: (7) Failed to connect to localhost port 80 after 0 ms: Connection refused
```

The NGINX container no longer responds to our `curl` connection.

Use the `docker` command to confirm the NGINX container's destruction.

```shell
$ docker ps -a

CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
72c99ddec658   hello-world   "/hello"   22 minutes ago   Exited (0) 22 minutes ago             boring_villani
```

You should no longer see the `nginx` container, named `training`, as shown in the previous step.

Use the `docker images` command to confirm the removal of the `nginx` image.

```shell
$ docker images

REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    feb5d9fea6a5   19 months ago   13.3kB
```

You should no longer see the `nginx` container image in the local registry.

Congratulations, you just used Terraform and infrastructure as code (IaC) to destroy your Docker infrastructure!

## Next Steps

In this guide, you experienced the value of infrastructure as code (IaC), through deploying an `nginx` container, in Docker, using Terraform and IaC.

You got hands-on with Terraform and performed the following activities:

- Installed Terraform and tested your Terraform installation
- Defined infrastructure as code (IaC) using Terraform
- Deployed infrastructure as code (IaC) using Terraform
- Destroyed infrastructure as code (IaC) using Terraform

Through these mini-exercises, you realized the power of IaC, using Terraform!

For more information on Terraform, see:

- [Terraform: Infrastructure as Code](https://www.hashicorp.com/products/terraform)
- [HashiCorp Developer: Terraform: Documentation](https://developer.hashicorp.com/terraform/docs)
- [HashiCorp Developer: Terraform: Tutorials](https://developer.hashicorp.com/terraform/tutorials)
- [HashiCorp Terraform Registry](https://registry.terraform.io/)

