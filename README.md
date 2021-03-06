# [Google Address Terraform Module](https://registry.terraform.io/modules/terraform-google-modules/address/google/)

This terraform module provides the means to permanently reserve an [IP address](https://cloud.google.com/compute/docs/ip-addresses/)
available to Google Cloud Platform (GCP) resources, and optionally create
forward and reverse entries within Google Cloud DNS. The intent is to provide an
address resource which exists independent of the lifecycle of the resources
that require the address.

## Compatibility

 This module is meant for use with Terraform 0.12. If you haven't [upgraded](https://www.terraform.io/upgrade-guides/0-12.html)
  and need a Terraform 0.11.x-compatible version of this module, the last released version intended for
  Terraform 0.11.x is [1.0.0](https://registry.terraform.io/modules/terraform-google-modules/address/google/1.0.0).


# Usage

## Examples without DNS

Examples are provided in the `examples` folder, but to simply reserve IP
addresses on a subnetwork without registering them in DNS refer to the
following example:

```hcl
module "address-fe" {
  source  = "terraform-google-modules/address/google"
  version = "0.1.0"

  subnetwork = "projects/gcp-network/regions/us-west1/subnetworks/dev-us-west1-dynamic"

  names = [
    "gusw1-dev-fooapp-fe-0001-a-001-ip",
    "gusw1-dev-fooapp-fe-0001-a-002-ip",
    "gusw1-dev-fooapp-fe-0001-a-003-ip"
  ]
}
```

Then perform the following commands on the root folder:

- `terraform init` to get the plugins
- `terraform plan` to see the infrastructure plan
- `terraform apply` to apply the infrastructure build
- `terraform destroy` to destroy the built infrastructure

An `addresses` output has been provided as the list of IP addresses that were
reserved by GCP. Because the `addresses` input variable was not specified, GCP has
reserved the next available IP addresses from the subnetwork provided. The number
of IP addresses reserved is equal to the length of the `names` input
variable, so size that list accordingly.

If you would prefer to provide the specific IP addresses to be reserved, that can be accomplished with the `addresses` input variable:

```hcl
module "address-fe" {
  source  = "terraform-google-modules/address/google"
  version = "0.1.0"

  subnetwork = "projects/gcp-network/regions/us-west1/subnetworks/dev-us-west1-dynamic"

  names = [
    "gusw1-dev-fooapp-fe-0001-a-001-ip",
    "gusw1-dev-fooapp-fe-0001-a-002-ip",
    "gusw1-dev-fooapp-fe-0001-a-003-ip"
  ]

  addresses = [
    "10.11.0.10",
    "10.11.0.11",
    "10.11.0.12"
  ]
}
```

Note that the IP addresses must not be reserved and must fall within the range of the provided subnetwork.

### External IP address

External IP addresses can be reserved by setting the `global` input var to `true` and omitting the subnetwork:

```hcl
module "address-fe" {
  source  = "terraform-google-modules/address/google"
  version = "0.1.0"

  names  = [ "external-facing-ip"]
  global = true
}
```

## DNS Examples

Optionally, the IP addresses you reserve can be registered in Google Cloud
DNS by providing information on the project hosting the Cloud DNS zone, the
managed zone name, the domain registered with Cloud DNS, and setting the
`enable_gcp_dns` feature flag to `true`:

```hcl
module "address-fe" {
  source  = "terraform-google-modules/address/google"
  version = "0.1.0"

  subnetwork           = "projects/gcp-network/regions/us-west1/subnetworks/dev-us-west1-dynamic"
  enable_gcp_dns       = true
  gcp_dns_project      = "gcp-dns"
  gcp_dns_domain       = "example.com"
  gcp_dns_managed_zone = "nonprod-dns-zone"

  names = [
    "gusw1-dev-fooapp-fe-0001-a-001-ip",
    "gusw1-dev-fooapp-fe-0001-a-002-ip",
    "gusw1-dev-fooapp-fe-0001-a-003-ip"
  ]

  dns_short_names = [
    "gusw1-dev-fooapp-fe-0001-a-001",
    "gusw1-dev-fooapp-fe-0001-a-002",
    "gusw1-dev-fooapp-fe-0001-a-003"
  ]
}
```

### Reverse DNS

The module also supports the ability to register reverse DNS entries within
their own zone by setting the `enable_gcp_ptr` feature flag to `true` and
specifying the zone with the `gcp_dns_reverse_zone` input variable:

```hcl
module "address-fe" {
  source  = "terraform-google-modules/address/google"
  version = "0.1.0"

  subnetwork           = "projects/gcp-network/regions/us-west1/subnetworks/dev-us-west1-dynamic"
  enable_gcp_dns       = true
  enable_gcp_ptr       = true
  gcp_dns_project      = "gcp-dns"
  gcp_dns_domain       = "example.com"
  gcp_dns_managed_zone = "nonprod-dns-zone"
  gcp_dns_reverse_zone = "nonprod-dns-reverse-zone"

  names = [
    "gusw1-dev-fooapp-fe-0001-a-001-ip",
    "gusw1-dev-fooapp-fe-0001-a-002-ip",
    "gusw1-dev-fooapp-fe-0001-a-003-ip"
  ]

  dns_short_names = [
    "gusw1-dev-fooapp-fe-0001-a-001",
    "gusw1-dev-fooapp-fe-0001-a-002",
    "gusw1-dev-fooapp-fe-0001-a-003"
  ]
}
```

As with the non-DNS examples above, the `addresses` input variable can be
provided with a list of specific IP addresses to be reserved if desired.

## Input variables that cannot contain computed values

Because of the way the module is structured, and due to the fact that
Terraform doesn't yet support computed count values, there are certain input
variables whose values cannot be computed values. The list of those input
variables is as follows:

```
var.dns_domain
var.dns_short_names
var.enable_cloud_dns
var.enable_reverse_dns
var.global
var.names
```

You must currently use literal values for these input variables. If you
don't you run the risk of failing validation (at the least) or surfacing the
dreaded `value of 'count' cannot be computed` error. Future versions of
Terraform may change this fact, but this is the current limitation.

[^]: (autogen_docs_start)

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| address\_type | The type of address to reserve, either "INTERNAL" or "EXTERNAL". If unspecified, defaults to "INTERNAL". | string | `"INTERNAL"` | no |
| addresses | A list of IP addresses to create.  GCP will reserve unreserved addresses if given the value "".  If multiple names are given the default value is sufficient to have multiple addresses automatically picked for each name. | list | `<list>` | no |
| dns\_domain | The domain to append to DNS short names when registering in Cloud DNS. | string | `""` | no |
| dns\_managed\_zone | The name of the managed zone to create records within.  This managed zone must exist in the host project. | string | `""` | no |
| dns\_project | The project where DNS A records will be configured. | string | `""` | no |
| dns\_record\_type | The type of records to create in the managed zone.  (e.g. "A") | string | `"A"` | no |
| dns\_reverse\_zone | The name of the managed zone to create PTR records within.  This managed zone must exist in the host project. | string | `""` | no |
| dns\_short\_names | A list of DNS short names to register within Cloud DNS.  Names corresponding to addresses must align by their list index position in the two input variables, `names` and `dns_short_names`.  If an empty list, no domain names are registered.  Multiple names may be registered to the same address by passing a single element list to names and multiple elements to dns_short_names.  (e.g. ["gusw1-dev-fooapp-fe-0001-a-001"]) | list | `<list>` | no |
| dns\_ttl | The DNS TTL in seconds for records created in Cloud DNS.  The default value should be used unless the application demands special handling. | string | `"300"` | no |
| enable\_cloud\_dns | If a value is set, register records in Cloud DNS. | string | `""` | no |
| enable\_reverse\_dns | If a value is set, register reverse DNS PTR records in Cloud DNS in the managed zone specified by dns_reverse_zone | string | `""` | no |
| global | The scope in which the address should live. If set to true, the IP address will be globally scoped. Defaults to false, i.e. regionally scoped. When set to true, do not provide a subnetwork. | string | `"false"` | no |
| names | A list of IP address resource names to create.  This is the GCP resource name and not the associated hostname of the IP address.  Existing resource names may be found with `gcloud compute addresses list` (e.g. ["gusw1-dev-fooapp-fe-0001-a-001-ip"]) | list | `<list>` | no |
| project\_id | The project ID to create the address in | string | n/a | yes |
| region | The region to create the address in | string | n/a | yes |
| subnetwork | The subnet containing the address.  For EXTERNAL addresses use the empty string, "".  (e.g. "projects/<project-name>/regions/<region-name>/subnetworks/<subnetwork-name>") | string | `""` | no |

## Outputs

| Name | Description |
|------|-------------|
| addresses | List of address values managed by this module (e.g. ["1.2.3.4"]) |
| dns\_fqdns | List of DNS fully qualified domain names registered in Cloud DNS.  (e.g. ["gusw1-dev-fooapp-fe-0001-a-001.example.com", "gusw1-dev-fooapp-fe-0001-a-0002.example.com"]) |
| names | List of address resource names managed by this module (e.g. ["gusw1-dev-fooapp-fe-0001-a-0001-ip"]) |
| reverse\_dns\_fqdns | List of reverse DNS PTR records registered in Cloud DNS.  (e.g. ["1.2.11.10.in-addr.arpa", "2.2.11.10.in-addr.arpa"]) |
| self\_links | List of URIs of the created address resources (e.g. ["https://www.googleapis.com/compute/v1/projects/project-abcde/regions/europe-west1/addresses/gusw1-dev-fooapp-fe-0001-a-0001-ip"] |

[^]: (autogen_docs_end)


## Requirements
### Terraform plugins
- [Terraform](https://www.terraform.io/downloads.html) 0.11.x
- [terraform-provider-google](https://github.com/terraform-providers/terraform-provider-google) plugin v1.12.0

### Configure a Service Account
In order to execute this module you must have a Service Account with the following roles:

- roles/dns.admin on the project (for DNS registration)
- roles/compute.networkAdmin on the organization (or the host project that defines the network)

#### Script Helper
A [helper script](./helpers/setup-sa.sh) is included to automatically grant all the
required roles at the project level. The `roles/compute.networkAdmin` can
either be assigned at the project level on the project hosting the network
and subnetworks where IP addresses will be reserved, or at the organization
level. The `setup-sa.sh` script will assign it at the project level on the
host project that is passed in. If this is not what you need then you will
need to adjust permissions accordingly.

Run the script as follows:

```
./helpers/setup-sa.sh <HOST_PROJECT_NAME> <SERVICE_ACCOUNT_NAME>
```

The `SERVICE_ACCOUNT_NAME` can be whatever you want the service account to be
named. Successful completion of the `setup-sa.sh` script will result in a
credentials file called `credentials.json` that can be used with
`gcloud` or referenced by the module tests.

### Enable API's
In order to operate with the Service Account you must activate the following API on the project where the Service Account was created:

- Compute Engine API - compute.googleapis.com
- Google Cloud DNS API - dns.googleapis.com

NOTE: These APIs are enabled by default on the host project passed in to the `./helpers/setup-sa.sh` helper script.

## Install

### Terraform
Be sure you have the correct Terraform version (0.12.x), you can choose the binary here:
- https://releases.hashicorp.com/terraform/

## File structure
The project has the following folders and files:

- /: root folder
- /examples: examples for using this module
- /test: Folders with files for testing the module (see Testing section on this file)
- /main.tf: main file for this module, contains all the resources to create
- /variables.tf: all the variables for the module
- /output.tf: the outputs of the module
- /README.md: this file

## Testing and documentation generation

### Requirements
- [terraform-docs](https://github.com/segmentio/terraform-docs/releases) 0.3.0

### Autogeneration of documentation from .tf files
Run
```
make generate_docs
```
### Integration test

Integration tests are run though [test-kitchen](https://github.com/test-kitchen/test-kitchen), [kitchen-terraform](https://github.com/newcontext-oss/kitchen-terraform), and [InSpec](https://github.com/inspec/inspec).

`test-kitchen` instances are defined in [`.kitchen.yml`](./.kitchen.yml). The test-kitchen instances in `test/fixtures/` wrap identically-named examples in the `examples/` directory.

#### Setup

1. Configure the [test fixtures](#test-configuration)
2. Download a Service Account key with the necessary permissions and put it in the module's root directory with the name `credentials.json`.
3. Build the Docker container for testing:

  ```
  make docker_build_kitchen_terraform
  ```
4. Run the testing container in interactive mode:

  ```
  make docker_run
  ```

  The module root directory will be loaded into the Docker container at `/cft/workdir/`.
5. Run kitchen-terraform to test the infrastructure:

  1. `kitchen create` creates Terraform state and downloads modules, if applicable.
  2. `kitchen converge` creates the underlying resources. Run `kitchen converge <INSTANCE_NAME>` to create resources for a specific test case.
  3. `kitchen verify` tests the created infrastructure. Run `kitchen verify <INSTANCE_NAME>` to run a specific test case.
  4. `kitchen destroy` tears down the underlying resources created by `kitchen converge`. Run `kitchen destroy <INSTANCE_NAME>` to tear down resources for a specific test case.

Alternatively, you can simply run `make test_integration_docker` to run all the test steps non-interactively.

#### Test configuration

Each test-kitchen instance is configured with a `terraform.tfvars` file in the test fixture directory. For convenience, since all of the variables are project-specific, these files have been symlinked to `test/fixtures/shared/terraform.tfvars`.
Similarly, each test fixture has a `variables.tf` to define these variables, and an `outputs.tf` to facilitate providing necessary information for `inspec` to locate and query against created resources.

Each test-kitchen instance creates necessary fixtures to house resources.

### Autogeneration of documentation from .tf files
Run
```
make generate_docs
```

### Linting
The makefile in this project will lint or sometimes just format any shell,
Python, golang, Terraform, or Dockerfiles. The linters will only be run if
the makefile finds files with the appropriate file extension.

All of the linter checks are in the default make target, so you just have to
run

```
make -s
```

The -s is for 'silent'. Successful output looks like this

```
Running shellcheck
Running flake8
Running gofmt
Running terraform validate
Test passed - Verified all file Apache 2 headers
```

The linters
are as follows:
* Shell - shellcheck. Can be found in homebrew
* Python - flake8. Can be installed with 'pip install flake8'
* Golang - gofmt. gofmt comes with the standard golang installation. golang
is a compiled language so there is no standard linter.
* Terraform - terraform has a built-in linter in the 'terraform validate'
command.
