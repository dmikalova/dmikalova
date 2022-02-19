# David Mikalova

[![résumé](https://img.shields.io/static/v1?&color=ccff90&label=&labelColor=424242&logo=libreoffice&logoColor=fff&message=résumé&&style=flat-square)](./resume.pdf)
[![linkedin](https://img.shields.io/static/v1?&color=2867b2&label=&labelColor=424242&logo=linkedin&logoColor=fff&message=linkedin&&style=flat-square)](https://github.com/dmikalova)

I am a cloud infrastructure engineer with a passion for automation and internal developer platforms. My free time is spent working on a private Kubernetes cluster for running homegrown web apps and other compute tasks with the ultimate goal of writing an open source music streaming server.

I have organized my code under separate Github organizations: [e91e63](https://github.com/e91e63) for infrastructure repositories and [cddc39](https://github.com/cddc39) for web app repositories - all of which are deployed through Terraform.

## Repositories

### [Infrastructure configuration](https://github.com/dmikalova/infrastructure)

Terragrunt configurations that define all of my deployed resources. Includes a Kubernetes cluster in DigitalOcean with routing, certificate rotation, CI/CD pipeline, and web apps. Terragrunt is used to DRY Terraform modules and configuration.

### [DigitalOcean Kubernetes cluster](https://github.com/e91e63/terraform-digitalocean-kubernetes)

Terraform code for deploying a Kubernetes cluster in DigitalOcean. Includes modules for a private container registry and default service account configuration to use the container registry.

### [Kubernetes manifests](https://github.com/e91e63/terraform-kubernetes-manifests)

Kubernetes manifests managed through Terraform. I use Terraform instead of YAML to create an immutable workflow for managing my Kubernetes cluster.

### [Tekton CI/CD pipeline](https://github.com/e91e63/terraform-tekton-pipelines)

Reusable CI/CD pipelines built with Tekton's Custom Resource Definitions. Webhooks are added to Github repositories by Terraform based on pipeline tags. JavaScript pipeline runs unit and end-to-end tests, builds a container, and continuously deploys on passing tests.

### [Lists web app](https://github.com/cddc39/lists)

In-progress Vue.js app for managing lists of things.

### [Brocket window manager](https://github.com/dmikalova/brocket)

A run-or-raise window manager for Linux written in Go. Useful for keyboard shortcuts to open, switch to, or resize windows.
