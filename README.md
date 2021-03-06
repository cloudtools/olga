# Cloudsweeper

## Overview
Cloudsweeper was created because developers have better things to do than to constantly monitor their cloud accounts for things they left around. The tool monitors developer accounts for old items that might have been forgotten, and then sends reports to you or others to let them know what will be deleted (and to whitelist anything they do not want deleted).  It also can let you know when you haven't been tagging things properly.

Detailed instructions can be found in `Instructions.md`.

## Setup
To setup Cloudsweeper to work with your account, you must create a role in AWS to allow access. This is easily done using the `--setup` flag of Cloudsweeper. It will handle everything for you. Then all you need to do is to make sure you're in the list of accounts that will be checked. 

It's also possible to run `aws_setup.sh`, if you have the `aws` CLI installed and properly setup.

**IMPORTANT:** When running any of these setup methods, an environemnt variable named `CS_MASTER_ARN` must be set. This should be the AWS ARN of a role in a specific account that should have permission to assume into your account. This should match up with the role and account used to run Cloudsweeper, as Cloudsweeper will attempt to assume into your account from the role and account specified with this ARN. An example of valid input it:
```
arn:aws:iam::123456789123:user/cloudsweeper-master
```

## Usage
The program relies on having a list of accounts to actually check. This list can either be provided manually, or through other scripts.

The recommended way of using Cloudsweeper is through Docker. For the most common use cases, there are make targets (take a look in the `Makefile`).

## Modes
Below are the different modes that Cloudsweeper runs in.

### Review - `make review`
The review target will look for really old resources that Cloudsweeper is too unsure about to automatically cleanup. These resources are filtered based on some rules
The defaults are:

- Resource is older than 30 days
- A whitelisted resource is older than 6 months
- An instance marked with do-not-delete is older than a week

The account owner will get an email with these resources listed.

These thresholds may be modified to your own preference.

### Warning - `make warn`
The warning target will look for resources that are about to be automatically cleaned up by Cloudsweeper (not resources that the owner explicitly said should be deleted) and warn the owner about this.

### Marking - `make mark`
Marking will go through resources in the a users account and look for those that match a certain set of rules. If a resource matches, it will be marked for deletion. Deletion is set a few days in the future, so the user has time to whitelist anything that shouldn't be deleted. Resources are matched using the following rules:
- unattached volumes > 30 days old
- unused/unaccessed buckets > 120 days old
- non-whitelisted AMIs > 6 months
- non-whitelisted snapshots > 6 months
- non-whitelisted volumes > 6 months
- untagged resources > 30 days (this should take care of instances)

The resources will be marked with a tag with key `cloudsweeper-delete-at` and the value be a RFC3339 encoded timestamp.

### Finding resources - `RESOURCE_ID=<resource ID> make find`
Cloudsweeper can be used to find out more details about a specified resource in AWS. This is useful to quickly get some more details if all you have is a resource ID. If using the make target, the `RESOURCE_ID` variable must be set. If running the command directly, use the `--resource-id` flag.

### Cleanup - `make cleanup`
The cleanup target will look through resources and delete those that should be cleaned up. This is determined by looking at tags of the resources. 
There are certain thresholds that can be configured for this target. You can get more information on what those are by looking at the `--help` flag in the executable or by looking at the `config.conf` file
There are three requirements for this deletion:
#### Lifetime
A resource can have a lifetime. This is specified with the tag `Key: cloudsweeper-lifetime, Value: days-X`, where `X` is the number of days to keep the resource after its creation date. If the current date is after a resource's creation date + the lifetime it will get cleaned up.
#### Expiry
A resource can have an expiry date. This is specified with the tag `Key: cloudsweeper-expiry, Value: YYYY-MM-DD`, where `YYYY-MM-DD` e.g. `2018-01-29`. If the current date is after the expiry date, the resource will be cleaned up.
#### Delete at
If cloudsweeper has automatically marked a resource for deletion, it will have a tag with the key `cloudsweeper-delete-at`, and the value will be an RFC3339 encoded timestamp. If the current time is after that timestamp, the resource will get cleaned up.

## LICENSE
CloudSweeper is licensed under the BSD 2-clause licenses. Originally written
at Bracket Computing, it was made open source by VMware to enable further
development by the original authors.
