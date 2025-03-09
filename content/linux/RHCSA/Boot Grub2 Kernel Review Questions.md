## Boot Grub2 Kernel Review Questions

Q. UEFI is replacing BIOS in newer computers. True or False?
A. True.

---

Q. You have changed the timeout value in the grub configuration file located in the /etc/default/ directory. Which command would you run now to ensure the change takes effect on next system reboots?
A. `grub2-mkconfig` 

---

Q. You want to view the parameters passed to the kernel at boot time. Which virtual file would you look at?
A.  /proc/cmdline

---

Q. Name the location of the grub.efi file in the UEFI-based systems.
A. /boot/efi/EFI/redhat/

---

Q. What is the name of the default bootloader program in RHEL 9?
A. GRUB2.

---

Q. The systemd command may be used to rebuild a new kernel. True or False?
A. False.

---

Q. What does the `chroot` command do?
A. Changes the specified directory path to /.

---

Q. At what stage should you interrupt the boot sequence to boot the system into a non-default target?
A. At the GRUB2 stage.

---

Q. Which two files would you view to obtain processor and memory information?
A. The `cpuinfo` and `meminfo` files in /proc/

---

Q. By default, GRUB2 is stored in the MBR on a BIOS-based system. True or False?
A. True.

---

Q. Which file stores the location of the boot partition on the BIOS systems?
A. The *grub.cfg* file stores the location information of the boot partition.

---

Q. You have lost the root user password and you need to reset it. What would you add to the default kernel boot string to break the boot process at an early stage?
A. You would add `rd.break` to the kernel boot string.

---

Q. You have installed a newer version of the kernel. What would you now have to do to make the new kernel the default boot kernel?
A. Nothing. The install process takes care of that.

---

Q. What is the system initialization and service management scheme called?
A. It is called *systemd*.
