# Set Up EC2 Instance Connect<a name="ec2-instance-connect-set-up"></a>

Amazon Linux 2 2\.0\.20190618 or later comes preconfigured with EC2 Instance Connect\. For other supported Linux distributions, you need to set up Instance Connect for every instance that will provide the Instance Connect capability for connecting\. This is a one\-time requirement for each instance\. 

**Topics**
+ [Step 1: Install EC2 Instance Connect on an Instance](#ec2-instance-connect-install)
+ [Step 2: \(Optional\) Install the EC2 Instance Connect CLI](#ec2-instance-connect-install-eic-CLI)
+ [Step 3: Configure IAM permissions for EC2 Instance Connect](#ec2-instance-connect-configure-IAM-role)

**Limitations**

If your SSH settings use or require `AuthorizedKeysCommand` and `AuthorizedKeysCommandUser`, it is not possible to install EC2 Instance Connect\.

If you have already configured `AuthorizedKeysCommand` and `AuthorizedKeysCommandUser`, the EC2 Instance Connect installation will not change the values and you will not be able to use Instance Connect\.

The following Linux distributions are supported for Instance Connect: 
+ Amazon Linux 2 \(any version\)
+ Ubuntu 16\.04 or later

**Prerequisites**
+ **Verify the general prerequisites for connecting to your instance using SSH\.**

  For more information, see [General Prerequisites for Connecting to Your Instance](connection-prereqs.md)\.
+ **Install an SSH client on your local computer\.**

  Your local computer most likely has an SSH client installed by default\. You can check for an SSH client by typing ssh at the command line\. If your local computer doesn't recognize the command, you can install an SSH client\. For information about installing an SSH client on Linux or macOS X, see [http://www\.openssh\.com](http://www.openssh.com/)\. For information about installing an SSH client on Windows 10, see [OpenSSH in Windows](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_overview)\.
+ **Install the AWS CLI on your local computer\.**

  To configure the IAM permissions and to install the EC2 Instance Connect CLI, you must use the AWS CLI\. For more information about installing the AWS CLI, see [Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html) in the *AWS Command Line Interface User Guide*\.
+ **\[Ubuntu\] Install the AWS CLI on your instance\.**

  To install EC2 Instance Connect on an Ubuntu instance, you must use the AWS CLI on the instance\. For more information about installing the AWS CLI, see [Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html) in the *AWS Command Line Interface User Guide*\.

## Step 1: Install EC2 Instance Connect on an Instance<a name="ec2-instance-connect-install"></a>

Installing EC2 Instance Connect configures the SSH daemon on the instance\. The procedure for installing EC2 Instance Connect is different for instances launched using Amazon Linux 2 and Ubuntu\.

**To install EC2 Instance Connect on an instance launched with Amazon Linux 2**

1. Connect to your instance using SSH\.

   Use the SSH key pair that was assigned to your instance when you launched it and the default user name of the AMI that you used to launch your instance\. For Amazon Linux 2, the default user name is `ec2-user`\.

   For example, if your instance was launched using Amazon Linux 2, your instance's public DNS is `ec2-a-b-c-d.us-west-2.compute.amazonaws.com`, and the key pair is `my_ec2_private_key.pem`, use the following command to SSH into your instance:

   ```
   $ ssh -i my_ec2_private_key.pem ec2-user@ec2-a-b-c-d.us-west-2.compute.amazonaws.com
   ```

   For more information about connecting to your instance, see [Connecting to Your Linux Instance Using SSH](AccessingInstancesLinux.md)\.

1. Install the EC2 Instance Connect package on your instance\.

   For Amazon Linux 2, use the yum install command\.

   ```
   [ec2-user ~]$ sudo yum install ec2-instance-connect
   ```

   You should see four new files in the `/opt/aws/bin/` folder:

   ```
   eic_curl_authorized_keys
   eic_harvest_hostkeys
   eic_parse_authorized_keys
   eic_run_authorized_keys
   ```

1. \(Optional\) Verify that Instance Connect was successfully installed on your instance\.

   Use the sudo less command to check that the `/etc/ssh/sshd_config` file was correctly updated as follows:

   ```
   [ec2-user ~]$ sudo less /etc/ssh/sshd_config
   ```

   Instance Connect was successfully installed if the `AuthorizedKeysCommand` and `AuthorizedKeysCommandUser` lines in the `/etc/ssh/sshd_config` file contain the following values:

   ```
   AuthorizedKeysCommand /opt/aws/bin/eic_run_authorized_keys %u %f
   AuthorizedKeysCommandUser ec2-instance-connect
   ```
   + `AuthorizedKeysCommand` sets the `eic_run_authorized_keys` file to look up the keys from the instance metadata
   + `AuthorizedKeysCommandUser` sets the system user as `ec2-instance-connect`
**Note**  
If you previously configured `AuthorizedKeysCommand` and `AuthorizedKeysCommandUser`, the Instance Connect installation will not change the values and you will not be able to use Instance Connect\.

**To install EC2 Instance Connect on an instance launched with Ubuntu 16\.04 or later**

1. Connect to your instance using SSH\.

   Use the SSH key pair that was assigned to your instance when you launched it and use the default user name of the AMI that you used to launch your instance\. For an Ubuntu AMI, the user name is `ubuntu`\.

   If your instance was launched using Ubuntu, your instance's public DNS is `ec2-a-b-c-d.us-west-2.compute.amazonaws.com`, and the key pair is `my_ec2_private_key.pem`, use the following command to SSH into your instance:

   ```
   $ ssh -i my_ec2_private_key.pem ubuntu@ec2-a-b-c-d.us-west-2.compute.amazonaws.com
   ```

   For more information about connecting to your instance, see [Connecting to Your Linux Instance Using SSH](AccessingInstancesLinux.md)\.

1. Install the Instance Connect package on your instance\.

   For Ubuntu, use the sudo apt\-get command to install the `.deb` package\.

   ```
   ubuntu:~$ sudo apt-get install ec2-instance-connect
   ```

   You should see four new files in the `/usr/share/ec2-instance-connect/` folder:

   ```
   eic_curl_authorized_keys
   eic_harvest_hostkeys
   eic_parse_authorized_keys
   eic_run_authorized_keys
   ```

1. \(Optional\) Verify that Instance Connect was successfully installed on your instance\.

   Use the sudo less command to check that the `/lib/systemd/system/ssh.service.d/ec2-instance-connect.conf` was correctly updated as follows:

   ```
   ubuntu:~$ sudo less /lib/systemd/system/ssh.service.d/ec2-instance-connect.conf
   ```

   Instance Connect was successfully installed if the `AuthorizedKeysCommand` and `AuthorizedKeysCommandUser` lines in the `/lib/systemd/system/ssh.service.d/ec2-instance-connect.conf` file contain the following values:

   ```
   AuthorizedKeysCommand /usr/share/ec2-instance-connect/eic_run_authorized_keys %u %f
   AuthorizedKeysCommandUser ec2-instance-connect
   ```
   + `AuthorizedKeysCommand` sets the `eic_run_authorized_keys` file to look up the keys from the instance metadata
   + `AuthorizedKeysCommandUser` sets the system user as `ec2-instance-connect`
**Note**  
If you previously configured `AuthorizedKeysCommand` and `AuthorizedKeysCommandUser`, the Instance Connect installation will not change the values and you will not be able to use Instance Connect\.

## Step 2: \(Optional\) Install the EC2 Instance Connect CLI<a name="ec2-instance-connect-install-eic-CLI"></a>

The EC2 Instance Connect CLI provides a similar interface to standard SSH calls, which includes querying EC2 instance information, generating and publishing ephemeral public keys, and establishing an SSH connection through a single command, `mssh instance_id`\.

**Note**  
There is no need to install the EC2 Instance Connect CLI if users only use the console or an SSH client to connect to an instance\.

**To install the EC2 Instance Connect CLI package**

1. Use the s3api get\-object command to download the `ec2instanceconnectcli-latest.tar.gz` package from Amazon S3 to your local computer using the AWS CLI, as follows:

   ```
   $ aws s3api get-object --bucket ec2-instance-connect --key cli/ec2instanceconnectcli-latest.tar.gz ec2instanceconnectcli-latest.tar.gz
   ```

1. Use `[pip](https://pip.pypa.io/en/stable/reference/pip_install/)` to install the `ec2instanceconnectcli-latest.tar.gz` package\. For more information about `pip`, see [https://pip.pypa.io/en/stable/](https://pip.pypa.io/en/stable/)\.

   ```
   $ sudo pip install ec2instanceconnectcli-latest.tar.gz
   ```

## Step 3: Configure IAM permissions for EC2 Instance Connect<a name="ec2-instance-connect-configure-IAM-role"></a>

If your IAM users will connect to an instance using EC2 Instance Connect, be sure to grant them the required permission to push the public key to the instance\. The following instructions explain how to create the policy and attach it to the user using the AWS CLI\. For information about how to do this using the AWS Management Console, see [Creating IAM Policies \(Console\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html#access_policies_create-start) and [Adding Permissions by Attaching Policies Directly to the User](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_change-permissions.html#users_change_permissions-add-directly-console) in the *IAM User Guide*\.

**Note**  
We currently do not support tag\-based authorization for Instance Connect\. Tags and context keys in the `ec2` namespace are not automatically applied to `ec2-instance-connect`\.

**To grant an IAM user permission for EC2 Instance Connect \(AWS CLI\)**

1. Create a JSON document that includes the following content for the new policy:
   + If your users only use the console or an SSH client to connect to an instance, use the following content for the policy:

     ```
     {
     	"Version": "2012-10-17",
     	"Statement": [{
     		"Effect": "Allow",
     		"Action": "ec2-instance-connect:SendSSHPublicKey",
     		"Resource": "arn:aws:ec2:region:account-id:instance/i-instance-id",
     		"Condition": {
     			"StringEquals": {
     				"ec2:osuser": "ami-username"
     			}
     		}
     	}]
     }
     ```
   + If your users use the EC2 Instance Connect CLI to connect to an instance, use the following content for the policy, which includes the `ec2:DescribeInstances` action:

     ```
     {
     	"Version": "2012-10-17",
     	"Statement": [{
     			"Effect": "Allow",
     			"Action": "ec2-instance-connect:SendSSHPublicKey",
     			"Resource": "arn:aws:ec2:region:account-id:instance/i-instance-id",
     			"Condition": {
     				"StringEquals": {
     					"ec2:osuser": "ami-username"
     				}
     			}
     		},
     		{
     			"Effect": "Allow",
     			"Action": "ec2:DescribeInstances",
     			"Resource": "*"
     		}
     	]
     }
     ```
   + The `ec2-instance-connect:SendSSHPublicKey` action grants an IAM user permission to push the public key to an instance\.
   + The `ec2:osuser` condition specifies the default user name for the AMI that you used to launch your instance\. For Amazon Linux 2, the default user name is `ec2-user`\. For the Ubuntu AMI, the default user name is `ubuntu`\.
   + \(Connecting via EC2 Instance Connect CLI only\) The `ec2:DescribeInstances` action is required because the wrapper makes a describe call at the first step\.

   For more information, see [Actions, Resources, and Condition Keys for Amazon EC2](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonec2.html) in the *IAM User Guide*\.

1. Use the [create\-policy](https://docs.aws.amazon.com/cli/latest/reference/iam/create-policy.html) command to create a new managed policy, and specify the JSON document that you created to use as the content for the new policy\.

   ```
   $ aws iam create-policy --policy-name my-policy --policy-document file://JSON-file-name
   ```

1. Use the [attach\-user\-policy](https://docs.aws.amazon.com/cli/latest/reference/iam/create-policy.html) command to attach the specified managed policy to the specified IAM user\. For the `--user-name` parameter, specify the friendly name \(not ARN\) of the IAM user\. 

   ```
   $ aws iam attach-user-policy --policy-arn arn:aws:iam::account-id:policy/my-policy --user-name IAM-friendly-name
   ```