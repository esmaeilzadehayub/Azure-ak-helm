# Azure-ak-helm

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
Every Terraform configuration has at least one module, known as its root module (the /deployment in this lab context), which consists of the resources defined in the .tf files in the main working directory.
