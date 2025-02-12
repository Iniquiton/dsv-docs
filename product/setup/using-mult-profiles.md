﻿[title]: # (Using Multiple Profiles)
[tags]: # (DevOps Secrets Vault,DSV,)
[priority]: # (1310)

# Using Multiple Profiles

On initial configuration, your DevOps Secrets Vault config will have just one profile with the choices you specified for credentials storage, authentication type, and cache strategy for secrets.

However, DSV supports creating other profiles, potentially with different credentials, and adding them to the config. Once the config has more than one profile, you can set which one DSV will use by default.

## Add a Profile to a Config

DSV syntax gives you two ways to add a profile to the config.

* Run `thy init` and type `add` or `a` at the prompt. Then enter the name of a new profile.

* To do it with one command, run `thy init --profile [name]` to add a profile with the default settings or (more likely) `thy init --advanced --profile [name]` to add a profile with the settings you choose.

## See the Config Contents

If you want to verify the profile has been added, output the updated config contents:

`thy cli-config read`

## Using an Alternate Profile for a Specific CLI Action

For a config with more than one profile, the profile used by default for any command will be the first profile created. However, you can override the default by specifying the profile to be used for a  command as a parameter:

`thy secret read --path mysecret --profile developer`

So commanded, the CLI will try to auth as the user specified in the `developer` profile and attempt to read the secret as that user.

The CLI does not have a command to set the default for all commands moving forward. For that, you should edit the `.thy.yml` file in the home directory to change the profile set as the default.

![Article End](../dsv-bug.png)

  
