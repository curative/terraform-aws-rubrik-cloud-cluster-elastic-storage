# Terraform Module - AWS Cloud Cluster Deployment

Terraform module which deploys a new Rubrik Cloud Cluster Elastic Storage (CCES) in AWS.

## Documentation

Here are some resources to get you started! If you find any challenges from this project are not properly documented or are unclear, please [raise an issue](../../issues/new/choose) and let us know! This is a fun, safe environment - don't worry if you're a GitHub newbie!

- [Terraform Module Registry](https://registry.terraform.io/modules/rubrikinc/rubrik-cloud-cluster-elastic-storage)
- [Quick Start Guide](docs/quick-start.md)

## Prerequisites

There are a few services you'll need in order to get this project off the ground:

- [Terraform](https://www.terraform.io/downloads.html) v1.2.2 or greater
- [Rubrik RSC Provider for Terraform](https://github.com/rubrikinc/terraform-provider-polaris) - provides Terraform functions for Rubrik
  - Only required to use the `polaris_cdm_bootstrap_cces_aws` resource.
- The Rubik Cloud Cluster product in the AWS Marketplace must be subscribed to. Otherwise, an error like this will be displayed:
  > Error: creating EC2 Instance: OptInRequired: In order to use this AWS Marketplace product you need to accept terms and subscribe. To do so please visit https://aws.amazon.com/marketplace/pp?sku=<sku_number>

    If this occurs, open the specific link from the error, while logged into the AWS account where Cloud Cluster will be deployed. Follow the instructions for subscribing to the product.

### Usage

```hcl
module "rubrik_aws_cloud_cluster" {
  source  = "rubrikinc/rubrik-cloud-cluster-elastic-storage/aws"

  aws_region                 = "us-west-1"
  aws_subnet_id              = "subnet-1234567890abcdefg"
  aws_ami_filter             = ["rubrik-mp-cc-7*"]
  cluster_name               = "rubrik-cloud-cluster"
  admin_email                = "build@rubrik.com"
  admin_password              = "RubrikGoForward"
  dns_search_domain          = ["rubrikdemo.com"]
  dns_name_servers           = ["192.168.100.5","192.168.100.6"]
  ntp_server1_name            = "8.8.8.8"
  ntp_server2_name            = "8.8.4.4"
}
```

### Inputs

The following are the variables accepted by the module.

#### Instance/Node Settings

| Name                                            | Description                                                                                                              |  Type  |          Default           | Required |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | :----: | :------------------------: | :------: |
| aws_region                                      | The region to deploy Rubrik Cloud Cluster nodes.                                                                         | string |                            |   yes    |
| aws_instance_imdsv2                             | Enable support for IMDSv2 instances. Only supported with CCES v8.1.3 or CCES v9.0 and higher.                            |  bool  |           false            |    no    |
| aws_instance_type                               | The type of instance to use as Rubrik Cloud Cluster nodes. CC-ES requires m5.4xlarge.                                    | string |         m5.4xlarge         |    no    |
| aws_disable_api_termination                     | If true, enables EC2 Instance Termination Protection on the Rubrik Cloud Cluster nodes.                                  |  bool  |            true            |    no    |
| aws_tags                                        | Tags to add to the resources that this Terraform script creates, including the Rubrik cluster nodes.                     |  map   |                            |    no    |
| number_of_nodes                                 | The total number of nodes in Rubrik Cloud Cluster.                                                                       |  int   |             3              |    no    |
| aws_ami_owners                                  | AWS marketplace account(s) that owns the Rubrik Cloud Cluster AMIs.                                                      |  list  |      ["679593333241"]      |    no    |
| aws_ami_filter                                  | Cloud Cluster AWS AMI name pattern(s) to search for. Use [\"rubrik-mp-cc-[X]*\"]. Where [X] is the major version of CDM. |  list  |                            |   yes    |
| aws_image_id                                    | AWS Image ID to deploy. Set to 'latest' or leave blank to deploy the latest version as determined by `aws_ami_filter`.   | string |           latest           |    no    |
| aws_key_pair_name                               | Name for the AWS SSH Key-Pair being created or the existing AWS SSH Key-Pair being used.                                 | string |                            |    no    |
| private_key_recovery_window_in_days             | Recovery window in days to recover script generated ssh private key.                                                     | string |             30             |    no    |

*Note: When using the `aws_tags` variable, the "Name" tag is automatically used by this TF for those resources that support it.*

*Note: The `aws_ami_filter` and `aws_ami_owners` variables are only used when the `aws_image_id` variable is blank or set to `latest`*

*Note: When using the `aws_image_id` variable, see [Selecting a specific image](#selecting-a-specific-image) for details on finding images.*

*Note: When using the `aws_key_pair_name` variable, if a new AWS SSH Key-Pair is being created and no name is specified, a name will be automatically generated.*

<br>

#### Network Settings

| Name                                            | Description                                                                                                              |  Type  |          Default           | Required |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | :----: | :------------------------: | :------: |
| create_cloud_cluster_nodes_sg                   | If true, creates a new Security Group for node to node traffic within the Rubrik cluster.                                |  bool  |            true            |    no    |
| aws_vpc_cloud_cluster_nodes_sg_name             | The name of the security group to create for Rubrik Cloud Cluster to use.                                                | string |    Rubrik Cloud Cluster    |    no    |
| cloud_cluster_nodes_admin_cidr                  | The CIDR range for the systems used to administer the Cloud Cluster via SSH and HTTPS.                                   | string |         0.0.0.0/0          |    no    |
| create_cloud_cluster_hosts_sg                   | If true, creates a new Security Group for node to host traffic from the Rubrik cluster.                                  | string |            true            |    no    |
| aws_vpc_cloud_cluster_hosts_sg_name             | The name of the security group to create for Rubrik Cloud Cluster to communicate with EC2 instances.                     | string | Rubrik Cloud Cluster Hosts |    no    |
| aws_cloud_cluster_nodes_sg_ids                  | Additional security groups to add to Rubrik cluster nodes.                                                               | string |                            |    no    |
| aws_subnet_id                                   | The VPC Subnet ID to launch Rubrik Cloud Cluster in.                                                                     | string |                            |   yes    |

#### Storage Settings

| Name                                            | Description                                                                                             |  Type  | Default | Required |
| ----------------------------------------------- |---------------------------------------------------------------------------------------------------------| :----: |:-------:| :------: |
| cluster_disk_type                               | Disk type for the data disks (gp2 or gp3).                                                              | string |   gp3   |    no    |
| cluster_disk_size                               | The size (in GB) of each data disk on each node. Cloud Cluster ES only requires 1 512 GB disk per node. | string |   512   |    no    |
| cluster_disk_count                              | The number of disks for each node in the cluster. Set to 1 to use with S3 storage for Cloud Cluster ES. |  int   |    1    |    no    |

#### Cloud Cluster ES Settings

| Name                                            | Description                                                                                                           |  Type  | Default | Required |
|-------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------| :----: |:-------:| :------: |
| create_iam_role                                 | If true, create required IAM role, role policy, and instance profile needed for Cloud Cluster ES.                     |  bool  |  true   |    no    |
| aws_cloud_cluster_iam_role_name                 | AWS IAM Role name for Cloud Cluster ES. If blank a name will be auto generated. Required if create_iam_role is false. | string |         |    no    |
| aws_cloud_cluster_iam_role_policy_name          | AWS IAM Role policy name for Cloud Cluster ES if create_iam_role is true. If blank a name will be auto generated.     | string |         |    no    |
| aws_cloud_cluster_ec2_instance_profile_name     | AWS EC2 Instance Profile name that links the IAM Role to Cloud Cluster ES. If blank a name will be auto generated.    | string |         |    no    |
| create_s3_bucket                                | If true, create am S3 bucket for Cloud Cluster ES data storage.                                                       |  bool  |  true   |    no    |
| s3_bucket_name                                  | Name of the S3 bucket to use with Cloud Cluster ES data storage. If blank a name will be auto generated.              | string |         |    no    |
| s3_bucket_force_destroy                         | Indicates all objects should be deleted from the bucket so that the bucket can be destroyed without error.            |  bool  |  false  |    no    |
| enableImmutability                              | Enable immutability on the S3 objects that CCES uses. Default value is true.                                          |  bool  |  true   |    no    |
| create_s3_vpc_endpoint                          | If true, create a VPC Endpoint and S3 Endpoint Service for Cloud Cluster ES.                                          |  bool  |  true   |    no    |
| s3_vpc_endpoint_route_table_ids                 | Route table IDs for VPC Endpoint and S3 Endpoint Service.                                                             |  list  |         |    no    |

#### Bootstrap Settings

| Name                                            | Description                                                                                                              |  Type  |       Default       | Required |
| ----------------------------------------------- |--------------------------------------------------------------------------------------------------------------------------|:------:|:-------------------:|:--------:|
| cluster_name                                    | Unique name to assign to Rubrik Cloud Cluster. Also used for EC2 instance name tag. For example, rubrik-1, rubrik-2 etc. | string |                     |   yes    |
| admin_email                                     | The Rubrik Cloud Cluster sends messages for the admin account to this email address.                                     | string |                     |   yes    |
| admin_password                                  | Password for the Rubrik Cloud Cluster admin account.                                                                     | string |   RubrikGoForward   |    no    |
| dns_search_domain                               | List of search domains that the DNS Service will use to resolve hostnames that are not fully qualified.                  |  list  |                     |   yes    |
| dns_name_servers                                | List of the IPv4 addresses of the DNS servers.                                                                           |  list  | ["169.254.169.253"] |    no    |
| ntp_server1_name                                | The FQDN or IPv4 addresses of network time protocol (NTP) server #1.                                                     | string |       8.8.8.8       |   yes    |
| ntp_server1_key_id                              | The ID # of the key for NTP server #1. Typically is set to 0. (Required with `ntp_server1_key` & `ntp_server1_key_type`) |  int   |          0          |    no    |
| ntp_server1_key                                 | Symmetric key material for NTP server #1. (Required with `ntp_server1_key_id` and `ntp_server1_key_type`)                | string |                     |    no    |
| ntp_server1_key_type                            | Symmetric key type for NTP server #1. (Required with `ntp_server1_key` and `ntp_server1_key_id`)                         | string |                     |    no    |
| ntp_server2_name                                | The FQDN or IPv4 addresses of network time protocol (NTP) server #2.                                                     | string |       8.8.4.4       |   yes    |
| ntp_server2_key_id                              | The ID # of the key for NTP server #2. Typically is set to 1. (Required with `ntp_server1_key` & `ntp_server1_key_type`) |  int   |          1          |    no    |
| ntp_server2_key                                 | Symmetric key material for NTP server #2. (Required with `ntp_server1_key_id` and `ntp_server1_key_type`)                | string |                     |    no    |
| ntp_server2_key_type                            | Symmetric key type for NTP server #2. (Required with `ntp_server1_key` and `ntp_server1_key_id`)                         | string |                     |    no    |
| timeout                                         | The number of seconds to wait to establish a connection the Rubrik cluster before returning a timeout error.             |  int   |         60          |    no    |
| node_boot_wait                                  | Number of seconds to wait for CCES nodes to boot before attempting to bootstrap them.                                    |  int   |         300         |    no    |
| register_cluster_with_rsc                       | Register the Rubrik Cloud Cluster with Rubrik Security Cloud. Default value is false.                                    |  bool  |        false        |    no    |

## Changes

Several variables have changed with this iteration of the script. Upgrades to existing deployments may cause unwanted changes.  Be sure to check the changes of `terraform plan` before `terraform apply` to avoid disruptive behavior.

## How You Can Help

We glady welcome contributions from the community. From updating the documentation to adding more functionality, all ideas are welcome. Thank you in advance for all of your issues, pull requests, and comments!

- [Contributing Guide](CONTRIBUTING.md)
- [Code of Conduct](CODE_OF_CONDUCT.md)

## License

- [MIT License](LICENSE)

## About Rubrik Build

We encourage all contributors to become members. We aim to grow an active, healthy community of contributors, reviewers, and code owners. Learn more in our [Welcome to the Rubrik Build Community](https://github.com/rubrikinc/welcome-to-rubrik-build) page.

We'd love to hear from you! Email us: build@rubrik.com
