**Thycotic Docs - dsv - 1.0.0**

**Authentication: Azure or AWS**

**Editing Your DevOps Secrets Vault Configuration**

After reviewing the information in this section, you may want to edit your DSV
configuration to make adjustments appropriate to your organization’s
infrastructure particulars.

Working on Linux or MacOS, you can use thy config edit --encoding yaml to open
your configuration in the OS’s default editor (typically **VI** or **nano**).
The editor directly updates the configuration in the vault when you save your
work.

On Windows, you must use thy config read -be YAML to read out the config; save
it as a file; edit locally; and use thy config update --path {path to file}
--data \\\@filename to upload your work into the vault, entirely overwriting the
prior config.

The initial config will look similar to this:

permissionDocument:

\- actions:

\- \<.\*\>

conditions: {}

description: Default Admin Policy

effect: allow

id: bl5hvib32rjc72ju81vg

meta: null

resources:

\- \<.\*\>

subjects:

\- users:\<thy-one:admin\@company.com\>

settings:

authentication:

\- ID: bl5hvib32rjc72ju8200

name: thy-one

properties:

baseUri: https://login.thycotic.com/

clientId: c139014a-147d-40c6-b6e2-b8e3e14af310

clientSecret: 22024095e13bc23586af1e758197c4f630b94c2b6ce6d5b729dee537f3206374

type: thycoticone

tenantName: company

You would modify the settings section using

thy config auth-provider create --name \<name\> --type \<type\> --{AWS or Azure
flag} \<AWS account ID or Azure tenant ID\>

-   name is the friendly name used in DSV to reference this policy

-   type is the authentication provider type; valid values are aws azure
    and thycoticone

-   properties are configuration settings specific to the authentication
    provider.

    -   The flag for AWS is --aws-account-id

    -   The Azure flag is --azure-tenant-id

The resulting addition to the config file thy config read -be yaml would look
like this:

permissionDocument:

\- actions:

\- \<.\*\>

conditions: {}

description: Default Admin Policy

effect: allow

id: bl5hvib32rjc72ju81vg

meta: null

resources:

\- \<.\*\>

subjects:

\- users:\<thy-one:admin\@company.com\>

settings:

authentication:

\- ID: bl5hvib32rjc72ju8200

name: thy-one

properties:

baseUri: https://login.thycotic.com/

clientId: c139014a-147d-40c6-b6e2-b8e3e14af310

clientSecret: 22024095e13bc23586af1e758197c4f630b94c2b6ce6d5b729dee537f3206374

type: thycoticone

\- ID: bm98r9asuqbc72ugmq00

name: azure-prod

properties:

tenantId: e2835781-b2e2-4613-98fd-d0d1eefd25ed

type: azure

\- ID: bm98paasuqbc72ugmpvg

name: aws-dev

properties:

accountId: "123457693"

type: aws

tenantName: company

**Authorizing a Security Principal**

After creating a provider account, you should authorize specific roles or users.

**AWS User Example**

When you create a user in AWS, remember that the username serves as a friendly
name within DSV, and does not have to match the Identity Access Management (IAM)
username—but the provider must match the provider name previously configured.

thy user create --username test-admin --external-id
arn:aws:iam::00000000000:user/test-admin --provider aws-dev

 

