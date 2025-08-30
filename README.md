![AWS](https://img.shields.io/badge/AWS-EKS-orange?logo=amazon-aws&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Cluster-326ce5?logo=kubernetes&logoColor=white)
![CloudFormation](https://img.shields.io/badge/IaC-CloudFormation-FF4F8B?logo=amazon-aws&logoColor=white)
![Status](https://img.shields.io/badge/Project-Completed-brightgreen)
![License](https://img.shields.io/badge/License-MIT-blue)

<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Launch a Kubernetes Cluster

**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-compute-eks1)

**Author:** Bradley Davel  
**Email:** bradley.davel@outlook.com

---

## Launch a Kubernetes Cluster

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-compute-eks1_e5f6g7h8)

---

## Introducing Today's Project!

In this project, I will launch and manage a Kubernetes cluster on AWS using EKS, because I want to gain hands-on experience with container orchestration in a real cloud environment.

By setting up an EKS cluster, monitoring it with CloudFormation, and configuring IAM access, I’ll strengthen my DevOps skills in cluster management, scalability, and resilience testing. This project directly ties to what companies expect from a DevOps engineer—being able to deploy, secure, and maintain highly available infrastructure in production.

### What is Amazon EKS?

### One thing I didn't expect

### This project took me...

---

## What is Kubernetes?

Kubernetes is a container orchestration platform, which is a fancy way to say that it coordinates containers so they're running smoothly across all your servers. It makes sure all your containers are running where they should, scales containers automatically to meet demand levels, and even restarts containers if something crashes.

It’s THE standard tool for keeping large, container-based applications steady and easy to scale with traffic. That's why big tech companies, startups, and developers worldwide use Kubernetes.

Without a tool like Kubernetes, you would create and manage every container manually. You’d have to start each container yourself and keep an eye on them to restart any that crash.

Traffic to your app going up or down would mean turning containers on or off one by one, and you’d also have to make sure each container has access to storage if it needs it. Updating your app would mean carefully swapping out containers without causing downtime.

I used eksctl to automatically provision my EKS cluster without having to manually configure all the CloudFormation templates, IAM roles, and networking.

The create cluster command I ran defined:

-Cluster name → to identify the EKS cluster.
-Region → where the cluster resources would live.
-Managed node group settings → number of worker nodes, min/max scaling values, and instance types.
-IAM role attachment → so the EC2 instance could talk to EKS and CloudFormation.

This single command spun up the cluster, node group, and all required resources, saving me time and reducing the chance of manual errors.

I initially ran into two errors while using eksctl:

-The first one was because the tool wasn’t installed (or not on my PATH). I saw eksctl: command not found. I fixed it by installing eksctl and verifying with eksctl version.

-The second one was because my EC2 instance didn’t have an IAM instance profile with the permissions eksctl needs. CloudFormation/EKS calls failed with AccessDenied/UnauthorizedOperation (e.g., eks:CreateCluster, cloudformation:CreateStack, iam:PassRole). I fixed it by attaching an IAM role to the instance with the required policies (e.g., AmazonEKSClusterPolicy, CloudFormationFullAccess, AmazonEC2ContainerRegistryReadOnly, plus nodegroup policies if creating managed nodes), then re-ran eksctl create cluster.

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-compute-eks1_ff9bfc221)

---

## eksctl and CloudFormation

CloudFormation helped create my EKS cluster because eksctl uses CloudFormation templates under the hood to provision all the required AWS resources automatically.

It created VPC resources because an EKS cluster needs networking to run — things like subnets, route tables, and security groups. CloudFormation also built the control plane, IAM roles, and node group stacks, ensuring everything was consistent and linked together without me having to manually configure each resource.

There was also a second CloudFormation stack.The second stack is specifically for your node group, which is a group of EC2 instances that will run your containers.

CloudFormation separates the core EKS cluster stack from the node group stack to make it easier to manage and troubleshoot each part independently, especially if one half fails. The difference between a cluster and a node group is that:

The cluster is the control plane — it manages Kubernetes resources, API requests, scheduling, and overall orchestration. In EKS, AWS runs the control plane for you.

A node group is the set of worker EC2 instances that actually run your pods and workloads. The cluster tells the node group what to run, and the nodes provide the compute power.

In short: the cluster manages, the node group executes.

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-compute-eks1_w3e4r5t6)

---

## The EKS console

I had to create an IAM access entry in order to let my EC2 instance role (or IAM identity) authenticate and interact with the Kubernetes cluster.

An access entry is the modern way in EKS to map an IAM principal (user or role) to Kubernetes RBAC permissions, replacing the older aws-auth ConfigMap method.

I set it up by running the AWS CLI command aws eks create-access-entry with my EC2 instance role ARN, then associating an access policy such as AmazonEKSClusterAdminPolicy. This granted my instance admin-level access to the cluster so I could run kubectl commands successfully.

It took about 20–25 minutes to create my cluster. Since I'll create this cluster again in the next project of this series, maybe this process could be sped up if I used a smaller node group, reused an existing VPC, or created the cluster with a predefined eksctl config file instead of default settings. That way CloudFormation would have fewer resources to provision, which shortens the setup time.

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-compute-eks1_e5f6g7h8)

---

## EXTRA: Deleting nodes

Did you know you can find your EKS cluster's nodes in Amazon EC2? This is because the worker nodes in a Kubernetes cluster are actually EC2 instances launched as part of a node group. They appear in the EC2 console since AWS provisions them like any other virtual machine, but they’re configured to connect to the EKS control plane and run your Kubernetes workloads.

The desired size is the number of nodes you want running in your node group.

The minimum size is the minimum number of nodes your group needs to have to keep your app available at all times (even in low-demand periods).

The maximum size is the maximum number of nodes that you'll allow inside this group. This also lets EKS scale up your node group from the desired size in high-demand periods.

When I deleted my EC2 instances, Kubernetes automatically rescheduled the pods onto the remaining nodes or launched new nodes in the managed node group. This is because Kubernetes and EKS are designed with self-healing and desired state management - the cluster notices that the node is gone and ensures the workloads are restored to maintain the defined replica count.

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-compute-eks1_q7r8s9t0)

---

---
