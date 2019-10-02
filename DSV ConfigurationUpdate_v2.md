**Configuration**

The configuration for DevOps Secrets Vault

-   defines the Default Admin Policy

-   contains settings for third-party authentication providers including
    Thycotic One, AWS or Azure.

**Commands that Act on Configurations**

| **Command** | **Action**                                                                                 |
|-------------|--------------------------------------------------------------------------------------------|
| read        | view the current configuration                                                             |
| edit        | modify the configuration in an OS-native text editor such as VI or nano (Linux shell only) |
| update      | upload a superseding configuration document                                                |

**Examples**

**Read**

To read out the current config, run:

thy config read -be yaml

In this command the b flag specifies to beautify the content while the e flag
specifies YAML (the other option being JSON).

In response, you should see a block of YAML code containing the Default Admin
Policy and the authentication configuration for Thycotic One, similar to this.

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

The initial user is setup with full administrator rights and is federated from
Thycotic One, the Thycotic identity provider. This allows self-service password
reset through Thycotic One. Best practices would be to setup a more restricted
user policy for common use and reserving usage of the initial Admin only when
necessary.

Additional users may also be setup in Thycotic One (at login.thycotic.com) and
then enabled is DevOps Secrets Vault using the command:

thy user create --username \<username\> --password \<password\> --provider
thy-one

**Edit**

