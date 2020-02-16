# Terraform Gandi Provider

This provider allows managing DNS records on the Gandi LiveDNS service.

https://doc.livedns.gandi.net/

This provider doesn't provide access to the other Gandi API (https://doc.rpc.gandi.net/); if you want to contribute, don't hesitate!

## Compiling

These commands will generate a compiled file in `${HOME}/go/bin` called `terraform-provider-gandi`.

```
make
```

or 

```
go build -o terraform-provider-gandi
```

## Installation

The file previously generated in Compiling section (`${HOME}/go/bin/terraform-provider-gandi`) must be moved/copied in your current terraform directory. Here the steps:

```sh
# create your directory where all your tf files will be stored
mkdir terraform_gandi
cd terraform_gandi

# create terraform plugins directory
mkdir -p ./terraform.d/plugins/${os}_${arch}/

# copy gandi provider
cp ${HOME}/go/terraform-provider-gandi ./terraform.d/plugins/${os}_${arch}/

# set your provider
touch gandi.tf

# init terraform
terraform init .
```

where `${os}` is your operating system (e.g. linux, freebsd openbsd) and `${arch}` is your architecture (e.g. x86, amd64). If your plugin was correctly installed, the last command should output something like:

```txt
Initializing the backend...
                                                        
Initializing provider plugins...  
                                                        
Terraform has been successfully initialized!                                                                    
                                                        
You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.                                                                                                

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

## Example

This example partly mimics the steps of [the official LiveDNS documentation example](http://doc.livedns.gandi.net/#quick-example), using the parts that have been implemented as Terraform resources.
Note: sharing_id is optional. It is used e.g. when the API key is registered to a user, where the domain you want to manage is not registered with that user (but the user does have rights on that zone/organization). 

```
provider "gandi" {
  key = "<the API key>"
  sharing_id = "<the sharing_id>"
}

resource "gandi_zone" "example_com" {
  name = "example.com Zone"
}

resource "gandi_zonerecord" "www" {
  zone = "${gandi_zone.example_com.id}"
  name = "www"
  type = "A"
  ttl = 3600
  values = [
    "192.168.0.1"
  ]
}

resource "gandi_domainattachment" "example_com" {
    domain = "example.com"
    zone = "${gandi_zone.example_com.id}"
}
```

This example sums up the available resources.

### Zone data source

If your zone already exists (which is very likely), you may use it as a data source:

```
provider "gandi" {
  key = "<the API key>"
  sharing_id = "<the sharing_id>"
}

data "gandi_zone" "example_com" {
  name = "example.com"
}

resource "gandi_zonerecord" "www" {
  zone = "${data.gandi_zone.example_com.id}"
  name = "www"
  type = "A"
  ttl = 3600
  values = [
    "192.168.0.1"
  ]
}

resource "gandi_domainattachment" "example_com" {
  domain = "example.com"
  zone   = "${data.gandi_zone.example_com.id}"
}
```

## Licensing and stuff

This provider is distributed under the terms of the Mozilla Public License version 2.0. See the `LICENSE` file.

Its main author is not affiliated in any way with Gandi - apart from being a happy customer of their services.
