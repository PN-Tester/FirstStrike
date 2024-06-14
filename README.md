# FirstStrike - Exploit OS from UEFI 
This is a module I made to extend the capabilities of the [PCILeech framework]("https://github.com/ufrisk/pcileech"). The purpose is to enable penetration testers to exploit the windows OS in **situations where the Operating System is protected by DMA protection but the preboot environment is not**. FirstStrike will inject a modified Kernel Module into ntoskrnl.exe code caves and automatically execute its payload during boot. The payload is designed to detonate 30-45 seconds after arriving at the windows logon screen. It will pop a messagebox with the content "PN" to let the user know it has succeeded. The tool will automatically create a local user called "demo" with the password "P@ssw0rd123!" and place that user in the Local Administrators group.

# Rational
This module was create to supplement existing (and excellent) tooling provided by PCILeech. PCILeech contains a UEFI exploitation module out of the box (UEFI_EXIT_BOOT_SERVICES) which can be used to implant a kernel module (ntos_patch) when DMA protection is not enabled in the preboot enviroment. The problem is that the existing ntos_patch module requires subsequent PCI express interaction in order to exploit the OS. This leads to a problem in situations where the OS has DMA protection features enabled, because it is not possible to communicate with the kernel module implanted during UEFI stage after OS has boot without causing a bluescreen.
FirstStrike solves that problem by automating the exploitation steps that need to occur after the OS boots, while removing the requirement for subsequent communication with the implant. Thus, we can avoid triggering DMA protection countermeasures and fully compromise the Operating System simply via exploiting the unprotected UEFI environment. Obviously, preboot DMA protection features will prevent this attack.

# Usage
1. Either compile from source or move the precompiled .ksh file to your pcileech folder alongside the other .ksh files on your attack computer.
2. With the proper PCIexpress connection to the target laptop established (using PCILeech firmware and appropriate hardware), boot the target computer and enter UEFI
3. Again, obviously the target needs to have preboot DMA protection disabled for this to work.
4. Run ```PCILeech.exe -kmdload UEFI_EXIT_BOOT_SERVICES``` to locate EFI system table and hook the exit function
5. Exit UEFI on target computer, which will trigger your hook and jump to your UEFI kmd. Boot process on target computer will hang here. You should see success message from PCILeech with address of your KMD for example 0x38000000
6. Run ```PCILeech.exe FirstStrike -kmd <AddressOfKmd> -0 0x99``` replacing <AddressOfKmd> with the address you received in the previous step
7. You should see success message.
8. Run ```PCIleech.exe kmdexit -kmd <AddressOfKmd>``` replacing <addressOfKmd> with the address you received before
9. Target computer should now proceed with normal boot.
10. Approximately 30-45 seconds after arriving at windows logon screen, shellcode should detonate
11. You will see a messagebox containing the message "PN" appear if shellcode detonation is successful
12. You can close messagebox and login to target computer with newly created local administrator ```demo``` and password ```P@ssw0rd123!```

# Warning
There are many many things that can go wrong during this process, and the tool has not undergone robust testing against various make and models of computer and windows operating systems. The tool has been tested on Win10 and Win11, but only in a small sample size.
Because of the nature of shellcode placement in memory, its possible that various factors corrupt the injected shellcode and prevent successful detonation. This is ESPECIALLY TRUE IF YOU COMPILE THE KSH YOURSELF. I was having a lot of difficulty with corruption of the shellcode in memory and circumvented it by adding 
the NOP sled in front, which was a hacky fix. I pushed the corruption outside the range of usefull shellcode by doing this, but this doesnt prevent corruption from occuring. I will attempt to improve the reliability and stability of the injected payload when I have time.
Hopefully this tool works to enable testers to exploit a scenario via DMA attacks that is entirely exploitable by definition (vulnerable preboot but protected OS) but is not currently exploitable using the existing tooling.

# Shoutout
I also want to provide a shoutout to [Ulf Frisk]("https://github.com/ufrisk") for the fantastic [PCILeech framework]("https://github.com/ufrisk/pcileech") which enabled this project. I hope that this contribution can add a tiny bit of versatility to the wider community by enabling DMA attacks when preboot DMA is disabled but the OS is otherwise well protected. This scenario can occur in real life
and tester's should be able to exploit and provide PoC without too much overhead using this module.
