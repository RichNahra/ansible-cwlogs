# CloudWatch Logs Agent Playbook

This playbook installs and configures AWS CloudWatch agent on servers hosted on premises.

> Note 1: This playbook has been tested on Ubuntu 20.04

> Note 2: Metrics require `collectd` dameon which is installed by this playbook

## Requirements

Ansible

```sh
sudo apt install ansible
# or 
pip install ansible --user
```

SSH key authentication

```sh
ssh-copy-id -i {path_to_ssh_public_key} {user}@{host}
```
IAM user with following policy attached. 

- Allow programmatic access
- Access keys

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData",
                "logs:PutLogEvents",
                "logs:DescribeLogStreams",
                "logs:DescribeLogGroups",
                "logs:CreateLogStream",
                "logs:CreateLogGroup"
            ],
            "Resource": "*"
        }
    ]
}
```

Next, create a file in the root directory called `access_keys.json` and add IAM users access key values

```json
{
    "key_id": "xxxxxxxxxxxxx",
    "access_key": "xxxxxxxxxxx"
}
```

## Configuration

Variables are located in `playbook.yml` under `vars` section

| name  | description  | type | example  | notes |
|---|---|---|---|---|
| `temp_dir` | download directory | string | `/tmp`  |   |
| `cwagent` | agent download url by OS | dictionary  |   | use lowercase os names  |
| `run_as` | OS user agent will run as | string  | `cwagent`  | CloudWatch Agent installer creates a new `cwagent` user  |
| `profile_name`  | file based credentials profile name  | string  | `default` | use a IAM user with min rights  |
| `log_group` | CloudWatch log group name  | string  | `/on-prem/web-servers`  | Pre-create if IAM user policy has insufficient rights   |
| `monitor_files`  | file paths to stream to CloudWatch  | string list  | `/var/log/syslog`  | `cwagent` must have permissions to read  |
| `region`  | AWS region  | string  | `us-east-2`  |   |
|  `metrics_aggregation_interval` |   |  number | `60`  |   |
|  `metrics_collection_interval` |   | number  | `30`  |   |

## Running the Playbook

### Locally

```sh
ansible-playbook -i inventory_localhost.yml playbook.yml
```

### Remotely

Update the inventory file with remote hosts and groups then run

```sh
ansible-playbook -i inventory_remotehosts.yml playbook.yml
```

### Dry Runs

```sh
  # --check | don't make any changes; instead, try to predict some of the changes that may occur
  # --diff  | when changing (small) files and templates, show the differences in those files; works great with --check
  ansible-playbook -i inventory_localhost.yml playbook.yml --check --diff
```