The primary reason to edit or update the config file is to add a user to the
Default Admin Policy. It is not recommended that the Default Admin Policy be
changed otherwise. See the
[Policies](https://prod.homer.thycotic.net/dsv/1.0.0/permissions.md) article for
adding and editing policies beyond the Default Admin Policy.

Working on Linux or MacOS, use edit to open your configuration in the OS’s
default editor (typically **VI** or **nano**).

thy config edit --encoding YAML

The editor directly updates the configuration in the vault when you save your
work.

Note: On Windows, you cannot use edit on the configuration. Instead, you must:

-   use thy config read -be YAML to read out the config

-   save it as a file

-   edit the file locally

-   use thy config update --path {path to file} --data \\\@filename to upload
    your work into the vault, entirely overwriting the prior config.

**Update**

Use update to change a config by uploading JSON data.

The value of the --data parameter for update accepts JSON entered directly at
the command line, or the path to a JSON file.

thy config update --path us-east/server02 --data
{\\\\"something\\\\":\\\\"value\\\\"}

or

thy secret update --path us-east/server02 --data \\\@configfilename.json

**Adding Authentication providers**

It is not recommended that the Thycotic One provider be changed because that is
the provider for the initial user (and any other users the customer adds that
federate to Thycotic One).

**To add AWS or Azure as a provider:**

thy config auth-provider create --name \<name\> --type \<type\> --\<properties\>

-   name is the friendly name used in DSV to reference this policy

-   type is the authentication provider type; valid values are aws azure
    and thycoticone

-   properties are configuration settings specific to the authentication
    provider.

    -   The flag for AWS is --aws-account-id

    -   Azure flag is --azure-tenant-id

**Azure Example**

thy config auth-provider create --name azure-prod --type azure --azure-tenant-id
e2835781-b2e2-4613-98fd-d0d1eefd25ed

{

"created": "2019-09-30T23:20:06Z",

"createdBy": "users:thy-one:admin\@company.com",

"id": "bm98r9asuqbc72ugmq00",

"lastModified": "2019-09-30T23:20:06Z",

"lastModifiedBy": "users:thy-one:admin\@company.com",

"name": "azure-prod",

"properties": {

"tenantId": "e2835781-b2e2-4613-98fd-d0d1eefd25ed"

},

"type": "azure",

"version": "0"

Note that the account identifiers for third-party authentication are a top level
setting that allow you or other users to authorize specific security principals
within that account. They do not automatically grant access to any user or role
within the provider.

[See the Authentication: AWS and
Azure](https://docs.thycotic.com/dsv/1.0.0/authent-azure-aws) for examples of
using AWS and Azure for authentication.

**Policies**

DevOps Secrets Vault permissions are foundational proper operation and security.
Policies control access and authorization of Resources

To get a json encoded list of all policies

thy policy search

Add a query item to search the policies by path

thy policy search secrets/databases

thy policy search –query secrets/databases

A typical policy looks like this:

{

"created": "2019-09-24T18:12:26Z",

"createdBy": "users:thy-one:admin\@company.com",

"id": "0813d0bf-b7dc-4a45-a30c-d32596253049",

"lastModified": "2019-09-24T20:13:53Z",

"lastModifiedBy": "users:thy-one:admin\@company.com",

"path": "secrets:servers:us-west",

"permissionDocument": [

{

"actions": ["read"],

"conditions": {},

"description": "",

"effect": "allow",

"id": "bm57i0a7uj7s72peq1pg",

"meta": null,

"resources": ["secrets:servers:us-west:\<.\*\>"],

"subjects": ["groups:west admins"]

}

],

"version": "5"

}

The syntax supports wildcards via the \<.\*\> construct.

| **Element**        | **Definition**                                                                                                                                                                                                                                                                                                                                          |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| actions            | a list of possible actions on the resource including create, read, update, delete, list, share, assign. (Regex and list supported)                                                                                                                                                                                                                      |
| conditions         | an optional CIDR range to lock down access to a specific IP range                                                                                                                                                                                                                                                                                       |
| description        | human friendly description of the policy intent                                                                                                                                                                                                                                                                                                         |
| effect             | whether the policy is allowing or preventing access; valid values are allow and deny                                                                                                                                                                                                                                                                    |
| id                 | system-generated unique identifier to track changes to a particular policy                                                                                                                                                                                                                                                                              |
| resources subjects | the resource path defining the targets to which the permissions apply; a resource path prefixes the entity type (secrets, clients, roles, users, config, config:auth, config:policies, audit, system:log) to a colon delimited path to the resource. users or entities to which the policy enables authorization. Prefixes include users, roles, groups |

**Policy Evaluation**

To correctly evaluate permission policies, you must know the rules that apply to
permissions.

-   Permissions are cumulative.

>   If there is a top level permission for the path secrets:servers:\<.\*\> that
>   grants a user write access, then even if they are only granted read access
>   at the resource path secrets:servers:webservers:\<.\*\>, they will still
>   have write access due to the top level implicit match.

-   An explicit deny trumps an explicit or implicit allow.

-   Actions are explicit. A user assigned update and read will not automatically
    have create for the resource path.

-   The list action has a special behavior.

    -   First, list (search) is global—it runs across all items of an entity,
        not limited to paths and sub-paths.

    -   Second, to grant a user an ability to search entities via list, use the
        root of the entity if you want list to include other entities and
        actions within the same policy. The root entity, for example,
        is secrets, with no other characters following.

**Policy Examples**

**Deny Access at a Lower Level**

**Case:** Subjects need access to secrets for an environment, but that logical
environment contains a more restricted area.

**Solution:** Two policies. The first provides the Subjects
(developer1\@thycotic.com\|developer2\@thycotic.com ) general access to the
secrets to Resources at the path - secrets:servers:us-east-1:\<.\*\>

path: secrets:servers:us-east-1

permissionDocument:

\- id: erji23829823

description: Developer Policy.

subjects:

\- users:\<developer1\@thycotic.com\|developer2\@thycotic.com\>

actions:

\- "\<read\|delete\|create\|update\|share\>"

effect: allow

resources:

\- secrets:servers:us-east-1:\<.\*\>

The second policy adds an explicit deny with a more specific path to deny access
at the more privileged level, as in the following example.

path: secrets:servers:us-east-1

permissionDocument:

\- id: erji23829825

description: Developer Deny Policy.

subjects:

\- users:\<developer1\@thycotic.com\>

actions:

\- "\<.\*\>"

effect: deny

resources:

\- secrets:servers:us-east-1:production:\<.\*\>

**Allow Users to Assign Specific Roles**

**Case:** A user needs to assign roles when they create client credentials, but
must not be able to self-elevate by assigning an admin level role.

**Solution:** Use a naming convention when creating roles and specify a prefix
with a wildcard to only allow users to assign roles that match the naming
convention, as modeled in the following example.

\- id: erji23829828

description: Limited Role Assignment Policy.

subjects:

\- users:developer\@thycotic.com

\- roles:onboarding-role

actions:

\- "\<assign\>"

effect: allow

resources:

\- roles:dev-role-\<.\*\>

**Allow Users to List Specific Entities**

**Case:** A user needs to read and list entities within a policy.

**Solution:** Add a list action and the root of the entity used for searching.

In the example below, roles is the entity for reading and searching
(list action). In the resources section, roles:dev-role-\<.\*\> is used for
reading, while roles is used for searching.

\- id: erji23829828

description: Limited Role Policy.

subjects:

\- users:developer\@thycotic.com

\- roles:onboarding-role

actions:

\- "\<read\|list\>"

effect: allow

resources:

\- roles:dev-role-\<.\*\>

\- roles

The syntax of the latter is important. In general, the root form of an entity
has no \* after the entity name, or anything besides the name.

**Delegate Policy Authority**

**Case:** An admin wants to delegate control to various team leads at a
sub-path.

**Solution:** Under Resources, add config:policies followed by the resource path

\- id: erji23829828

description: Delegate sub-domains.

subjects:

\- - users:\<developer1\@thycotic.com\|developer2\@thycotic.com\>

actions:

\- "\<create\|read\|delete\|update\>"

effect: allow

resources:

\- secrets:servers:\<.\*\>

\- config:policies:secrets:servers:\<.\*\>

Now the developers can create policies below the secrets:servers: path, so for
example developer1 can create policies for secrets:servers:webservers and
developer2 can do the same at secrets:servers:databases