After creating the user, modify the config to give that user access to the
default administrator permission policy. For details on limiting access to
specific paths, see
the [Policy](https://prod.homer.thycotic.net/dsv/cli-ref/policy.md) section of
the [CLI Reference](https://prod.homer.thycotic.net/dsv/cli-ref/index.md).

thy config edit --encoding yaml

 

Add test-admin as a user subject to the **Default Admin Policy**. Third party
accounts must be prefixed with the provider name; in this case, the fully
qualified username will be **aws-dev:test-admin**.

permissionDocument:

\- actions:

\- \<.\*\>

conditions: {}

description: Default Admin Policy

effect: allow

id: bgn8gjei66jc7148d9i0

meta: null

resources:

\- \<.\*\>

subjects:

\- users:\<aws-dev:test-admin\| admin\@company.com\>

settings:

authentication:

\- ID: bgno6etchfrc72getij0

name: aws-dev

properties:

accountid: "00000000000"

type: aws

\- ID: bfdk8le2bmr1ok3b5u0g

name: azure-prod

properties:

tenantId: e2835781-b2e2-4613-98fd-d0d1eefd25ed

type: azure

tenantName: company

 

On a machine with the [AWS CLI](https://aws.amazon.com/cli/) installed and
configured with an AWS IAM user, download and initialize the thy CLI.

thy init --advanced

 

Choose the AWS IAM federated auth option when prompted.

Please enter auth type:

(1) Password (default)

(2) Client Credential

(3) AWS IAM (federated)

(4) Azure (federated)

 

When you select AWS IAM DSV will prompt for the specific AWS profile to use if
you are authenticating using a non-default AWS profile.

Please enter aws profile for federated aws auth (optional, default:default)

 

Read an existing secret to verify you are able to authenticate and access data.

thy secret read --path /servers/us-east/server01 -bf data.password

secretp\@ssword

 

**AWS Role Example**

This example assumes that you:

-   have your own thy CLI configured locally with an admin account

-   created an IAM role in the AWS Console

-   launched an EC2 instance using the IAM role

-   [downloaded](https://dsv.thycotic.com/downloads) the thy CLI onto the EC2
    instance

Create a corresponding role in DSV with the external-id of the IAM role's ARN.

thy role create --name test-role --external-id
arn:aws:iam::00000000000:role/testlogin --provider aws-dev

 

You should see a result similar to this:

{

"description": "",

"externalId": "arn:aws:iam::00000000000:role/testlogin",

"name": "test-role",

"provider": "aws-dev"

}

 

Add the role **aws-dev:test-role** to the **Default Admin Policy** in your vault
config to grant the new role admin access.

permissionDocument:

\- actions:

\- \<.\*\>

conditions: {}

description: Default Admin Policy

effect: allow

id: bgn8gjei66jc7148d9i0

meta: null

resources:

\- \<.\*\>

subjects:

\- users:\<aws-dev:test-admin\|admin\@company.com\>

\- roles:\<aws-dev:test-role\>

settings:

authentication:

\- ID: bgno6etchfrc72getij0

name: aws-dev

properties:

accountid: "00000000000"

type: aws

\- ID: bfdk8le2bmr1ok3b5u0g

name: azure-prod

properties:

tenantId: e2835781-b2e2-4613-98fd-d0d1eefd25ed

type: azure

tenantName: company

 

On the EC2 instance, configure the CLI by running thy init --advanced and
choosing AWS IAM as the authentication type.

Once configured, you can read an existing secret to verify the EC2 instance is
able able to authenticate and access data.

thy secret read --path /servers/us-east/server01 -bf data.password

secretp\@ssword

 

**Azure User Assigned MSI Example**

First you will need to configure the user that corresponds to an [Azure User
Assigned
MSI](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview).

The username is a friendly name within DSV and does not need to match the MSI
username. The provider must match the resource id of the MSI in azure.

thy user create --username test-api --provider azure-prod --external-id
/subscriptions/216d58f0-9fa1-49fa-b1f6-81e9f8a12f82/resourcegroups/build/providers/Microsoft.ManagedIdentity/userAssignedIdentities/test-api

 

Modify the config to give that user access to the default administrator
permission policy. For details on limiting access to specific paths see
the [Policy](https://prod.homer.thycotic.net/dsv/cli-ref/policy.md) section of
the [CLI Reference](https://prod.homer.thycotic.net/dsv/cli-ref/index.md).

thy config edit --encoding yaml

 

Add the user as a subject to the **Default Admin Policy**. Third party accounts
must be prefixed with the provider name, in this case the fully qualified
username will be azure-prod:test-api.

permissionDocument:

\- actions:

\- \<.\*\>

conditions: {}

description: Default Admin Policy

effect: allow

id: bgn8gjei66jc7148d9i0

meta: null

resources:

\- \<.\*\>

subjects:

\- users:\<azure-prod:test-api\|aws-dev:test-admin\|admin\@company.com\>

settings:

authentication:

\- ID: bgno6etchfrc72getij0

name: aws-dev

properties:

accountid: "00000000000"

type: aws

\- ID: bfdk8le2bmr1ok3b5u0g

name: azure-prod

properties:

tenantId: e2835781-b2e2-4613-98fd-d0d1eefd25ed

type: azure

tenantName: company

 

On a VM in Azure that is assigned the user MSI as the identity, download and
initialize the CLI. Choose the Azure federated authentication option.

thy init --advanced

 

Read an existing secret to verify you can authenticate and access data.

thy secret read --path /servers/us-east/server01 -b

 

**Azure Resource Group**

If you want to grant access to a set of VM’s in a resource group that use a
system assigned MSI rather than a user assigned MSI, you can create a role that
corresponds to the resource group’s resource ID.

thy role create --name identity-rg --provider azure-prod --external-id
/subscriptions/216d58f0-9fa1-49fa-b1f6-81e9f8a12f82/resourceGroups/build

 

Modify the config to give that role access to the default administrator
permission policy. For details on limiting access to specific paths see
the [Policy](https://prod.homer.thycotic.net/dsv/cli-ref/policy.md) section of
the [CLI Reference](https://prod.homer.thycotic.net/dsv/cli-ref/index.md).

thy config edit --encoding yaml

 

Add the user as a subject to the **Default Admin Policy**. Third party accounts
must be prefixed with the provider name; in this case the fully qualified role
name will be azure-prod:identity-rg.

permissionDocument:

\- actions:

\- \<.\*\>

conditions: {}

description: Default Admin Policy

effect: allow

id: bgn8gjei66jc7148d9i0

meta: null

resources:

\- \<.\*\>

subjects:

\- users:\<azure-prod:test-api\|aws-dev:test-admin\|admin\@company.com\>

\- roles:azure-prod:identity-rg

settings:

authentication:

\- ID: bgno6etchfrc72getij0

name: aws-dev

properties:

accountid: "00000000000"

type: aws

\- ID: bfdk8le2bmr1ok3b5u0g

name: azure-prod

properties:

tenantId: e2835781-b2e2-4613-98fd-d0d1eefd25ed

type: azure

tenantName: company

 

On a VM in Azure that is part of the resource group and has a system-assigned
MSI, run the init command. Choose the Azure auth option when prompted.

thy init --advanced

 

Read an existing secret to verify you are able to authenticate and access data.

thy secret read --path /servers/us-east/server01 -b

![](media/b5d5b24147a1d43c69c5cc1f2a5b7095.png)
