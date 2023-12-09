---
author: michal
title: Structuring Terraform for world domination
date: 2019-06-25T07:00:19+01:00
categories:
  - Technology
tags:
  - Terraform
  - AWS
  - infrastructure
  - DevOps
warning: "Newer Terraform versions have changed some of the syntax used in the examples here."
---

How many programmers does it take to launch an AWS EC2 instance? How about launching a hundred? With an orderly set of [Terraform](https://www.terraform.io/) configurations, our four-person team looks after a worldwide set of cloud resources, with *no* sysadmins.

<!--more-->

I'll share the current setup we have working. Things change, our infrastructure morphs, [Terraform 0.12 will be out soon](https://www.hashicorp.com/resources/introducing-terraform-0-12), shaking things up. But for the moment, this is what works for us and keeps our customers' applications running 24/7.

**A word of caution** before you proceed: the text presumes some knowledge of Terraform and AWS, in particular the syntax and common resource types. Most of the code examples are incomplete, shortened for clarity. The final section contains a link to the full Git repository, with all examples in one place. Might be easier to follow than the fragmented code snippets posted here.

## Environment

Let's start with the requirements we have to work to:

* **Use cloud- and managed services.** We're only four programmers working on this project, and so want to spend as little time on infrastructure as possible.
* **Use a variety of services.** The right tool for the right job---AWS S3, Elastic Beanstalk, CloudFront, Lambda and others, plus non-AWS ones, like [MongoDB Atlas](https://www.mongodb.com/cloud/atlas). (That's why a single-vendor automation solution, like [AWS's CloudFormation](https://aws.amazon.com/cloudformation/), won't work.)
* **Deploy to multiple regions.** Our customer operates in dozens of countries, and we want infrastructure to stay close to the users---for performance and legal reasons.
* **Support multiple brands in each country.** Some of their infrastructure is shared, and some is distinct.

Multiply the countries, times brands, times regions, and we're looking at tens, hundreds, eventually thousands of resources. Daunting? Perhaps. But here's where core programming principles come in handy.

## Structure

The three principles we follow, in no particular order, are these:

* **Don't Repeat Yourself**---use Terraform variables and modules to reuse configurations for similar resources.
* **Encapsulation**---package resources comprising a single service into their own modules.
* **Domain-Driven Design**---name the modules, as much as possible, in business terms.

We started with a root structure of:

```
.
├── countries
├── infrastructure
├── main.tf
├── main.tfvars
└── variables.tf
```

`main.tf` is the entry point. `variables.tf` contains definitions for parameters we either want to keep out of the source code repository (like AWS keys and passwords) or those we may want to override via the command line. `main.tfvars` contains sensitive variables and it's *never* checked into the repository (guarded by `.gitignore`). Then there are the directories: infrastructure contains reusable modules used inside `countries` to build up their specific, *uhm*, infrastructure.

### Global Resources

`/main.tf` begins with definitions of global resources. Some AWS resources are one-per-account---things like [roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html), [users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html). These need to be set up once. We keep them in their own modules referenced from `/main.tf`:

``` Terraform
module "roles" {
  source = "./infrastructure/roles"
}

module "users" {
  source = "./infrastructure/users"
}
```

Both the `roles` and `users` modules have their own `outputs.tf` with ARNs, usernames and other identifiers that are then fed into the country modules.

On this level, we also have a definition for the `us-east-1` (Virginia) AWS region. (Other regions for the AWS provider are defined inside country modules---remember, we're multi-region.) It's used to configure resources which AWS defines as “Global”, while in reality they're based in Virginia:

``` Terraform
provider "aws" {
  alias   = "us_east_1"
  region  = "us-east-1"
}
```

Notice the `alias` field, which [lets us use this provider only when we need it](https://developer.hashicorp.com/terraform/language/meta-arguments/resource-provider), ie. with [SSL certificates for CloudFront](https://aws.amazon.com/premiumsupport/knowledge-center/install-ssl-cloudfront/):

``` Terraform
resource "aws_acm_certificate" "cdn" {
  provider = "aws.us_east_1"
}
```

### Country Configurations

`/main.tf` ends with the countries. Each one has its own module inside the `/countries` directory:

```
countries/
├── france
│ ├── main.tf
│ └── variables.tf
├── poland
│ ├── main.tf
│ └── variables.tf
└── spain
  ├── main.tf
  └── variables.tf
```

Each country is configured with the variables we need for it:

``` Terraform
module "poland" {
  source = "./countries/poland"
  
  aws_region                    = "eu-central-1"
  cidr_pragmatists_office       = "${var.cidr_pragmatists_office}"
  elasticbeanstalk_ec2_role     = "${module.roles.elasticbeanstalk_ec2_role_name}"
  elasticbeanstalk_service_role = "${module.roles.elasticbeanstalk_service_role_name}"
  organization                  = "poland"
}
```

We're passing the `region` to set up the `provider` for that country, then our office's CIDR for VPC Security Group setup, then [roles needed for Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts-roles.html) (notice their `module` prefix---these are the global resources), and finally there's `organization` which we're using for billing.

Corporate accounting *always* manages to somehow creep into code, and infrastructure is no different. Our customer wants each of their country organizations to be billed separately, so we're adding an `organization` tag to every AWS resource, [which we then use to calculate costs](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html).

Every country's `main.tf` starts with the AWS `provider` configuration, binding it to the specified region:

``` Terraform
provider "aws" {
  region = "${var.aws_region}"
}
```

Unlike the `us-east-1` region provider we configured in `/main.tf`, this one has no `alias` param, so it'll implicitly be used for that country's resources.

The second item for each country is its **VPC setup**. Because AWS doesn't have a region in each country, we're usually hosting multiple customer countries in one region---each one with its own VPC. We decided to use a `10.*.0.0/16` subnet for each country, with the second octet set to the [international calling code](https://countrycode.org/) for the country---`48` for Poland, `33` for France etc.

``` Terraform
locals {
  vpc_cidr_block = "10.48.0.0/16"
}

module "networking" {
  source         = "../../infrastructure/networking"

  organization   = "${var.organization}"
  vpc_cidr_block = "${local.vpc_cidr_block}"
}
```

You'll notice that the above starts to get a bit ugly. We have an `/infrastructure/networking` module, with the [VPC, subnets, routing, gateways and everything related](https://docs.aws.amazon.com/vpc/latest/userguide/getting-started-ipv4.html), and we have to reference it with its relative path, hence `../../`. Then there's the `organization` parameter again. Terraform doesn't support global variables, so we literally have to pass it around *everywhere*.

With networking in place, each country has **nested modules for their global infrastructure and for each brand**:

``` Terraform
module "global" {
  source     = "./global"

  aws_region = "${var.aws_region}"
  vpc_id     = "${module.networking.vpc_id}"
}

module "brand_one" {
  source     = "./brand-one"

  aws_region = "${var.aws_region}"
  vpc_id     = "${module.networking.vpc_id}"
}

module "brand_two" {
  source     = "./brand-two"

  aws_region = "${var.aws_region}"
  vpc_id     = "${module.networking.vpc_id}"
}
```

We're passing in the `aws_region` here, because some configurations are different depending on the region, i.e. when new EC2 machine types are introduced, they're usually getting rolled out gradually across the world. This way, we can take advantage of them early, where available.

There's *more*. We have **test environments for each brand and country**, so these also need to be accounted for. They form the next level of nesting in country modules:

```
countries/poland/
├── global
│ ├── main.tf
│ ├── dev
│ │ └── main.tf
│ ├── prod
│ │ └── main.tf
│ └── uat
│   └── main.tf
└── brand-one
  ├── main.tf
  ├── dev
  │ └── main.tf
  ├── prod
  │ └── main.tf
  └── uat
    └── main.tf
```

A typical `main.tf` inside brand-one will look like:

``` Terraform
# Application

module "client_panel_application" {
  source = "../../../infrastructure/api/application"
  name   = "Application Name"
}

# Environments

module "dev" {
  source           = "./dev"

  organization     = "${var.organization}"
  application_name = "${module.client_panel_application.name}"
  vpc_id           = "${var.vpc_id}"
  region           = "${var.aws_region}"
}

module "uat" {
  source           = "./uat"

  organization     = "${var.organization}"
  application_name = "${module.client_panel_application.name}"
  vpc_id           = "${var.vpc_id}"
  region           = "${var.aws_region}"
}

module "prod" {
  source           = "./prod"

  organization     = "${var.organization}"
  vpc_id           = "${var.vpc_id}"
  application_name = "${module.client_panel_application.name}"
  region           = "${var.aws_region}"
}
```

The `client_panel_application` module [creates an Elastic Beanstalk Application](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/applications.html), which is then shared by the environments.

Is that it? Not necessarily. Sometimes we have more submodules inside `global` for each brand, in order to group things that go together, i.e. all DNS configurations for Route 53, which only applies to the `prod` environment:

```
countries/poland/brand-one/
└── prod
  ├── dns
  │ └── main.tf
  └── main.tf
```

### What's in Infrastructure?

While it looks as though we have a *lot* of code inside the `countries` directory (and technically that's true), most of it is referencing modules inside `infrastructure` because all brands and countries rely on the same application code and share mostly the same setup.

We're trying to name infrastructure modules following DDD-rules, using names related to the business domains we're building for. It doesn't always apply, i.e. with `roles` or `networking` but in any case, we avoid using technology names, like S3 or Lambda or anything else vendor-specific.

Here are some examples:

```
infrastructure/
├── api
├── asset-store
├── backoffice
├── client-panel
└── networking
```

Some of these directories contain multiple modules, i.e. for the `api` which is running on Elastic Beanstalk, where we need a separate module for creating the application, and there are differences between production and non-production environments (mostly in load-balancing capacity, if you're wondering):

```
infrastructure/api/
├── application
├── dev
├── prod
└── uat
```

Infrastructure modules group everything needed to set up a given group of resources, including external templates, policies and the like. For instance, there's an `asset-store` which is an S3 bucket with a public read policy and specific user access policies for uploading:

```
infrastructure/asset-store/
├── main.tf
├── policy-iam-s3-user-access.json.tpl
└── policy-s3-asset-store.json.tpl
```

We're then rendering these policies inside the module's `main.tf`:

``` Terraform
data "template_file" "s3_bucket_policy" {
  template = "${file("${path.module}/policy-s3-asset-store.json.tpl")}"
  vars {
    s3_bucket = "${local.s3_bucket}"
  }
}

resource "aws_s3_bucket" "default" {
  bucket = "${local.s3_bucket}"
  policy = "${data.template_file.s3_bucket_policy.rendered}"
}
```

## Complete Project

Not *really* "complete", because we cannot, obviously, publish actual customer code. Plus, the whole setup now includes over 100 files and counting. But here's a repo with all the examples described above, and all the cross-references between them. You can use it for inspiration or as a skeleton for any similar project you might have.

https://github.com/Pragmatists/terraform-structure-sample

Once Terraform 0.12 is out, we're sure to upgrade our code and very likely to post a follow-up, detailing the benefits (or drawbacks) we encountered.

&nbsp;

Thanks to Łukasz Kyć and Zbigniew Artemiuk for their reviews and feedback. Reposted with permission from [Pragmatists](https://pragmatists.com/).
