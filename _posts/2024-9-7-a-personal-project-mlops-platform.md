---
layout: post
title: A Personal Project MLOps Platform'
date: 2024-12-02 13:11:00
description: A Basic MLOps Setup With Kubeflow
tags: mlops, kubeflow, terraform
categories: mlops
featured: true
---

## Build A Kubeflow MLOps Platform With Terraform

### Inspiration
Machine Learning Engineers need to be able to deploy infrastructure to facilitate develop, train and deploy models as efficiently as possible. Since before the invention of transformers machine learning has depended on quick iteration cycles and feedback loops. But when machine learning teams scale, the quickness of iteration is stimied by reacting to the needs of individual models and problem areas. Some models require distributed GPU clusters, others can be experimented on single CPU. Some have hard-to-configure dependency environments which must be shared seamlessly between scientists. Model serving creates another large layer of complexity, with batch vs online serving, event-driven vs response-based, and CICD requirements. 

TODO - add CRISP cycle image

An ideal MLOps solution will allow teams to manage the varying components of the machine learning cycle without jumping between different interfaces and solutions. It also would allow flexibility to tackle brand new configurations, such as switching to event-driven serving or adding a batch processing solution. For these reasons, Kubernetes[3] stands out as a solution and is widely used across top tier tech companies and AI labs. Just prior to releasing ChatGPT, OpenAI described [scaling a Kubernetes cluster to 7500 nodes](https://openai.com/index/scaling-kubernetes-to-7500-nodes/).

### Why A Kubernetes-based Solution
Why not a full-service ML platform like [Domino Data Labs](https://domino.ai/), H2O.ai, or even [Sagemaker Studio](https://aws.amazon.com/sagemaker/studio/)? Kubernetes lets ML Engineering teams scale on their own terms and best tailor their solutions to their own needs. Conversely, ML platforms lock users into their format and then annual subscription fees. At its a best, an internally-built containerized solution can be designed to be multi-cloud and avoid vendor lock-in. Finally, the advent of generative AI means organizations are experimenting with workflows which may have needs beyond what traditional ML platforms support. In order to fully empower AI teams to innovate, teams can give their engineers access to theoretically limitless compute and high flexibility. 

### Starting Out 
Terraform is the most popular and versatile Infrastructure as Code (IaC) language. Importantly, its format can be used across cloud vendors. A basic Google Cloud 'Google Kubernetes Engine' (GKE) cluster is shown below. It initializes the GKE cluster with a single node, and provides a workload itendifier, so each node in the cluster can access cloud resources without needing to individually authenticate. 

```tf
# GKE Cluster 
resource "google_container_cluster" "primary" {
    name = local.cluster_name
    location = local.gcp_region

    remove_default_node_pool = true
    initial_node_count = 1 

    # GKE Version
    min_master_version = local.gke_version

    # Workload Identity
    workload_identity_config {
       workload_pool = "${local.project_id}.svc.id.goog"
    }    
}

```

[See full file here](TODO - add github link)

Running `terraform plan` will demonstrate the attributes of this stack. It should describe a simple cluster. Next, to deploy this cluster to our GCP project, we can run `terraform apply`. If this is a new project, we'll need to enable the GKE API for the project and also link the project to a billing account. 

### Picking An ML Kubernetes Framework
Kubeflow, Metaflow, and Bodywork are potential ML Frameworks which run on Kubernetes. Kubeflow has multiple advantages. First, it is well-supported as it initially was developed Google and is now maintained by the Cloud Native Computing Foundation. Another strength is its thoroughness and customization. Kubeflow offers specialized Kubernetes operators [4] for Notebooks, Workflows and Schedules, Training, Model Tuning, Serving, Model registry and a central monitoring dashboard. These custom operators also provide a customization in that they allow users to device which components are necessary at a given time. This makes Kubeflow adjustable to small projects or enterprise scale, as opposed to say Metaflow which is more of an all-in-one solution meant for a large scale project. 

### Installing Kubeflow
Kubernetes Deployments require Objects [10] which include a *spec* which describe the desired state of your application. Kubernetes works to ensure your application matches the spec,and uses the *status* to describe how the application is actually running. That spec is defined via a config file (usually YAML) called a *manifest*. Kubernetes offers a myriad of ways to install packages. Individual Kubeflow components can be installed via the [Kubeflow manifests repository](https://github.com/kubeflow/manifests). Kubeflow allows you to install the entire platform or individual components as needed. For sake of example, we'll install the training component. At time of writing I can install the stable release of the training componenent with the kubectl cli:
1. Authenticate our cluster with glcoud cli
```bash
gcloud container clusters get-credentials kubeflow-mlops     --region=us-central1 
```
2. Apply kubeflow training-operator manifest to our cluster
```bash 
kubectl apply -k "github.com/kubeflow/training-operator.git/manifests/overlays/standalone?ref=v1.7.0"
```
The output should look something like this 
```
namespace/kubeflow created
customresourcedefinition.apiextensions.k8s.io/mpijobs.kubeflow.org created
customresourcedefinition.apiextensions.k8s.io/mxjobs.kubeflow.org created
customresourcedefinition.apiextensions.k8s.io/paddlejobs.kubeflow.org created
customresourcedefinition.apiextensions.k8s.io/pytorchjobs.kubeflow.org created
customresourcedefinition.apiextensions.k8s.io/tfjobs.kubeflow.org created
customresourcedefinition.apiextensions.k8s.io/xgboostjobs.kubeflow.org created
serviceaccount/training-operator created
clusterrole.rbac.authorization.k8s.io/training-operator created
clusterrolebinding.rbac.authorization.k8s.io/training-operator created
service/training-operator created
deployment.apps/training-operator created
```

### Defining A Training Job
Thanks to our Kubeconfig file (created when we authenticated above) we can define a Python training job locally and run it on our distributed Kubernetes cluster. 

### Creating A Workflow
A Deployment 

### Creating Seperate Node Pools

### Wrapping Up

### Resources

[1][Spotify's cluster](https://github.com/spotify/terraform-gke-kubeflow-cluster)
[2] [Combinator.ml](https://combinator.ml/)
[3] [Kubernetes](https://kubernetes.io/docs/concepts/overview/)
[4] [Kubeflow Componenets](https://www.kubeflow.org/docs/components/)
[5] [Metaflow](https://metaflow.org/)
[6] [Bodywork](https://github.com/bodywork-ml/bodywork-core)
[7] [Kubernetes Manifest](https://kubernetes.io/docs/concepts/workloads/management/)
[8] [OpenAI Scaling Kubernetes](https://openai.com/index/scaling-kubernetes-to-7500-nodes/)
[9] [OpenAI Case Study Kubernetes](https://kubernetes.io/case-studies/openai/)
[10] [Kubernetes Objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/#kubernetes-objects)
[11] [Swiss Army Cube](https://github.com/provectus/sak-kubeflow)
[12] [Kubeflow on EKS](https://registry.terraform.io/modules/young-ook/eks/aws/1.7.6/examples/kubeflow)