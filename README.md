# Terraform-Profider-Bigv

[BigV](http://bigv.io) is a private dedicated virtual machine platfom provided by Bytemark,

This terraform provider allows the management of machines on that platform.

See [BigV prices](http://www.bigv.io/prices), [Resource definitions](http://www.bigv.io/support/api/definitions/) and 
[API documentation](http://www.bigv.io/support/api/virtual_machines/) for configuration options to resource parameters.

## Provider parameters

* **account**

   Bigv account name.

* **user**

   Bigv Username

* **group**

   The group to use. Often this is just "default".

* **password**

   Your bigv password.
   Yubikey is not yet supported. Patches welcome.

## Resource parameters

* **name**

   The VM name

* **cores**

   How many cores to allocate. Note that cores must be allocated 1 per 4GiB of RAM.
   So 1-3GiB = 1 core, 4-7GiB = 2 core.

   We force you to state the cores instead of computing them for you to avoid suprises.

* **memory**

   How much RAM to allocate in MiB. Note that this affects cores; See above.

* **os**

   Short name of operating system to image.
   See: [Resource definitions](http://www.bigv.io/support/api/definitions/)
   Common options are: jessie, vivid and none

* **ipv4**
* **ipv6**

   IP address to allocate.
   We force ipv6 to be specified because it eases the burden on bytemark's allocation process,
   and should allow concurrent imaging without deadlocks.

* **disc_size**

   Disc size in MiB. More options in the API are not yet supported, such as storage grade.

## Computed values

* **root_password**

   The root password assigned to this vm.

## Example Usage

variables.tf:
```
variable "account" {
  default = "myaccount"
}
variable "user" {
  default = "myuser"
}
 variable "group" {
  default = "default"
}
// Probably best to supply this interactively
variable "password" { }
```

provider.tf:
```
provider "bigv" {
  account  = "${var.account}"
  user     = "${var.user}"
  password = "${var.password}"
  group    = "${var.group}"
}
```

resources.tf:
```
resource "bigv_vm" "tf01" {
  name    = "tf01"
  ipv4    = "49.48.12.201"
  ipv6    = "1996:41c8:20:5ed::3:1"
  cores   = 1
  memory  = 1024
  os      = "vivid"
}
 
resource "bigv_vm" "tf02" {
  name    = "tf02"
  ipv4    = "49.48.12.202"
  ipv6    = "1996:41c8:20:5ed::3:2"
  cores   = 2
  memory  = 4096
  os      = "trusty"
}
 
resource "bigv_vm" "tf03" {
  name    = "tf03"
  ipv4     = "49.48.12.203"
  ipv6    = "1996:41c8:20:5ed::3:3"
  cores   = 1
  memory  = 1024
  os      = "none"
} 
```

And then just:
```
terraform plan
terraform apply
terraform refresh
terraform destroy
```

## Debugging and troubleshooting

Terraform commands support TF_LOG=level environment variables:
```
TF_LOG=DEBUG terraform apply
```
This provider will be fairly verbose.
