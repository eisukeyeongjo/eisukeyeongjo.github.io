# 9. terraformの基本操作をおさらいしつつAWS VPCを構築する

今後仕事でAWSを触る機会が増えそうなので、もっと理解を深めたい！
ということで、たくさん検証環境を作って壊したいのでterraformを始めます。
先ずは[公式ドキュメント](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)に沿ってVPCとサブネットの構築をしたいと思います。
環境はUbuntu 22.04、terraformのバージョンは1.3.9です。
インストールは以下のドキュメントを参考に行いました。

参考：[Install Terraform  Terraform  HashiCorp Developer](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)

参考：[Official Packaging Guide](https://www.hashicorp.com/official-packaging-guide)

先ずは作業ディレクトリを作成し main.tf ファイルを作成します。

```
$ mkdir terraform-aws
$ cd terraform-aws/
$ touch main.tf
```

AWSプロバイダーを使用することを宣言

```
$ vi main.tf 
$ cat main.tf 
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}
```

リージョンの指定と認証情報の設定を行う 認証情報はマスキングしてます。
後ほど変数化して別ファイルで管理するか別の認証方式を採用してセキュリティを高めたいです。

```
# Configure the AWS Provider
provider "aws" {
  # Use Tokyo region
  region = "ap-northeast-1"
  access_key = "XXXXXXXXXX"
  secret_key = "XXXXXXXXXX" 
}
```

今回はCIDRが 10.0.0.0/16 で名前が terraform-aws のVPCを作成します。

```
# Create a VPC
resource "aws_vpc" "example" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "terraform-aws"
  }
}
```

## terraform init

```
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 4.0"...
- Installing hashicorp/aws v4.55.0...
- Installed hashicorp/aws v4.55.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

## terraform plan

```
$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
  + create

Terraform will perform the following actions:

  # aws_vpc.example will be created
  + resource "aws_vpc" "example" {
      + arn                                  = (known after apply)
      + cidr_block                           = "10.0.0.0/16"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_classiclink                   = (known after apply)
      + enable_classiclink_dns_support       = (known after apply)
      + enable_dns_hostnames                 = (known after apply)
      + enable_dns_support                   = true
      + enable_network_address_usage_metrics = (known after apply)
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags                                 = {
          + "Name" = "terraform-aws"
        }
      + tags_all                             = {
          + "Name" = "terraform-aws"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run
"terraform apply" now.
```

## terraform apply

```
$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
  + create

Terraform will perform the following actions:

  # aws_vpc.example will be created
  + resource "aws_vpc" "example" {
      + arn                                  = (known after apply)
      + cidr_block                           = "10.0.0.0/16"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_classiclink                   = (known after apply)
      + enable_classiclink_dns_support       = (known after apply)
      + enable_dns_hostnames                 = (known after apply)
      + enable_dns_support                   = true
      + enable_network_address_usage_metrics = (known after apply)
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags                                 = {
          + "Name" = "terraform-aws"
        }
      + tags_all                             = {
          + "Name" = "terraform-aws"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_vpc.example: Creating...
aws_vpc.example: Creation complete after 2s [id=vpc-xxxxx]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

## terraform destroy

環境を削除する場合は terraform destroy

```
$ terraform destroy
aws_vpc.example: Refreshing state... [id=vpc-xxxxx]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
  - destroy

Terraform will perform the following actions:

  # aws_vpc.example will be destroyed
  - resource "aws_vpc" "example" {

  ～～　中略　～～

    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_vpc.example: Destroying... [id=vpc-xxxxx]
aws_vpc.example: Destruction complete after 1s

Destroy complete! Resources: 1 destroyed.
```

VPCが削除されていることを確認
