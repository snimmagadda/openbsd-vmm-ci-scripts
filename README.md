With the following instructions, an OpenBSD virtual machine (guest)
could be bootstrapped (unattended, autoinstalled) on an OpenBSD
host using vmd(8).

To autoinstall(8) OpenBSD vm unattended, it needs to be netbooted
following which the installer picks up an answer file via HTTP where
the url is a combination of DHCP option `server-name` and the answer
file name; for example: `http://100.64.1.2/install.conf.  further
details: https://man.openbsd.org/autoinstall.8.  With '-L' option,
vmctl(8) configures and runs a simple DHCP server for the vm.

First, the host configurations...

1. Enable vmd(8) with...

	rcctl enable vmd && rcctl start vmd

2. A simple config file for httpd(8)...

	doas sh -c 'cat << EOF > /etc/httpd.conf
	server "default" {
		listen on lo0 port 80
	}
	EOF'

3. Enable httpd(8) with...

	rcctl enable httpd && rcctl start httpd

4. An answer file as install.conf is required in /var/www/htdocs,
   refer to Examples in autoinstall(8).
5. Configure host's firewall to NAT virtual machine traffic with...

	pass out on egress from 100.64.0.0/10 to any nat-to egress

   appended to /etc/pf.conf

Create a disk for virtual machine... 

	vmctl create -s 4.5G disk.qcow2

Start a virtual machine netbooted with the host's ramdisk kernel..

	vmctl start -Bnet -b/bsd.rd -L -m1G -ddisk.qcow2 "vmm1"

This should autoinstall OpenBSD after initial 5 second timeout at
boot and pick options given in install.conf answer file.  Append
`-c` option to actually see what's happening on the console.

Once the installation completes, the installer reboots the system.

Boot and login to the new virtual machine and attach a console with...

	vmctl start -L -m1G -ddisk.qcow2 -c "vmm1"

