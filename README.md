# aws_opencms

This application is an attempt to create opencms on AWS with three tiered architecture. It comprises cloudformation to create the stack. The automation is enabled through ansible starting from stack creation to application deployment. 

Setup [Opencms](http://www.opencms.org/en/)  on your AWS with Ansible and Cloudformation, you can install it from your laptop by making itself a Workstation

If you are fluent with [Ansible](http://docs.ansible.com/ansible/intro_getting_started.html) and [Cloudformation](http://odecee.com.au/cloudformation-and-ansible/), you know where to look and what to do. If you are new to Ansible and cloudformation, just follow along with step-by-step instructions below.


## Pre-requisites
* [Install git](https://git-scm.com/downloads)

* Install recent version of [Ansible](http://docs.ansible.com/ansible/intro_installation.html)

* Create [AWS](https://console.aws.amazon.com/) account and generate security credentials.

* Create and Download private key from [AWS](https://console.aws.amazon.com/) account which would be used to access Instances.


## Simple installation

Clone the aws_opencms repo, and start up Ansible:

```bash

git clone https://github.com/mrigeshpriyadarshi/aws_opencms.git
cd aws_opencms/ansible

```

Update [set_ansible_env.sh](set_ansible_env.sh) (Mac or Linux) with environment variables for Ansible parameters and AWS creds.

```
export ANSIBLE_HOSTS=./ec2.py 
export EC2_INI_PATH=./ec2.ini 
export ANSIBLE_HOST_KEY_CHECKING=False
export AWS_ACCESS_KEY_ID=<ID>
export AWS_SECRET_ACCESS_KEY=<KEY>
export AWS_ACCESS_KEY=$AWS_ACCESS_KEY_ID
export AWS_SECRET_KEY=$AWS_SECRET_ACCESS_KEY
export AWS_DEFAULT_REGION=us-east-1
export AWS_REGION=$AWS_DEFAULT_REGION


```

We have divided the whole implentation in four roles:-

* network - It would create the AWS infrastructure.

* db - It would install and configure MySQL on DB instance.

* app - It would install and configure JAVA, Tomcat and Opencms on App instance

* web - It would install and configure Apache on Web instance


Update [Ansible Vars](./vars/network/cloudformation.yaml) YAML with relevant data.

```
---
bootstrap:
        StackName: mediacorp
        VpcCidrBlock: 10.11.0.0/24
        Environment : production
        WebSubnetCIDR : 10.11.0.0/26
        AppSubnetCIDR : 10.11.0.64/26
        DataSubnetCIDR : 10.11.0.128/25
        StackAvailabilityZone : us-east-1a
        InstanceType : t2.micro
        AMI : ami-46c1b650
        KeyPairName : build

```

Also, We have a [common Variable file](./vars/common.yaml) which contains the respective names of Cloudformation Stacks

```
---
stacks:
        network: BootstrapNetStack
        web: WebInstanceStack
        app : AppInstanceStack
        db : DbInstanceStack

```

## Add SSH Private Key in Forwading Agent

Add the Private pem file in SSH forwarding agent for seamless Ansible execution

```
ssh-add ~/Downloads/build.pem 

```

Afer the adding it forward agent, you can check if it properly added.

```
$ ssh-add -l
2048 SHA256:2zb3gGB7pvr+56b5oeJvcCA4lXrHKoMMC0/4yINui9Y /Users/mpriyada/Downloads/build.pem (RSA)

```

Then, execute following commands for Ansible to create the instance

```bash

# To download extra Ansible modules for Cloudformation
ansible-playbook -i bootstrap.ini bootstrap-init.yaml

# To create Infrastructure comprising 3 instances inside a VPC with respective subnets, it uses `bootstrap-network.json` CFN to create the stack. 
ansible-playbook -i bootstrap.ini cloudformation.yaml --extra-vars='stack=network stack_action=create'

# To create Install and configure Database Server 
ansible-playbook  db.yaml

# To create Install and configure OpenCMS Server 
ansible-playbook  app.yaml

# To create Install and configure Web Server 
ansible-playbook  web.yaml

# After all execution, generate the App URL which needs to accessed
ansible-playbook -i bootstrap.ini get_app_url.yaml 

```

These command will create whole stack and then install the OpenCMS. After Completion you can hit the Webserver IP in browser.

If something went wrong, jump to [Troubleshooting](https://github.com/mrigeshpriyadarshi/aws_opencms#common-problems-and-solutions) section below.


## Customize your installation

To evaluate this app , please consider using the following boxes for best results:

* ami-46c1b650 for CentOS 7.4


## Common problems and solutions

#### AWS Credentials missing from ENV

In the event you receive an error related to `client authentication error`, then execute the [set_ansible_env.sh](set_ansible_env.sh). For example:

```
    . set_ansible_env.sh
```

## Deletion of Stack

Once the testing completes, you can delete the stack to save the extra cost on AWS.

```
ansible-playbook -i bootstrap.ini cloudformation.yaml --extra-vars='stack=network stack_action=delete'

```


## Contribute

1. Fork it
1. Create your feature branch (git checkout -b my-new-feature)
1. Commit your changes (git commit -am 'Add some feature')
1. Push to the branch (git push origin my-new-feature)
1. Create new Pull Request


## License

|  |  |
| ------ | --- |
| **Author:** | Mrigesh Priyadarshi |
| **Copyright:** | [Mrigesh Priyadarshi](mailto:mrigeshpriyadarshi@gmail.com) |
| **License:** | Apache License, Version 2.0 |

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

See [LICENSE](license) for more information.

ansible-playbook -i bootstrap.ini bootstrap-init.yaml
