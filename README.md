# Dell WWAN sysID Bypass
Dell WWAN sysID bypass for WWAN driver setups.\
Use Dell WWAN cards from newer models with your older one (without their driver setup refusing the install).

Broadband (aka. WWAN) drivers are notoriously difficult to install on Windows without their proprietary setup files.\
These setups actually do more than just installing drivers for your WWAN card, they also actually activate them.

Activating them means dropping the appropriate driver but also installing a Windows NT service that communicates with your motherboard firmware via ACPI.\
What the proprietary Windows installer does is to install this Windows NT service for your card, but it also sends it a specific vendor command to make it detected.

Then when the proprietary WWAN service receives this command, it understands that it needs to tell the motherboard firmware to start announcing the 'arrival' of a new device, your WWAN card.
After it did it once, it saves a specific value in its registry configuration that tells the driver to automatically do it again at every Windows boot.

### What happens if you don't have the original Setup?
You just cannot use your WWAN card on Windows, that's it.\
You can drop all the WWAN drivers in the world from DriverPack Solution and yet you will still never see the WWAN Mini-PCI device in your device manager.

Installing the raw unpacked drivers just means pre-storing them incase you use this device in the future, it doesn't mean 'activate my WWAN card'.

### Why exactly we need a WWAN sysID bypass?
Because Dell is cucked and uses a driver authorization system known as 'WWAN sysID' in their WWAN drivers.\
They basically want to force people into only installing Dell WWAN cards that are whitelisted for their Dell laptop model.

And if you have a laptop from a different brand, even if it also has a SIM slot and broadband support you cannot use a Dell card.\
The Dell setup will refuse to install the driver and its necessary ACPI activation service.

You also cannot install a different Dell WWAN card than the approved one for your laptop model, even if it's actually a Dell laptop.

That's very much why we need a Dell sysID bypass, to bypass the Dell driver authorization system.

### How do I bypass the Dell sysID with my unapproved Dell laptop?
If you have a Dell laptop, then you can use Dell's own **driver_auth.exe** file with the provided password to generate an uncucked *dell_wwan_sysID.txt* file.\
Then edit the sysID file and whitelist your computer model inside, and re-generate a cucked version of the *dell_wwan_sysID.dat* file.

If you need the 'magic number' at the first column of the sysID file for your model, you can grab it from the WWAN sysID file of your own laptop model's approved WWAN setup file.

If you still can't figure out what your 'magic number' should be, or if you don't have a Dell brandband-enabled laptop, then see [Method 2](#i-dont-have-a-dell-laptop-but-it-also-has-a-sim-slot-how-do-i-bypass-the-sysid-check).

*Note: the older driver_auth.exe file seems to work better than the newer versions, and it's the same algorithm anyway.*

### I don't have a Dell laptop, but it also has a SIM slot. How do I bypass the sysID check?
Then you cannot generate a self-whitelisted version of the *dell_wwan_sysID.dat* file because your laptop isn't even a Dell one, even if you also have a motherboard with brandband support (and *SIM* slot, not a normal *smartcard* reader).

Instead you have to use Windows **IFEO** (*Image File Execution Options*) to hijack the execution of any file named **driver_auth.exe** to a fake program that just returns the exit code 0 and exits.

Example: use IFEO to hijack **driver_auth.exe** process names and redirect them to **nop-pe**:
- [https://github.com/myfreeer/nop](https://github.com/myfreeer/nop)

Myfreeer's *nop* program is a dummy EXE file that just does nothing and exits with exitcode 0.\
Dell's WWAN card setups actually rely on the exitcode as the result of its laptop model verification process.

You can easily configure IFEO with Diversenok's **ExecutionMaster**:
- [https://github.com/diversenok/ExecutionMaster](https://github.com/diversenok/ExecutionMaster)

Diversenok's *ExecutionMaster* program allows not only you to easily configure IFEOs without regedit, it also even has its own 'Deny and return error code' feature.\
Thanks to the way ExecutionMaster implements this feature, it also works on programs that verify the digital signature of the files they use.

So even if your driver_auth.exe was verified with digital signature validation as well, it would still be successfully verified by the Dell setup because *ExecutionMaster*'s 'Deny and return error code' is a special custom action that doesn't redirect execution to another file, it directly prevents execution and changes the return code in memory(?).

So whereas *nop-pe* sometimes failed to work due to signature verification, *ExecutionMaster* however has always been able to bypass the execution of unwanted programs without causing integrity verification errors in bloated Windows programs.
