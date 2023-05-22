Title: shim-x64 package being downgraded

Jira ticket: <https://issues.redhat.com/browse/RHELC-899>

GitHub Pull Request: <https://github.com/oamg/convert2rhel/pull/833>

## Important notes

- ~~This problem happens only on UEFI machines.~~
	- Later in the investigation, we verified that it happens on UEFI and BIOS.
- It affects both RHEL 7 and RHEL 8

## Introduction

From my understanding of the problem related in that ticket, it seems that after we finish the conversion (which, apparently, doesn't error out anywhere) the package shim-x64 is in an older version than it is supposed to be.

It is strange that the package starts in one version and ends-up in a lower one after the conversion finishes.

For Oracle Linux 7.9 we have set this package to be in the [list for removal during the conversion](https://github.com/oamg/convert2rhel/blob/50216c1ee8ff6c4f1c7d3f165a206d0dee93bb81/convert2rhel/data/7/x86_64/configs/oracle-7-x86_64.cfg#L24). Not exactly sure on why we did that in the past, but it is know that the package caused problems during the transaction validation (both the old one and the new one).

Looking at the workflow that we go inside our transaction handler, it seems that the shim-x64 package is trying to:

1. Update the package (which doesn't error out)
2. Reinstall the package (Here it fails)
3. Then it downgrades the package to the lower version

I wonder if we actually need to try to reinstall the package even if we can update it.


### RHEL 7 (CentOS 7.9)

- Package version on a fresh machine: `shim-x64-15-8.el7.x86_64`.
- Package version after conversion: `shim-x64-15-7.el7_8.x86_64` and it seems to be the only one installed on the system. This version is lightly different from the one in the ticket as the date for this package in the rhel-7-server-rpms repos is `2023-04-18 07:16`.

### RHEL 8 (CentOS 8.5)

- Package version on a fresh machine: `shim-x64-15-15.el8_2.x86_64`
- Package version after conversion: `shim-x64-15-15.el8_2.x86_64` (reinstallation)

### Notes for 05/18/2023

After speaking with [Michal Bocek](https://github.com/bocekm), we confirmed that the correct behavior for the transaction validation/replacement is that we only should proceed on the reinstall of a package only if the package update will not work out, meaning that if a package is marked for update, then we don't need to do anything else.

After that conversation, I adjusted the code for the Yum transaction handler to apply that concept and it seems that not only the shim-x64 package is updated properly, but a lot of other packages that were suffering from the same condition. In the end, we were not only causing problems with that specific package, but with others as well, which were being reinstalled and downgraded.

I'm still waiting for some reply from the DNF team to make sure that I can do the same approach for Yum in DNF as well, as it will suffers from the same problem, but this time, not related to downgrade, but it is related to the reinstall process, as the DNF API is reinstalling the package with the same version.

### Notes for 05/19/2023

After more talking and investigation with the DNF team, we were able to come up with a simple solution that relies on querying the upgradable packages on the system, and then, checking if that package can be upgraded or not. If the package is not able to be upgradable, then we do down the route for reinstall/downgrade. This strategy is very similar to the one done in YUM, with the only difference that in YUM, we already have a sentinel value to check in the code, and for DNF, we need to query it first to check if the package can be upgaded.

## Conclusion

We had an incorrect behavior regarding the way we treat the packages that we pass down to the transaction validation/replacement. The operations we execute in this steps are the following:

1. Update the packages installed on the system
2. Reinstall the pakcages installed on the system
3. Downgrade the packages installed on the system

The last two operations should only be executed if the package can't be updated, for any reason. Think of it like a cascade, if the `1` operation "fails", then we go to the `2`, if that fails, then `3`.

## Todo List

1. [x] Check if the reproducer is valid
1. [x] Test if the `yum update shim-x64` really tries to update the package

It does seem that we are either reinstalling the package to an older version, or, dowgrading the package.

## Todo list for yum debugging

1. [x] Debug the path we go through the yum library to see if the package can't be updated (and the reason for that)