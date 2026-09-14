# patching-rhel-demo
This repo includes ansible playbooks for a demo project of automating periodic patching process for RHEL with Red Hat Ansible Automation Platform.

## Automating patcing process for RHEL with Red Hat Ansible Automation Platform
The goal behind the code is to demonstrate a simple example for automating patcing process with Red Hat Asible Automation Platform as an automation orchestrator.

## Ansible Automation Platform environment
The assumed environment can be set up on AWS EC2 easily by using playbooks and roles in the [setup](./setup) folder. Please refer to [SETUP.md](./setup/SETUP.md) for more details.


## Included contents
### Playbooks
#### Environment
|Name     |Description|
|:--------|:----------|
|`create_jobtemplate.yml`|JobTemplateを作成するPlaybook|
|`create_workflow.yml`|Workflowを作成するPlaybook|
|`create_workflow2.yml`|Workflowを作成するPlaybook|
|`create_workflow3.yml`|Workflowを作成するPlaybook|
|`create_managed_vms.yml`|パッチ適用対象のVMを作成するPlaybook|
|`delete_managed_vms.yml`|パッチ適用対象のVMを削除するPlaybook|
|`create_demo_portal.yml`|デモ用ポータル画面をVM構築〜アプリ設定まで実施するPlaybook|

#### Scan
|Name     |Description|
|:--------|:----------|
|`scan_advisory.yml`|パッチ適用対象のVMの適用可能なAdvisoryをスキャンするPlaybook|
|`send_slack_scan.yml`|スキャン結果をSlackに通知するPlaybook|

#### Patch
|Name     |Description|
|:--------|:----------|
|`publish_advisory.yml`|Push the advisory list to a Git Repo.|
|`send_slack_launch.yml`|Send Slack for Approval.|
|`backup_vm.yml`|Snapshot the disk of runnning managed VMs.|
|`apply_errata.yml`|Apply all security advisories in the advisory list.|
|`reboot_vm.yml`|Reboot the running managed VMs.|
|`test_vm.yml`|Test the infrastructure of the resbooted VMs.|
|`test_app.yml`|Test the application of the rebooted VMs.|
|`* scan_advisory.yml`|パッチ適用対象のVMの適用可能なAdvisoryをスキャンするPlaybook|
|`send_slack_report.yml`|Send slack for finish report.|

### Workflows
|Name     |Description|
|:--------|:----------|
|`Delete and Create VMs WF`|パッチ適用対象のVMを削除し、新しいVMを作成するWorkflow|
|`Scan WF`|パッチ適用対象のVMのスキャンを実施して、Slack通知するWorkflow|
|`Periodic Security Patching WF`|デモポータルからの依頼を受け取って、パッチ適用の一連の流れを実施するWorkflow|
|`Advisory List Publisher WF`|`Periodic Security Patching WF`の一部で、デモポータルからの依頼を受け取って、GitHubに`remediation_dataset`を登録するWorkflow|
|`Apply Patch WF`|`Periodic Security Patching WF`の一部で、`remediation_dataset`のパッチ適用内容を元にパッチ適用の一連の流れを実施するWorkflow|


### Group variables
These variables have already been set as follows. You can adjust them based on your environment.
```
---
purpose: patch_demo

aws_region: ap-northeast-1 # adjust with your preference
wp_weblog_title: "DemoSite"

# Location of remediation_dataset downloaded.
remediation_dataset_file: "{{ playbook_dir }}/datasets/remediation_dataset.yml"

remediation_group: canonical_remediation_targets
```

## Prerequisite

### Prepare a deploy key to your GitHub repo
You need to set up a deploy key to gives workflows running on AAP safe access to just one single repository. 

Generate a dedicated key pair:
```
$ ssh-keygen -t ed25519 \
    -C "aap-publish-dataset" \
    -f aap_publish_dataset
```

Please refer to [Deploy keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/managing-deploy-keys#deploy-keys) for more details.

## Installation and usage
Assuming the demo environement has been already created by the way of before mentioned and you've also performed `git clone` this repo. Ensure that you are logged in to your Ansible Automation Controller before proceeding with following steps.

### Create credentials
At leaset the following four credentails need to be defined. 

#### Credential for AWS
1. Click `Credentials` in the left menu.
2. Click `Create credential` button.
3. Enter the following fields:
   - Name: `aws_cred`
   - Organization: `Default`
   - Credential Type: `Amazon Web Services`
   - Access Key: your AWS_ACCESS_KEY_ID
   - Secret Key: your AWS_SECRET_ACCESS_KEY
4. Click `Create credential` button.

#### Credential for ssh to AWS instances
1. Click `Credentials` in the left menu.
2. Click `Create credential` button.
3. Enter the following fields:
   - Name: `aws_key`
   - Organization: `Default`
   - Credential Type: `Machine`
   - SSH Private Key: your AWS private key
   - Username: `ec2-user`
4. Click `Create credential` button.

#### Credential for pusing to Git repo
1. Click `Credentials` in the left menu.
2. Click `Create credential` button.
3. Enter the following fields:
   - Name: `GitHub Push Credential`
   - Organization: `Default`
   - Credential Type: `Machine`
   - Git SSH Private Key: your Git private key
4. Click `Create credential` button.


### Create inventories
1. Click `Inventories` in the left menu.
2. Click `Create inventory` button and select `Create inventory`.
3. Enter the following fields:
   - Name: `Patch_Demo`
   - Organization: `Default`
4. Click `Create inventory` button and then select `Sources` tab.
5. Click `Create source` button.
6. Enter the following fields:
   - Name: `Patch_Demo_Source`
   - Source: `Amazon EC2`
   - Credential: `aws_cred`
   - Update options: `Overwrite`, `Overwrite variables`
   - Source variables:
   ```
   ---
   # Minimal example using environment variables
   # Fetch all hosts in ap-northeast
   plugin: amazon.aws.aws_ec2
   keyed_groups:
   - prefix: tag
      key: tags

   # Change regions corresponding to your environment
   regions:
   - ap-northeast-1 # adjust with your preference

   # Filter only objects taged with "purpose" tag as "patch_demo"
   filters:
   tag:purpose: patch_demo

   hostnames:
   - private-dns-name

   compose:
   ansible_host: public_dns_name

   # Ignores 403 errors rather than failing
   strict_permissions: false
   ```
7. Click `Create source` button.


### Create a project
1. Click `Projects` in the left menu.
2. Click `Add` button.
3. Enter the following fields:
   - Name: `Patch_RHEL_Demo`
   - Organization: `Default` (or your prefered organization)
   - Execution Environment: `Default execution environment`
   - Source Control Type: `Git`
   - Source Control URL: your Git repositoriy
   - Options: `Clean`
4. Click `Create project` button

Please refer to [Ansible Doc](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.7/develop-proc_controller_adding_a_project) for more details.

### Create a AAP API-Token
1. Click User > admin > API Tokens
2. Click `Create API token` button.
3. Enter the following fields:
   - token description: `Patch Demo Integration`
   - Scope: `Write`
4. Click `Create token` button
5. Note `API token`.

### Create job templates
1. Click `Templates` in the left menu.
2. Click `Create template` button and select `Create job template`.
3. Follow the next steps respectively.
4. Click `Create job template` button

- Create JobTemplate: create_jobtemplate.yml
Playbook内に記載している変数をExtra_varsに設定して実行するとJobTemplateが自動生成されます。
- Create Workflow: create_workflow.yml
Playbook内に記載している変数をExtra_varsに設定して実行するとWorkflowが自動生成されます。
