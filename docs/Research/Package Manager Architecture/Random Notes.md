## Notes from 05/22/2023

We probably can move away a few functions from pkghandler.py to live inside specific modules after this refactor. 

For example, the [get_system_packages_for_replacement](https://github.com/r0x0d/convert2rhel/blob/50216c1ee8ff6c4f1c7d3f165a206d0dee93bb81/convert2rhel/pkghandler.py), is a function that is used only for [[DNF]] and [[YUM]] in transaction handler mode. 

Probably would be better to start making a list of things that can be moved to the new module and removed from pkghandler.

------------------------------------------------------------------------
Probably it would be better to think in a way that minimize all of this changes to the tool, otherwise, it would take super long to make this viable. 

We could use the Tech Preview feature to start introducing changes in a first release, just to get the hang of it, probably behind a flag so the user has to explicitly ask for it. 

If we decide that releasing this as a Tech Preview is the easiest way for now, I wonder if we could maintain the same code running as we have right now, but only execute the new one if the flag is used.  Of course, changes would be needed to do this, but they would "be minimal", for now... 

Taking as an example the transaction handler, that would be a very comfortable way of introducing this first piece of change, as we don't have any return value for this operation and we could very easily toggle between on and off for that feature.

Other places on the system that would benefit from that refactor still needs to be investigated and select, but for now, transaction handler is a big candidate.

------------------------------------------------------------------------
