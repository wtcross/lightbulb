Ansible AWS training provisioner
================================

This is an automated lab setup for Ansible training. It creates five nodes per user in the `users` list.

* One control node from which Ansible will be executed from and where Ansible Tower can be installed
* Two web nodes that coincide with the two nodes in lightbulb's original design

## Usage ##


### AWS Setup ###

The `provision.yml` playbook creates instances, configures them for password authentication, creates an inventory file for each user with their IPs and credentials, and emails every user their respective inventory file. An instructor inventory file is also created in the current directory which will let the instructor access the nodes of any student by simply targeting the the username as a host group. The lab is created in `us-east-1` by default.

**Note:** Emails are sent _every_ time the playbook is run. To prevent emails from being sent on subsequent runs of the playbook, add `email: no` to `vars/main.yml`.

To set up the lab for Ansible training, follow these steps.

1. Create an Amazon AWS account.

2. Create an ssh key pair called 'ansible' (Network & Security->Key Pairs->Create Key Pair). Download the private key to your `.ssh` directory, e.g. to `.ssh/ansible.pem`. Alternatively, you can upload your own public key into AWS.

      If using an AWS generated key add it to the ssh-agent:

        ssh-add ~/.ssh/ansible.pem

3. Create an [Access Key ID and Secret Access Key](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSGettingStartedGuide/AWSCredentials.html).

1. Install `boto`.

        pip install boto

1. Create a `boto` configuration file containing your AWS access key ID and secret access key.

    ```bash
    mkdir ~/.aws
    touch ~/.aws/credentials
    chmod 600 ~/.aws/credentials

    # The file should contain the following:
    [default]
    aws_access_key_id = [access key ID]
    aws_secret_access_key = [secret key]
    ```

4. Create a free [Sendgrid](http://sendgrid.com) account if you don't have one. Optionally, create an API key to use with this the playbook.

5. Install the `sendgrid` python library:

    **Note:** The `sendgrid` module does not work with `sendgrid >= 3`. Please install the latest `2.x` version.

        pip install sendgrid==2.2.1

6. Clone the lightbulb repo:

        git clone https://github.com/ansible/lightbulb.git
        cd lightbulb/aws_lab_setup

7. Define the following variables, either in a file passed in using `-e @extra_vars.yml` or directly in a `vars` section in `aws_lab_setup/vars/main.yml`:

      ```yaml
      ec2_key_name: username                # SSH key in AWS to put in all the instances
      ec2_region: us-west-1                 # region where the nodes will live
      ec2_name_prefix: TRAINING-LAB         # name prefix for all the VMs
      ec2_vpc_id: vpc-1234aaaa              # EC2 VPC ID in your region
      ec2_vpc_subnet_id: subnet-5678bbbb    # EC2 subnet ID in your VPC
      sendgrid_user: username               # username for the Sendgrid module
      sendgrid_pass: 'passwordgoeshere'     # sendgrid accound password
      sendgrid_api_key: 'APIkey'            # Instead of username and password, you may use an API key. Don't define both.
      instructor_email: 'Ansible Instructor <helloworld@acme.com>'  # address you want the emails to arrive from
      admin_password: changeme123           # Set this to something better if you'd like. Defaults to 'LearnAnsible[two digit month][two digit year]', e.g., LearnAnsible0416
      ansible_pip_version: '2.4.4'              # Set to desired ansible version. Not required. Will default to latest. Latest may cause errors with modules used in labs
      tower_license: ''                     # Set to the text of the license from https://www.ansible.com/workshop-license
      student_count: 1
      ssh_port: 22
      ```

8. Run the playbook:

        ansible-playbook provision.yml

9. Check on the EC2 console and you should see instances being created like:

        TRAINING-LAB-<student_username>-node1|2|3|haproxy|tower|control

If successful all your students will be emailed the details of their hosts including addresses and credentials, and an `instructor_inventory.txt` file will be created listing all the student machines.


### AWS Teardown ###

The `teardown.yml` playbook deletes all the training instances as well as local inventory files.

To destroy all the EC2 instaances after training is complete:

1. Run the playbook:

        ansible-playbook teardown.yml
