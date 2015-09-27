# Web Message Mini-project

*Automation for the People!*

This project uses [Ansible] to deploy and configure an EC2 instance running in a VPC.
The launched instance is configured with Nginx and a message *Automation for the People!* is the index.html file in the document root.

The AWS Regions supported: `us-east-1` and `us-west-2`.

The project supports multiple environments which are isolated from each other: `dev`, `test`, and `production`

The playbook `deploy-web-message.yml` will create an environment-specific VPC (e.g. `production`, `test`, `dev`), with Subnets, Internet Gateway, and default route table entry for Internet access via the Interet Gateway (no NAT).
It will also create necessary Security Groups and launch a `t2.micro` instance into the first Subnet.
Visiting the instance's public DNS name, or public IP address, will show a web page with the message: **Automation for the People**

[Ansible]: http://www.ansible.com

## Dependencies

The project requires [Ansible] 1.9.3 and Python [Boto] library.
The playbook, `deploy-web-message.yml`, makes use of the Ansible AWS dynamic inventory script, which is checked into the project at `inventory/ec2.py`.

The easiest way to install Ansible is with `pip`:

    sudo pip install ansible==1.9.3

[Boto]: https://boto.readthedocs.org/en/latest/index.html

## AWS Credentials
If you execute this playbook from an EC2 instance, the API calls made by Boto will use the IAM Role/Instance Profile.
If you need to use other credentials, you can use one of the [Boto config] mechanisms.

[Boto config]: https://boto.readthedocs.org/en/latest/boto_config_tut.html

For example, to use a [shared credentials] profile named `miniproject`:
**$HOME/.aws/credentials**

```ini
# arn:aws:iam::123456789012:user/miniproject
[miniproject]
aws_access_key_id = < access key for this profile >
aws_secret_access_key = < secret key for this profile >
```
[shared credentials]: https://boto.readthedocs.org/en/latest/boto_config_tut.html#credentials

### AWS Policies required

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "IAM",
            "Effect": "Allow",
            "Action": [
                "iam:ListRoles",
                "iam:CreateRole",
                "iam:CreateInstanceProfile",
                "iam:AddRoleToInstanceProfile",
                "iam:PassRole",
                "ec2:*",
                "rds:Describe*",
                "route53:List*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

## Usage

To create an `Automation for the People!` message on a EC2 web server in the US-East-1 AWS Region, using the `miniproject` credentials profile:


```shell
AWS_PROFILE=miniproject ansible-playbook -i inventory deploy-web-message.yml --extra-vars "profile=miniproject region=us-east-1 env=production ansible_iface=public" --private-key /path/to/keypair/named/ec2.pem
```


### Notification when complete

At the end of execution, if the playbook is run from an a Macintosh system, your computer will audibly tell you via the Ansible [osx_say] module that:
> Your web message, *Automation for the People!*, is live.

[osx_say]: https://docs.ansible.com/ansible/osx_say_module.html

### EC2 Keypair

This project does not create an EC2 Keypair for the launched instance. The default Keypair name is `ec2`.
You can override the keypair used by setting a `keypair=<KEYPAIR NAME>` via the `--extra-args` argument.

### Unix socket path length limitation
If you receive an error similar to this:
`SSH Error: unix_listener: "/LONG/PATH/TO/SSH/CONTROL/SOCKET" too long for Unix domain socket`

You will need to override the Ansible `control_path` configuration directive to a shorter form, in your [ansible.cfg] file.

You can copy the file from Ansible 1.9.3 tag in Github `examples/ansible.cfg` to `~/.ansible.cfg` and override the `control_path` directive: `control_path = %(directory)s/%%h-%%r`

[ansible.cfg]: https://github.com/ansible/ansible/blob/v1.9.3-1/examples/ansible.cfg#L194
