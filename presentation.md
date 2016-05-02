title: Terraform
theme: sjaakvandenberg/cleaver-light
author:
  name: Mike Ball
  url: https://github.com/mdb
output: index.html
style: style.css

---

# [Terraform](https://terraform.io)

## [Mike Ball](https://github.com/mdb)

---

### Mike Ball

* [mikeball.info](http://mikeball.info)
* [github.com/mdb](https://github.com/mdb)
* [twitter.com/clapexcitement](https://twitter.com/clapexcitement)

---

### Follow along

Slides are here:

[mdb.github.io/terraform-presentation](https://mdb.github.io/terraform-presentation)

---

<img src="images/comcast_logo.png" />

---

<img src="images/xtv.png" />

---

# Relevancy to Senior Dev Day?

---

<img src="images/xfinity_origin.png" />

---

# Enter Terraform

---

### What is Terraform?

> Terraform provides a common configuration to launch infrastructure &mdash; from physical and virtual servers to email and DNS providers.

Says [terraform.io](https://www.terraform.io/)

(oh, and [HashiCorp](https://www.hashicorp.com/) built it)

---

### Huh?

What does this definition mean?

* build, change, version infrastructure through common config file
* codify everything from physical hardware, VMs, and containers, to email and DNS providers
* with multi-cloud/provider support!

---

### How's that work?

* `.tf` config file allows teams to describe their infrastructure
* `terraform` CLI creates, changes, and destroys resources accordingly

---

### What's that look like?

Declare `.tf` file resources via HCL (HashiCorp Configuration Lanaguage):

```
resource "digitalocean_droplet" "web" {
  name = "tf-web"
  size = "512mb"
  image = "centos-5-8-x32"
  region = "sfo1"
}

resource "dnsimple_record" "hello" {
  domain = "example.com"
  name = "test"
  value = "${digitalocean_droplet.web.ipv4_address}"
  type = "A"
}
```

---

### Basic CLI usage

* `terraform plan` to view the execution plan
* `terraform apply` to execute the plan
* `terraform destroy` to destroy infrastructure

(there are more; these are the basics)

---

### tfstate

Terraform saves record of infrastructure state in JSON.

* current state lives in `terraform.tfstate`
* backup of previous state lives in `terraform.tfstate.backup`

---

### CRUD lifecycle

`plan` + `apply` resolve what you _want_ with what _exists_, using `.tf`, provider APIs, and `tfstate`.

---

### How to orchestrate such changes across a team?

* how to coordinate?
* how to run infrastructure changes through CI?

---

### Demo time

Continuous delivery with Terraform from [TravisCI](http://travis-ci.org)

[github.com/mdb/terraform-example](htps://github.com/mdb/terraform-example)

---

### Thesis statement

terraform
+
software engineering practices (git, code review, CI)
=
😎

---

### TravisCI will...

1. use Node.js to compile source to a directory of static HTML, CSS, and image files
2. if `master`, execute `terraform apply` to configure DNS & AWS S3 & deploy the site at `mikeball.me`

---

# Let's tour the [`.travis.yml`](https://github.com/mdb/terraform-example/blob/master/.travis.yml)

---

# And let's tour [`terraform/main.tf`](https://github.com/mdb/terraform-example/blob/master/terraform/main.tf)

---

# And let's tour [`deploy.sh`](https://github.com/mdb/terraform-example/blob/master/deploy.sh)

---

The "kinda" demo...

turns `mikeball.me` from 1) nothing into 2) a functional website

---

<img src="images/demo_1.png" />

---

<a href="https://github.com/mdb/terraform-example/pull/1"><img src="images/demo_2.png" /></a>

---

<img src="images/demo_3.png" />

---

<a href="https://travis-ci.org/mdb/terraform-example/builds/127083558"><img src="images/demo_4.png" /></a>

---

<a href="https://github.com/mdb/terraform-example/pull/1"><img src="images/demo_5.png" /></a>

---

<a href="https://travis-ci.org/mdb/terraform-example/builds/127086149"><img src="images/demo_6.png" /></a>

---

[terraform-example/terraform/terraform.tfstate](https://github.com/mdb/terraform-example/blob/master/terraform/terraform.tfstate)

---

# [mikeball.me](http://mikeball.me)

---

# Why?

---

# Quality

---

codified infrastructure managed like software source code via code reviews!

* quality at scale
* change is cheap (safe + easy)
* declarative DSL is preferable to custom business logic
* state backups

---

### VS other tools

---

### gophercloud, Fog, etc.

API client libraries vs. syntax for describing cloud resources

---

```ruby
if server.nil?
  compute.servers.create(flavor_id: 1, image_id: 3, name: 'my_server')
else
  # do something else
end
```

---

```
resource "aws_instance" "my_server" {
  ami = "ami-408c7f28"
  instance_type = "t1.micro"
}
```

---

### Chef, Puppet, etc.

Individual machine management vs. higher level datacenter/service abstraction

---

### Config management

Good at...

---

<img src="images/config_management_0.png" />

---

<img src="images/config_management_1.png" />

---

# Terraform is good at...

---


---

<img src="images/config_management_2.png" />

---

### CloudFormation, Heat, etc.

Provider-specific vs. multi-provider agnostic

---

### CloudFormation & Heat

Good at...

---

<img src="images/single_cloud.png" />

---

# Terraform is good at...

---

<img src="images/multi_cloud.png" />

---

### Multi-provider support

[github.com/hashicorp/terraform/tree/master/builtin/providers](https://github.com/hashicorp/terraform/tree/master/builtin/providers)

---

# Some other points of note...

---

### Remote state

How to coordinate across teams? And avoid merge conflicts?

And keep sensitive data secret?

Enter [remote state](https://www.terraform.io/docs/state/remote/index.html)

---

### Supported backends

* artifactory
* atlas
* consul
* etcd
* http
* s3
* swift

---

# Custom providers

---

<img src="images/paul_hinze.png" />

---

> If it's an internet-accessible API that can be modeled declaratively, it can be terraform'd.

says Paul Hinze, Terraform core contributor

---

### Custom providers

> A provider in Terraform is responsible for the lifecycle of a resource: create, read, update, delete

---

### Write your own

Write your own for internal services; good intro to Golang!

* abstract away per-team business logic to a shared `terraform-provider-${foo}`
* ensure against lock-in; `tf` changes are cheap
* implement [ResourceProvider](https://github.com/hashicorp/terraform/blob/master/terraform/resource_provider.go) interface

[Read more &raquo;](https://www.terraform.io/docs/plugins/provider.html)

---

### Gotchas

* sensitive data in `tfstate`
* edits outside of remote state get wacky
* "destructive" actions can will yield downtime

---