**Author:** Edith Forestal
**GitHub:** [elforestal](https://github.com/elforestal)
**LinkedIn:** [linkedin.com/in/forestal](https://linkedin.com/in/forestal)

<img width="611" height="819" alt="image" src="https://github.com/user-attachments/assets/872dabe4-7e2b-48e8-8af9-c3f260e46922" />


---

## Introducing Today's Project!

In this project, I will use Terraform to automate the deployment of an AWS S3 bucket as infrastructure as code (IaC). 

The goal is to define cloud infrastructure in a configuration file, then create it with a repeatable command instead of clicking through the console. 

I'll also codify security into the deployment — the bucket includes a public-access block that prevents it from ever being exposed to the internet, which is secure-by-default infrastructure. 

I'm doing this project to build my DevSecOps and IaC skills, applying the security-first thinking from my earlier AWS projects to automated, version-controlled infrastructure. This is how security controls become part of the deployment pipeline rather than manual, easy-to-miss console steps.

**🧰 Services used**

- **Terraform** — infrastructure as code
- 🪣 **AWS S3** — object storage
- 🔐 **AWS IAM** — identity and access management
- ⌨️ **AWS CLI** — authenticating my local environment to AWS

**💡 Key concepts I learnt**

- 📄 **Infrastructure as code** — defining and version-controlling cloud resources in `main.tf` instead of clicking through the console
- 🔄 **init → plan → apply workflow** — a prepare, preview, then commit sequence that gates changes before they hit live infrastructure
- 🔒 **Secure-by-default configuration** — an S3 public access block with all four settings enabled, so the bucket is never exposed
- 🏷️ **Resource tagging** — `Environment = Dev` as the foundation for attribute-based access control (ABAC)
- 🗝️ **Credential hygiene** — generating access keys under a dedicated IAM Admin user instead of root

### Project reflection

This project took me approximately 2 hours. The most challenging part was getting Terraform onto my system PATH on Windows — the binary had to live in a folder that PATH actually pointed to, and the folder didn't exist yet, so the command kept failing until I created C:\terraform, moved the executable in, and opened a fresh terminal to reload PATH. 

It was most rewarding to see terraform apply build the S3 bucket and upload an object entirely from code, and to close the loop by verifying both in the S3 console — proving that the same declarative, version-controlled approach I'd write for security controls could stand up real infrastructure end-to-end.

I chose to do this project today because I'm moving my security background into cloud security and DevSecOps engineering, and Terraform is one of the core infrastructure-as-code tools those roles expect hands-on. Provisioning a secure-by-default S3 bucket from code — and framing it through least privilege, defense in depth, and credential hygiene — is exactly the kind of portfolio proof I'm building toward Senior Cloud Security Engineer roles. Something that would make learning with NextWork even better is clearer Windows-specific guidance in the Terraform install step — the PATH setup assumed the C:\terraform folder already existed and didn't mention creating it or moving the binary in, which is where I got stuck.

---

## Introducing Terraform

Terraform is an infrastructure-as-code (IaC) tool that lets you define cloud resources in a configuration file, then create and manage them automatically.

Instead of clicking through a console to set up servers, storage, or networks, you write what you want in code — and Terraform builds it for you.

It's cloud-agnostic, meaning the same tool works across AWS, Azure, and Google Cloud. You can even manage resources across all three from one config.

Its syntax is designed to read almost like plain English, so it's approachable to start with but powerful enough to model complex infrastructure.

Because the infrastructure is defined as code, it can be versioned, reviewed, and reused — which makes deployments consistent, repeatable, and easy to audit.

Infrastructure as Code (IaC) means defining cloud infrastructure in config files instead of setting it up manually.

You write what you want in code, and a tool like Terraform builds it.

Because it's code, it can be versioned, peer-reviewed, and redeployed the same way every time.

That makes deployments consistent and auditable — and lets you catch misconfigurations before they ever go live.

main.tf is the central configuration file in a Terraform project — it's where I declare what infrastructure I want AWS to build.

I write the desired state in Terraform's language (which resources, and how they should be configured), and Terraform reads this file as the blueprint for provisioning.

Keeping this definition in code is what makes the setup auditable and repeatable — I can version-control it, review changes before they hit live infrastructure, and stand up the same secure configuration again from scratch.

![Image](http://nextwork.ai/passionate_rose_serene_emblica/uploads/aws-devops-terraform1_9i0j1k2l)

---

## Configuration files

My configuration is defined in self-contained blocks inside main.tf.

Terraform stores its instructions as separate blocks, each doing a different job — a provider block tells Terraform which cloud to work with, and a resource block describes what to create and how it should look.

Keeping each piece of infrastructure in its own block keeps the file easy to read and lets me change one part without touching the rest — an approach called modularity.

### My main.tf configuration has three blocks

My main.tf file is split into three blocks, each describing a different part of the setup.

The provider block tells Terraform to use AWS, turning my configuration into the API calls that create and manage the infrastructure.

The resource "aws_s3_bucket" block creates the S3 bucket itself, with an internal name my_bucket that other blocks use to reference it.

The third block, resource "aws_s3_bucket_public_access_block", controls who can access the bucket — with block_public_acls, ignore_public_acls, block_public_policy, and restrict_public_buckets all set to true to prevent public access to the bucket and its contents.

![Image](http://nextwork.ai/passionate_rose_serene_emblica/uploads/aws-devops-terraform1_ljvh9876)

---

## Customizing my S3 Bucket

In the official Terraform documentation for aws_s3_bucket, I see the full reference for defining an S3 bucket as a resource.

It includes an example usage block showing the basic syntax, an argument reference listing every configurable parameter — like bucket, tags, and force_destroy — and an attribute reference showing the values Terraform exports after creation, such as the bucket's arn and id.

Reading this is what lets me customize my bucket deliberately rather than guessing — I can pull the exact arguments I need straight from the source, which is the same habit of working from authoritative documentation that keeps configurations correct and auditable.

I chose to customise my bucket by adding a tags block with Environment = "Dev", because tags are the foundation that attribute-based access control (ABAC) is built on — an IAM policy can grant or deny access based on a tag's value rather than naming resources individually, so labelling this bucket as a development resource is the hook that access control, cost allocation, and automation all hang off of. When I launch my bucket, I can verify my customization by opening the bucket in the S3 console and checking the Properties tab, where the Environment = Dev tag appears in the Tags section — confirming Terraform applied the label exactly as I defined it in main.tf.

![Image](http://nextwork.ai/passionate_rose_serene_emblica/uploads/aws-devops-terraform1_ffe757cd3)

---

## Terraform commands

I ran 'terraform init' to prepare my working directory before building any infrastructure.

This downloaded the AWS provider plugin that connects Terraform to my AWS environment, set up the backend where Terraform tracks the state of my infrastructure, and created a lock file that pins the exact provider version so the setup stays consistent across machines and teammates.

It didn't create any AWS resources itself — it just got the project ready so that terraform plan and terraform apply could run against a properly initialized directory.

Next, I ran 'terraform plan' to preview exactly what changes Terraform would make to my AWS environment before applying anything.

terraform plan builds an execution plan by comparing my configuration against the current state, showing what would be created, updated, or destroyed — in my case, the new S3 bucket and its public access block.

Planning first is the safeguard: it lets me review changes while they're still on paper and catch problems before they ever reach live infrastructure, rather than discovering them after the fact.

![Image](http://nextwork.ai/passionate_rose_serene_emblica/uploads/aws-devops-terraform1_3g4h5i6j)

---

## AWS CLI and Access Keys

When I tried to plan my Terraform configuration, I received an error message that says "No valid credential sources found" because Terraform had no AWS credentials to authenticate with.

To create resources in my account, Terraform needs to prove who it is to AWS — but I hadn't set up any access keys yet, so it had nothing to present.

It also fell back to checking for an EC2 instance role (IMDS) and found none, since I'm running Terraform locally on my own machine rather than on an EC2 instance — so with no keys and no instance role, it had no valid credential source and stopped before making any changes.

To resolve my error, first I installed AWS CLI, which is a command-line tool that lets me manage AWS services directly from my terminal instead of clicking through the Management Console.

In this project, the CLI is how I store my AWS credentials locally — when I run aws configure, it saves my access key so that Terraform can read those credentials and authenticate to AWS on my behalf.

That authenticated connection is exactly what was missing when my terraform plan failed, so installing and configuring the CLI is the fix for the "No valid credential sources found" error.

I set up AWS access keys to give Terraform a way to authenticate to my AWS account programmatically.
An access key is the CLI's method of proving identity — since the CLI and Management Console aren't connected, logging into the console doesn't authenticate my terminal, so Terraform needs its own credentials to make API calls on my behalf.

I generated these keys under a dedicated IAM Admin user rather than the root account, following AWS's guidance to keep root credentials out of programmatic access, and I'll deactivate and delete the key once the project is done to limit the window that a long-lived credential exists.

![Image](http://nextwork.ai/passionate_rose_serene_emblica/uploads/aws-devops-terraform1_7j8k9l0m)

---

## Lanching the S3 Bucket

I ran 'terraform apply' to execute my configuration and provision the resources I'd defined in main.tf.

Running 'terraform apply' affects my AWS account by turning my code into real infrastructure — it creates my S3 bucket and its public access block, applying the exact changes shown in the plan after I confirmed with "yes."

Unlike plan, which only previews changes, apply is the command that actually reaches into AWS and builds, modifies, or deletes resources — so it's the point where my configuration stops being a blueprint and becomes live infrastructure I'm responsible for.

The sequence of running terraform init, plan, and apply is crucial because each command sets up the conditions the next one depends on.

terraform init comes first to prepare the working directory — downloading the AWS provider and setting up the state file — so without it, plan and apply have no provider to talk to AWS and would error out. terraform plan comes next to preview exactly what will be created, updated, or destroyed, giving me a chance to review the changes before they touch anything live. terraform apply comes last to execute those changes and build the real infrastructure.

The order enforces a safe workflow: prepare, preview, then commit. apply will actually run its own plan and ask for confirmation even if I skip the explicit plan step, but running plan deliberately is best practice — it's the review gate that catches mistakes while they're still on paper rather than after they've hit my AWS environment.

![Image](http://nextwork.ai/passionate_rose_serene_emblica/uploads/aws-devops-terraform1_1q2w3e4r)

---

## Uploading an S3 Object

I created a new resource block to define an S3 object — the file I want to upload — as its own piece of managed infrastructure.

Terraform organizes each resource in its own self-contained block, so declaring an aws_s3_object block lets me manage the file separately from the bucket while still linking them: the object block references my bucket by its Terraform name, so Terraform knows exactly where to place the file.

This keeps my configuration modular — the bucket and its contents are defined independently but connected — and it means the object's upload is now handled as code, so it's version-controlled and repeatable rather than a one-off manual action in the console.

We need to run terraform apply again because I've changed my configuration since the last apply — I added a new aws_s3_object resource to main.tf.

Terraform doesn't automatically push changes; it only reconciles my infrastructure with my code when I explicitly apply. When I run apply again, Terraform compares my updated configuration against the current state, sees that the bucket already exists but the object doesn't, and creates only the new object without recreating or disturbing what's already there.

To validate that I've updated my configuration successfully, I headed to the S3 console and opened my bucket to confirm the file had actually been uploaded.

I saw the object listed inside the bucket, then selected it and used Download to pull the file back down and open it, confirming its contents matched exactly what I'd uploaded from my project folder.

This closes the loop between my Terraform code and the live result: seeing the object in the console and verifying the downloaded file proves the apply worked end-to-end, and that the same declarative approach managing my bucket now reliably manages what's stored inside it.

![Image](http://nextwork.ai/passionate_rose_serene_emblica/uploads/aws-devops-terraform1_9o0p1a2s)

---

---
