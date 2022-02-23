# terraform-qingcloud

Terraform-Qingcloud-Plugin [![CircleCI](https://circleci.com/gh/yunify/terraform-provider-qingcloud/tree/master.svg?style=svg)](https://circleci.com/gh/yunify/terraform-provider-qingcloud/tree/master)
[![codebeat badge](https://codebeat.co/badges/d6cc83ea-779f-4fce-8091-abc0b719d271)](https://codebeat.co/projects/github-com-yunify-qingcloud-terraform-provider-master-3c5cd450-e81b-4eb1-aaf6-aa9b76158d6f)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fyunify%2Fterraform-provider-qingcloud.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fyunify%2Fterraform-provider-qingcloud?ref=badge_shield)

## Usage

### Install terraform-provider-qingcloud

To install Terraform, find the [appropriate package](https://github.com/yunify/terraform-provider-qingcloud/releases) for your system and download it. Terraform is packaged as a tgz archive.  
After downloading Terraform, unzip the package.   
On Linux or Mac , put the binary file to in the sub-path .terraform.d/plugins in your user's home directory.
On Windows , put the binary file to in the sub-path terraform.d/plugins beneath your user's "Application Data" directory.
Then put the binary file into terraform 's PATH.

### Verifying the Installation

```shell
git clone https://github.com/yunify/terraform-provider-qingcloud.git
cd ./terraform-provider-qingcloud/terraform/example/init
terraform init
terraform -v
```

You can execute the above script . If you installed the provider correctly, you should see output similar to the one below .  

```shell
Terraform v0.13.0
+ provider registry.terraform.io/yunify/qingcloud v1.2.6
```

## Finish Resource:

- [x] Instance
- [x] Volume
- [x] Vxnet
- [ ] Router(Deprecated,Use Vpc in SDN2.0)
- [x] Eip
- [x] SecurityGroups
- [x] SecurityGroupRules
- [x] Keypairs
- [x] Vpc
- [x] Tag
- [x] VpcStatic
- [x] LoadBalancer
- [x] LoadBalancerListener
- [x] LoadBalancerBackend
- [x] Server Certificate
- [x] VPN Cert

## Contributing

1. Fork it ( https://github.com/yunify/terraform-provider-qingcloud/fork )
2. Create your feature branch (`git checkout -b new-feature`)
3. Commit your changes (`git commit -asm 'Add some feature'`)
4. Push to the branch (`git push origin new-feature`)
5. Create a new Pull Request    

## Special Thanks

[CuriosityChina](https://github.com/CuriosityChina)

---
---

## dcm readme

#### 打包
- 修改Makefile第一行
- 执行命令：```make```

#### how to use test
> 以下更改仅用于本地测试用，打包时请还原
1. 在test方法run的environment里加上```TF_ACC=1```表示真跑test
2. 然后修改provider.go文件的最后的init方法，填写正确的ak/sk之类的信息， 修改providerConfigure方法，直接赋值相关信息
```
# provider.go的func providerConfigure 中 修改一段

var accesskey string  = "我的ak"
secretkey := "我的sk"
zone := "LFRZ1"
endpoint := "https://api.enncloud.cn/iaas/"

config := Config{
    ID:       accesskey,
    Secret:   secretkey,
    Zone:     zone,
    EndPoint: endpoint,
}
```
3. 在import_qingcloud_instance_test.go里面
```
PreCheck:     func() { 
    //testAccPreCheck(t)  // 注释掉provider校验
},

//Providers:    testAccProviders, // 注释掉本行， 跳过ak，sk校验(会从os环境变量中取)，改为下面这样
Providers: map[string]terraform.ResourceProvider{
    "qingcloud": Provider().(*schema.Provider),
},

//Config: fmt.Sprintf(testAccInstanceConfig, testTag),
Config: fmt.Sprintf(testDcmInstanceConfig, testTag), // 其中testDcmInstanceConfig为terraform脚本内容
```
4. 在resource_qingcloud_instance_test.go里加入：
```
const testDcmInstanceConfig = `
resource "qingcloud_tag" "test"{
    name="%v"
}
resource "qingcloud_instance" "dcm001" {
    name = "dcm-tarraform-sourcecode1"
    image_id = "img-jrmtompa"
    cpu = 1
    memory = 1024
    os_disk_size = "50"
    login_passwd = "1qazXSW@"
    managed_vxnet_id = "vxnet-0"
    tag_ids = ["${qingcloud_tag.test.id}"]
}
`
```

#### code change log
1. 调整安全组规格类型的校验，加了一些安全组规则类型
```
# 更改 resource_qingcloud_security_group_rule.go第37行为：
ValidateFunc: withinArrayString("tcp", "udp", "icmp", "gre", "esp", "ah", "ipip", "vrrp", "ipv6", "ipv6-icmp", "ipencap", "all"),
```