# Setup Guide to get started with Hubo + ROS

### New Kernel Download and Install
1. Create a folder for a new linux kernel and (download & install) it:
  * ```mkdir -p ~/Downloads/kernel && cd ~/Downloads/kernel```
  * ```wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.4.43/linux-headers-4.4.43-040443_4.4.43-040443.201701150831_all.deb```
  * ```wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.4.43/linux-headers-4.4.43-040443-generic_4.4.43-040443.201701150831_amd64.deb```
  * ```wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.4.43/linux-image-4.4.43-040443-generic_4.4.43-040443.201701150831_amd64.deb```
  * ```sudo dpkg -i *.deb```
  * ```sudo update-grub```
  * ```sudo reboot```
2. Verify that correct kernel is loaded by comparing output of the following:
  * ```uname -a```
    * The above should output should include ```4.4.43-040443-generic```. If it does, continue on, otherwise reboot and select the 4.4.43 kernel during startup in the grub menu

### Kernel real-time patch with Xenomai
1. Install the following prerequisites:
  * ```sudo apt-get install devscripts debhelper dh-kpatches findutils kernel-package ncurses-base mtr-tiny ncurses-bin libncurses5-dev lib32ncurses5-dev libncursesw5-dbg libncursesw5-dev libncurses-dev fakeroot zlib1g-dev xterm```
2. Download the xenomai real-time framework:
  * ```mkdir -p ~/Downloads/xenomai && cd ~/Downloads/xenomai```
  * ```wget -O - http://xenomai.org/downloads/xenomai/stable/xenomai-3.0.3.tar.bz2 | tar -jxf -```
  * ```cd xenomai-3.0.3```
3. Create new debian changelog entry and (build & install) the packages (**Make Sure to fill in your email and name!!**):
  * ```DEBEMAIL="your@email" DEBFULLNAME="Your Name" debchange -v 3.0.3 Release 3.0.3```
  * ```sudo debuild -uc -us && cd ../```
  * ```sudo dpkg -i *.deb```
4. Download I-pipe patch and Kernel Source:
  * ```mkdir -p ~/Downloads/patch && cd ~/Downloads/patch```
  * ```wget http://xenomai.org/downloads/ipipe/v4.x/x86/ipipe-core-4.4.43-x86-6.patch```
  * ```wget https://github.com/kentsommer/hubo-setup/releases/download/0.1/linux-4.4.43.tar.xz```
    * **Note** this is not a completly vanilla 4.4.43 kernel as there is a compile time issue with the unisys visorbus included in the 4.4.43 source. So this linked kernel uses the latest (as of 2/28/17) release of the visorbus device drivers released by unisys. 
  * ```tar -xf linux-4.4.43.tar.xz && cd linux-4.4.43```
5. Patch and configure the kernel
  * ```~/Downloads/xenomai/xenomai-3.0.3/scripts/prepare-kernel.sh --linux=. --adeos=../ipipe-core-4.4.43-x86-6.patch```
  * ```make menuconfig```
    * Unselect the following (Press N over option):
      * ```Power management and ACPI options → CPU Frequency scaling → [ ] CPU Frequency scaling```
      * ```Power management and ACPI options → [ ] Cpuidle Driver for Intel Processors```
      * ```Power management and ACPI options → [ ] Power Management Debug Support```
      * ```Power management and ACPI options → ACPI → < > Processor```
      * ```Power management and ACPI options → CPU Idle → [ ] CPU idle PM support```
      * ```Device Drivers → Input device support → Miscellaneous devices → < > PC Speaker support```
    * Save changes (leave name and everything as default!).
    * Exit after finished saving. 
  * ```sudo CONCURRENCY_LEVEL=8 CLEAN_SOURCE=no fakeroot make-kpkg --initrd --append-to-version -xenomai-3.0.3 --revision 1.0 kernel_image kernel_headers```
  * ```cd ~/Downloads/patch```
  * ```sudo dpkg -i *.deb```
  * ```sudo update-grub```
  * ```sudo reboot```
  
### Setup permissions and user groups
1. Create and add yourself to xenomai user group:
  * ```sudo groupadd xenomai```
    * Group may already exist so ignore error if that is the case
  * ```sudo usermod -aG xenomai your-username```
    * Example: ```usermod -aG xenomai kent```
2. Download grub customizer
  * ```sudo add-apt-repository ppa:danielrichter2007/grub-customizer```
  * ```sudo apt-get update```
  * ```sudo apt-get install grub-customizer```
3. Setup xenomai kernel parameters
  * Run the above installed Grub Customizer and find the ```Linux 4.4.43-xenomai-3.0.3``` entry.
  * Right click on that entry and select edit.
  * Find the line beginning with "linux" (should be near the bottom)
  * Add ```xenomai.allowed_group=<gid of xenomai group>``` to the end of the linux line
    * Can find gid of xenomai group by running ```id your-username``` in a terminal. The number next to xenomai is the gid
    * Example:
      * id kent output shows ```132(xenomai)```
      * full entry in grub entry would then be: ```linux	/boot/vmlinuz-4.4.43-xenomai-3.0.3 root=UUID=f9db850a-4660-4b15-8d64-074a92f6bfd6 ro  quiet splash $vt_handoff xenomai.allowed_group=132```
  * Select OK
  * Select Save 
  * Quit the Grub Customizer application
4. Reboot and test xenomai permissions
  * ```sudo reboot```
  * After reboot run the following:
    * ```/usr/xenomai/bin/latency```
      * If working you should start seeing latency test ouput
      
### Setup hubo for ROS and PODO
1. Run script to do fully automatic setup for both PODO and hubo for ROS
  * ```cd ~/Downloads && wget https://github.com/kentsommer/hubo-setup/releases/download/0.1/setup.sh && sudo chmod +x setup.sh && ./setup.sh```
    

  
  

  
  
  
  
  
  

 
  
