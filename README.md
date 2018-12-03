Ansible Pip Bundler
=========

This is an Ansible project to automate creation of pip installation bundles. The resulting bundle is a tar.bz archive, which can be used to install python packages (and their dependencies) on hosts that are unable to reach traditional PyPI sources, i.e. "air gapped" hosts. The bundle is created in an Amazon Web Services (AWS) EC2 instance. All AWS resouces are ephemeral and will be terminated/deleted by the playbook upon successful completion.

Requirements
------------

The following python packages are required on the host where the playbook is executed:

* ansible (tested on v2.7.4)
* boto (tested on v2.49.0)
* boto3 (tested on v1.9.55)
* botocore (tested on v1.12.55)

The project requires access keys for an IAM account with permissions for the following:

* Create EC2 security groups.
* Launch/terminate EC2 instances.
* Delete EBS volumes.
* Use an existing key pair.

The user executing the playbook must have read access to the private key from the key pair that's assigned to the EC2 instance.

Variables
--------------

A `vars.yml` file is included in the project repo and can be modified accordingly prior to executing the playbook. Variable names and descriptions are listed below.

* `aws_secret_access_key`: Secret access key for authentication with AWS API. Leave this commented out if you are sourcing access key from the ansible-iam-keys role.
* `aws_access_key_id`: Access key ID for authentication with AWS API. Leave this commented out if you are sourcing acces key from the ansible-iam-keys role.
* `source_keys_from_role`: Boolean to indicate whether or not you are sourcing access key from the ansible-iam-keys role. Comment this out if you are explicitly defining secret access key and access key ID in vars file.
* `my_ssh_key_name`: Name of key pair for SSH authentication with launched instance. This allows ansible to connect to instance and create/collect installation bundle.
* `my_ec2_region`: EC2 region where instance is launched.
* `my_ami_id`: ID for Amazon Machine Image (AMI) used to launch instance.
* `my_instance_type`: EC2 instance type, `t2.micro` is recommended and free-tier eligible.
* `my_pip_package`: Name of desired pip package to be installed.

Example
----------------

This `vars.yml` file is configured to create an installation bundle for the `ansible-lint` package by launching a t2.micro EC2 instance from the standard CentOS 7 AMI in the us-east-1 region:

    # authentication
    #aws_secret_access_key: ''
    #aws_access_key_id: ''
    source_keys_from_role: true
    my_ssh_key_name: 'rycummins'

    # ec2 stuff
    my_ec2_region: 'us-east-1'
    my_ami_id: 'ami-9887c6e7'
    my_instance_type: 't2.micro'

    # pip stuff
    my_pip_package: 'ansible-lint'

Since the example is sourcing access key values from a role instead of explicitly them in the vars file, the following `ansible-galaxy` command should be run prior to executing the playbook to ensure the latest version of the role is installed:

    $ ansible-galaxy install -r requirements.yml -p roles --force

Execute the playbook as follows:

    $ ansible-playbook -u centos --private-key /path/to/private.pem --vault-password-file /path/to/ansible_vault.pass pip_bundle_ec2.yml

License
-------

MIT

Author Information
------------------

Ryan Cummins
