+++
title = '用 terraform 建立一個 EC2 Server'
date = 2024-05-23T23:30:49+08:00
draft = false
categories = ['Server']
tags = ['terraform', 'aws', 'ec2']
+++
## 前言

最近想認真學 CI / CD 相關技術，所以想先從 GitHub action 和 terraform 開始，而按照開發流程來說，先建立一台開發 / 實驗用的 Server 一定是 Hello world 一般的開始了，這篇文章會按照 terraform 建立一台開發用的 EC2 server。

<https://developer.hashicorp.com/terraform/tutorials/aws-get-started>

## 步驟

在這次的步驟中我假設你已經有了 AWS 的基礎知識，並且安裝好了 terraform，我們就可以按照以下幾個步驟進行實作。

- Terraform 步驟
- main.tf 內容解釋
- 建立 Server
- 實際連線測試

## Terraform 步驟

一個基本的 Terraform 使用流程，會有以下幾個步驟。

- init
  - 安裝各種插件，像是 Provider(註1)。
- validate
  - 事先驗證語法是否正確，但不保證 runtime 不會有 error。
- plan
  - 檢視即將建立怎麼樣的資源。
- apply
  - 實際在該服務上建立資源。
- show
  - 觀察已建立的資源的資訊，以 EC2 為例，我們就可以看到我們建立的 Instance 的 IP, Image 及區域等等資訊。
- change
  - 更改已建立的資源的內容。
- destroy
  - 將建立起的資源刪除掉。

> *註1: Terraform 中用來與對應的服務(像 AWS)溝通的插件*
>

## main.tf 內容

以下是我們的 main.tf 內容，也就是描述我們資源內容的 terraform 檔案，為了可讀性，以下提供兩個版本，一個有附上註解一個沒有，方便讀者比對。

```bash
# 無註解版
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "ap-northeast-3"
}

resource "aws_instance" "dev_server" {
  ami           = "ami-0f33119c704c2aa59"
  instance_type = "t2.micro"

  tags = {
    Name = "MyDevServerInstance"
  }
}
```

```bash
# 有註解版
# terraform block 中是 Terraform 的設定內容，
# 像是 provider 以及版本選擇都在這個區塊進行設定
terraform {
 # 設定我們這個檔案需要哪些 provider
 # 並且設定 provider 的來源以及其版本
 # hashicorp/aws => registry.terraform.io/hashicorp/aws
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }
  
 # 定義這個檔案需要跑在 terraform 的版本上
  required_version = ">= 1.2.0"
}

# provider 的初始設定
# 以 aws 為例就需要設定其 region, access key id, secret access key 等
# 但由於後兩個資料有機密性，所以放在 local 端，讓 aws cli 自行讀取
# 以防誤傳上 github 等外部程式庫
provider "aws" {
 # osaka
  region = "ap-northeast-3"
}

# 定義資源內容
# 格式:
# resource <resource type> <resource_name>
resource "aws_instance" "dev_server" {
 # 請注意在不同的 region 的同一個 AMI 會有不同的 ID
 # 所以請記得對應 region 填入對應的 AMI ID
  ami           = "ami-0f33119c704c2aa59"
  instance_type = "t2.micro"
  
  tags = {
    Name = "MyDevServerInstance"
  }
}
```

所以透過以上的程式碼，我們就可以得到一個建立在 ap-northeast-3(Osaka)，以 t2.micro 規格運行 Amazon Linux 2 AMI 的 EC2 Instance 了。

## 建立 Server

透過以下指令，我們可以建立對應的資源了，輸入之後他會要求你輸入 yes 進行確認，記得輸入後就可以順利建立伺服器。

```bash
terraform apply
```

看到以下這行就是創建成功了

```bash
Apply complete! Resources: 1 added, 0 changed, 0 destroyed
```

## 實際連線測試

透過以下指令可以看到資源的詳細資訊，於是我們就使用該 IP 進行 ssh 連線。

```bash
terraform show

# aws_instance.dev_server:
resource "aws_instance" "dev_server" {
    ami                                  = "ami-0f33119c704c2aa59"
  
  ...
  
    public_dns                           = "ec2-13-208-xxx-xxx.ap-northeast-3.compute.amazonaws.com"
    public_ip                            = "13.208.xxx.xxx"
    secondary_private_ips                = []
    security_groups                      = [
        "default",
    ]
    ...
}
```

但是我們這裡遇到兩個問題，沒辦法連上 server

1. 由於沒指定 security_groups，沒有開放 server 的 22 port
2. 直接透過 ssh 連上 EC2 instance 需要 key pair。

於是我們透過更改 main.tf 的內容，新增 subnet_id 和 vpc_security_group_ids 以加入我有開放 22 port 的 security group 和對應的 subnet，並且加上新的 key pair 資源，這樣我們才能連上 server。

```bash
# 無註解版
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "ap-northeast-3"
}

resource "aws_key_pair" "dev_server_key" {
  key_name   = "dev-server-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

resource "aws_instance" "dev_server" {
  ami           = "ami-0f33119c704c2aa59"
  instance_type = "t2.micro"
  
  key_name = aws_key_pair.dev_server_key.key_name
  
  subnet_id = "subnet-xxxxxxxx"

  vpc_security_group_ids = ["sg-xxxxxxxx"]

  tags = {
    Name = "MyDevServerInstance"
  }
}
```

再次執行 apply 以更新我們的 server，就可以更新我們的伺服器資源，另外我們也可以再次執行 show 就可以看到我們的 IP 更新了，代表這個操作是直接刪除並重新建立 server。

```bash
ssh ec2-user@xxx.xxx.xxx.xxx

   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$
```

最後要記得下 destroy 指令來關掉機器，不然在沒有 free-tier 的狀況下，一直開著就是一直在噴錢呢。

## 結語

跟著官方教學進行機器的建置，基本上不會有什麼問題，最大的問題就是我想連上去當開發機使用，畢竟 terraform 應該是為了快速部署所建立的服務。

但是透過這篇文章把遇到的幾個問題解決之後，也更清楚 AWS 建立 EC2 Instance的流程了，也是另外的收穫~
