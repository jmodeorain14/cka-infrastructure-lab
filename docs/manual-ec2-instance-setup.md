# Manual EC2 Provisioning

## Overview

This guide documents the manual provisioning of an Amazon EC2 instance running Ubuntu Server through the AWS Management Console.

The EC2 instance is used as a remote Linux environment for Certified Kubernetes Administrator (CKA) preparation.

The instance is accessed securely using SSH from a local Linux workstation.

## Prerequisites

- An AWS account with permissions to manage EC2 resources
- Access to the AWS Management Console
- A local Linux workstation with OpenSSH installed
- An understanding of AWS Free Tier eligibility and potential charges

## 1. Configure the AWS Region

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. Navigate to **EC2**.
3. Select the desired AWS Region from the upper-right corner.

Use the same Region consistently when managing the instance and its associated resources.

## 2. Launch the EC2 Instance

Navigate to **EC2 → Instances → Launch instances**.

### Name and tags

Set the instance name:

`cka-ubuntu-lab`

### Application and OS Images

Select the official Canonical Ubuntu Server AMI:

- Operating system: Ubuntu Server 24.04 LTS
- Architecture: 64-bit (x86_64)
- Image type: Standard Ubuntu Server

Avoid selecting paid Marketplace or Ubuntu Pro images unless explicitly required.

### Instance type

Select:

`t3.small`

This instance type provides 2 vCPUs and 2 GiB of memory.

It is sufficient for an initial Ubuntu terminal environment, and should be adequate for running a Kubernetes control plane.

Confirm Free Tier eligibility and pricing before launching.

### Key pair

Create a new SSH key pair:

| Setting | Value |
|---|---|
| Key pair name | `cka-ubuntu-key` |
| Key pair type | ED25519 |
| Private key file format | .pem |

Download the private key and store it securely on your local workstation.

**Important:** Never commit private keys to Git.

### Network settings

Configure:

| Setting | Value |
|---|---|
| VPC | Default VPC |
| Subnet | Public subnet |
| Auto-assign public IP | Enabled |
| Security group | `cka-ssh-sg` |

Create the following inbound security group rule:

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | My IP (`/32`) |

This restricts SSH connectivity to your workstation's public IPv4 address.

**Important:** Do not expose SSH to all IPv4 addresses (`0.0.0.0/0`) for routine access.

### Storage

Configure the root EBS volume:

| Setting | Value |
|---|---|
| Storage type | General Purpose SSD (gp3) |
| Size (GiB) | 20 |
| Encrypted | Encrypted |

Verify the volume settings and any applicable charges.

### Launch

Review the configuration and select **Launch instance**.

Wait until the instance is running and its status checks have passed.

## 3. Configure SSH Access

### Store the private key

On the local Linux workstation:

```bash
mkdir -p ~/.ssh
mv ~/Downloads/cka-ubuntu-key.pem ~/.ssh/
chmod 700 ~/.ssh
chmod 400 ~/.ssh/cka-ubuntu-key.pem
```

The private key must remain outside the Git repository.

### Retrieve the public IP address

Navigate to **EC2 → Instances** and select the instance.

Copy its **Public IPv4 address**.

### Connect using SSH

Run:

```bash
ssh -i ~/.ssh/cka-ubuntu-key.pem ubuntu@<PUBLIC_IP>
```

Replace `<PUBLIC_IP>` with the actual address.

The default username for official Ubuntu EC2 images is `ubuntu`.

Verify the server's SSH host-key fingerprint through a trusted source before accepting it on the first connection.

### Optional SSH configuration

To simplify subsequent connections, edit `~/.ssh/config` on the local workstation:

```sshconfig
Host cka-ubuntu-lab
    HostName <PUBLIC_IP>
    User ubuntu
    IdentityFile ~/.ssh/cka-ubuntu-key.pem
    IdentitiesOnly yes
```

Protect the configuration:

```bash
chmod 600 ~/.ssh/config
```

Connect with:

```bash
ssh cka-ubuntu-lab
```

**Note:** The public IPv4 address may change after stopping and starting the instance.

## 4. Verify the Ubuntu Environment

After connecting to the instance, run:

```bash
# Verify the operating system
lsb_release -a

# Verify the system architecture
uname -m

# Inspect CPU information
lscpu

# Inspect available memory
free -h

# Inspect disk space
df -h

# Check outbound connectivity
curl -I https://ubuntu.com
```

Optionally install available updates:

```bash
sudo apt update
sudo apt upgrade -y
```

Disconnect from the instance:

```bash
exit
```

## 5. Troubleshooting

### Permission denied (publickey)

- Verify that the SSH username is `ubuntu`.
- Confirm that the private key matches the EC2 key pair.
- Verify the private key file permissions.

### Connection timed out

- Confirm that the instance is running.
- Verify its public IPv4 address.
- Check that the security group allows SSH from your current public IPv4 address.
- Verify that the subnet has appropriate internet routing.

### SSH host-key warning

An unexpected host-key change can indicate that the server or its public IP address has changed.

Verify the server's identity before modifying local SSH host-key records.

Never disable strict SSH host-key checking simply to bypass a warning.

## 6. Cost Management

An EC2 instance may incur charges while running, even when idle.

Free Tier eligibility depends on the AWS account, instance configuration, resource usage, and current AWS pricing.

To stop the instance:

1. Open **EC2 → Instances**.
2. Select `cka-ubuntu-lab`.
3. Choose **Instance state → Stop instance**.

Stopping the instance avoids ongoing instance compute charges, but attached EBS volumes and other retained resources may still incur costs.

To permanently remove the environment, terminate the instance and review associated resources, including EBS volumes and public IPv4 allocations.

## 7. Security Considerations

- Keep AWS credentials and private SSH keys outside the repository.
- Limit inbound SSH access to trusted source IP addresses.
- Avoid using the AWS root user for routine administration.
- Keep the Ubuntu system updated.
- Use EBS encryption where available.
- Monitor AWS costs and resource usage.
- Do not expose unnecessary network ports.
- Do not commit environment-specific sensitive information.

## 8. Next Steps

The next stage is to reproduce the infrastructure using Terraform.

The Terraform implementation should preserve the important characteristics of the manually provisioned environment, including the Ubuntu AMI, EC2 instance type, storage configuration, SSH authentication, and restricted network access.

## References

- [Amazon EC2 Getting Started](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
- [Launch an EC2 Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/LaunchingAndUsingInstances.html)
- [AWS EC2 Free Tier Usage](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-free-tier-usage.html)
- [Ubuntu on AWS Documentation](https://ubuntu.com/aws/docs/)

