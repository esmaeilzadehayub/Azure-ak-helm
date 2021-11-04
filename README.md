# Azure_aks_helm

Define AKS cluster with Terraform


Introduction
This written Infra as Code (IaC) workshop show how to create AKS cluster using Hashicorp Terraform
Azure Kubernetes Service (AKS) is a highly available, secure, and fully managed Kubernetes service of Microsoft Azure.
The fully managed Azure Kubernetes Service (AKS) makes deploying and managing containerized applications easy. It offers serverless Kubernetes, an integrated continuous integration and continuous delivery (CI/CD) experience, and enterprise-grade security and governance. Unite your development and operations teams on a single platform to rapidly build, deliver, and scale applications with confidence.
Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions.
Admitting, there are number of Cloud provisioning IaC tools, each with its own implementation. With the object of this lab, We will focus exclusively on deploying AKS cluster on Azure.

The directory that holds the Terraform configuration files for this lab has a special tree structure.
Obviously, there are 2 main subfolders: deployment and modules. In order to see the source code structure you can run tre
![image](https://user-images.githubusercontent.com/28998255/139659002-35422604-1583-45d7-986c-337909c36d71.png)

Project structure
1- modules: represent here in this layout the Terraform modules (general re-used functions) . In this lab, we have basically 4 modules:
- aks_cluster: the main unit providing the AKS service
- aks_identities: the cluster identity unit that manage the cluster service principal
- aks_network: Create the cluster Virtual Network and subnetwork on Azure
- log_analytics: Formally Azure Operational Insight is the unit that manages logs and cluster health checks

2- Deployment: is the main function of this layout, responsible of the AKS Kubernetes cluster deployment on Azure.
In main.tf we define the Terraform modules already created in /modules sub-folder with the appropriate inputs defined in variables.tf or in a terraform.tfvars file (wich is not covered in this guide).
Stay tuned, in the next section, we’re going to talk about how to create reusable infrastructure with Terraform.

Terraform modules

With Terraform, you can put a bunch of code inside of a Terraform module and reuse that module in multiple places throughout your code. Instead of having the same code copy/pasted in the staging and production environments, you’ll be able to have both environments reuse code from the same module.

This is a big deal. Modules are the key ingredient to writing reusable, maintainable, and testable Terraform code.
Every Terraform configuration has at least one module, known as its root module (the /deployment in this lab context), which consists of the resources defined in the .tf files in the main working directory.When we run terraform apply, Terraform deploys the Helm release and runs the tests. By using Terraform to deploy the Helm release,
we can pass attributes from infrastructure resources to the curated application definition in Helm and run available acceptance tests in a single, common workflow.

# kubernetes_ingress

Ingress is a collection of rules that allow inbound connections to reach the endpoints defined by a backend. 
An Ingress can be configured to give services externally-reachable urls, load balance traffic, terminate SSL, offer name based virtual hosting etc.
``` yml
resource "kubernetes_ingress" "example_ingress" {
  metadata {
    name = "example-ingress"
  }

  spec {
    backend {
      service_name = "MyApp1"
      service_port = 8080
    }

    rule {
      http {
        path {
          backend {
            service_name = "MyApp1"
            service_port = 8080
          }

          path = "/app1/*"
        }

        path {
          backend {
            service_name = "MyApp2"
            service_port = 8080
          }

          path = "/app2/*"
        }
      }
    }

    tls {
      secret_name = "tls-secret"
    }
  }
}

resource "kubernetes_pod" "example" {
  metadata {
    name = "terraform-example"
    labels = {
      app = "MyApp1"
    }
  }

  spec {
    container {
      image = "nginx:1.7.9"
      name  = "example"

      port {
        container_port = 8080
      }
    }
  }
}

resource "kubernetes_pod" "example2" {
  metadata {
    name = "terraform-example2"
    labels = {
      app = "MyApp2"
    }
  }

  spec {
    container {
      image = "nginx:1.7.9"
      name  = "example"

      port {
        container_port = 8080
      }
    }
  }
}
```

# Managing Helm Charts via Terraform

For services packaged with Helm, we can also use Terraform to deploy charts and run tests.
Helm provides application definitions in the form of charts. Services or applications often have official charts for streamlining deployment. 
For example, we might want to use Consul, a service mesh that provides a key-value store, to connect applications and manage configuration in our Kubernetes cluster.

We can use the official Consul Helm chart, which packages the necessary Consul application definitions for deployment.
When using Helm directly, we would first deploy a component called Tiller for version 2 of Helm. 
Then, we would store the Consul chart locally, deploy the chart with helm install, and test the deployment with helm test.

When using Terraform Helm provider, the provider will handle deployment of Tiller, installation of a Consul cluster via the chart, and triggering of acceptance tests. First, we include an option to install_tiller with the Helm provider
```yml
provider "helm" {
  version        = "~> 0.9"
  install_tiller = true
}
```
Next, we use the Terraform helm_release resource to deploy the chart.We pass the variables to the Helm chart with set blocks.

We also include a provisioner to run a set of acceptance tests after deployment, using helm test. The acceptance tests confirm if Consul is ready for use.
```yml
resource "helm_release" "consul" {
  name      = var.name
  chart     = "${path.module}/consul-helm"
  namespace = var.namespace

  set {
    name  = "server.replicas"
    value = var.replicas
  }

  set {
    name  = "server.bootstrapExpect"
    value = var.replicas
  }

  set {
    name  = "server.connect"
    value = true
  }

  provisioner "local-exec" {
    command = "helm test ${var.name}"
  }
}
```


