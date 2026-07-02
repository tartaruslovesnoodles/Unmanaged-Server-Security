# Unmanned Server Security - Basic Operational Guide   
## (context): This repository is about unmanaged server (mostly headless linux deployments) setup/security and best practices. This markdown file is a simple prototype for conveying this information, utilize sub folders for automated versions of portions of the instructions listed here as they come. You'll notice this document contains many vague suggestions. Simply because I want to keep this relevant/helpful even as technology and companies shift or mutate over time. This document is designed to be easily researchable (i.e. the newest distros of certain OSs) to counteract any errors caused by ambiguity.

Unmanaged server within this document's context refers to servers you purchase that come with largely unmodified and default permissions, with you (end user) getting full root access. That is to say the ISP doesn't manage it or run the service for you.    

The expected threat model throughout this is a medium/typical to high threat environment. It addresses opportunistic attacks (automated scanning, known vuln attempts on a target list, any other "blind" attack) and well-resourced adversaries/targeted attacks (targeting your infrastructure intentionally).     

This guide does not claim absolute protection against groups with near unlimited resources, lawful coercive/control powers, or zero-days for every app on Earth. It focuses on raising cost, reducing attack vectors, and sectioning off damage while also providing some reactive actions once something has happened.
                                

0 - Basic Checklist/Golden Path aka tl;dr (re-read this whenever you set up new servers if you follow this guide).  


 A. Confirm out-of-band access.  
 B. Test rollback/snapshot if present (highly recommended for most applications).  
  * Can represent a security risk if it isn't encrypted and secure. Some redundant data plane servers may be better off without snapshots if it's on rented hardware.

 C. Store your backups outside the renting ISP (if applicable) or off-host.  
 D. Install a minimal and supported Linux OS (check out section 2 on OS, it is short), headless if you only plan to use the cli.  
 E. Update the system immediately upon deployment.  
 F. Remove and disable all non-essential services.  
 G. Secure physical access (if applicable).  
 H. Encrypt disk (unless renting, but still consider it even then).  
 I. Create a sudo user, disable root, change ssh port, swap to ssh keys and off of passwords, whitelist ssh port and move ssh to an IP with no existing service or app (ie no rdns, no web server, dns resolver, etc). Check out section 3, SSH setup, for more in-depth SSH security and SSH hardening.  
 J. Apply your firewall. Firewall doctrine should be to whitelist needed services and drop the rest (mirror this upstream if your host has upstream firewall customization).  
 K. Setup Apparmor/SELinux and build around it without making convenience changes.   
 L. Rotate logs and do remote log shipping if possible.   
 M. Enable automatic security updates.  
 N. Monitor auth, disk, cpu, ram, and nic saturation.  


1 - Network/Env Concerns
- If the server is rented in a different country and or region operating under different digital technology laws, ensure you're privy to the difference and know your rights.
- Confirm third party 'broad-dependencies' (ie a payment merchant like stripe) are secure or you have a failsafe for each of their specific security failures.   
- Confirm out of band access (kvm, rescue boot, console, etc).   
- Confirm a rollback path (i.e. snapshot/backup and test a restore) and secure offsite storage.            
 A. Recovery is primarily for the control plane or the server that holds referenced files and communicates through the network. For data plane or servers that primarily fetch and transmit data, snapshots aren't needed and may be a security detriment in some cases.    
- Find out how you receive alerts for server issues and who sends them.   
- Decide where "secrets" will live at this stage, do NOT budge for convenience. Find work arounds and only disregard if a security risk presents itself.   
 A. Never store on repos.   
 B. Replace them on any leak suspicion.   
 C. Separate production, staging, and dev secrets. 


 Depending on if you are renting a server in a datacenter, placing this server into an existing datacenter, are running local (WAN) hardware, or are running local (LAN) hardware then the security prerequisites are different.     

 Rented Servers      
  A. Decide on what you value in a server before browsing ISPs: CPU, parallel processing(GPU), port speed, RAM, DoS protection, customizability, scalability, upstream capabilities, stateless vs stateful network, null route policy, datacenter backbone, etc.  
  B. Look at the 'top 20' hosts (do NOT rely on "top #n Dedicated Server Hosts" articles, these are often sponsored, look around and talk to devs instead). As of Jan 2026 (take w/ more salt the more years pass), a few solid choices are Datapacket, OVH, Hetzner, VOX, and IBM).  
  C. Know your host -- Research them extensively for limits and perks ( from A).   
  D. Dedicated servers are ideal for security, using a VM (often called VPS/VDS) from rented hosts opens up vulnerabilits you can't control/patch, for instance VM escapes.   
  E. Utilize the host's upstream firewall if present to filter sensitive ports ( ssh) before packets reach your server, this can block port scans (soley dropping/rejecting locally to a port still gets scanned).  
  F. Investigate privacy polics, data sales, and EULA carefully.  
  G. If you have a network composed of servers from multiple hosts, consider using different emails and account credentials/verification methods to minimize spread against panel-access compromise attacks.  
  H. An encrypted disk is an option for rented servers but I wouldn't recommend it despite its potential security benefits. It is near mandatory with minimal downside for other types of deployment (not rented).  
  * Theft is a possibility with rented servers, you have very little control over the security level and who can access your server in a datacenter you don't own: If the sec of the ISP looks bad, encrypt  (…and maybe go to a different ISP).                                         

Inputting Into datacenter  
  A. Encrypt your disk.  
  B. Ensure employees verify who are performing reboots, taking console cables, drive swaps, etc. Keep this logged and backed up. It is a social engineer's paradise without this.  
  C. Draft and document the chain of custody and your hardware handling policy.  
  D. Restrict console access.   
  E. Confirm other tenants can't arp-poison/sniff anything.  
  F. If you have an upstream bandwidth provider, ensure they offer: RTBH, fast response team, and a clear scrubbing/null route policy.  
  G. Disable BMC if you don't trust it.  
  H. Hardware backed ssh.  
   Security key, TPM-backed SSH, or Secure Enclave. A key/smartcard, like from Yubikey or FIDO2 would be the best choice in my opinion.   
Running Local(LAN)  
  A. Encrypt your disk.  
  B. Secure the physical location of your server --  in a locked cabinet, in a locked room, in a private locked building  
  C. Create a VLAN for servers to partition/quarantine the network.  
  D. Create internal firewall rules even if it is LAN, another compromised device on the same LAN opens the door to 10+ different attack vectors to attempt on your server.  
  E. Ensure safe shutdown in the event of power loss.  
  F. A generator or some way to ensure a failover for electricity in sudden outages is ideal.  
  G. Hardware backed ssh.  
   - Security key, TPM-backed SSH, or Secure Enclave. A key/smartcard, like from Yubikey or FIDO2 would be the best choice in my opinion.  
   
Running Locally(WAN)    
  A. Encrypt your disk.  
  B. Drop UPnP.  
  C. Your DDoS protection is only as good as your hardware, if this is a concern consider buying upstream bandwidth. If not just do some simple rules on a "whitelist and drop everything else" policy and some load balancing.  
  D. Decide how admin access will work, don't leave SSH open to the entire internet for example.  
  * IP whitelisting, proxy ssh, permission distribution, etc.              
               
  E. Split DNS and don't leak internal names.  
  F. Router hardening is most relevant for this type of server deployment.  
  G. Without purchasing an IP block from ARIN/RIPE your reputation will likely be variable   
  H. Use reverse proxs or edge relays when possible to hide your IP.  
  I. WAF/CDN if applicable.  
  J. Hardware backed ssh.  
   - Security key, TPM-backed SSH, or Secure Enclave. A sec key/smartcard, like from Yubikey or FIDO2 would be the best choice in my opinion.

-more coming soon  



 

2 - Operating System  

 Pick an operating system that is trusted, high support with your hardware arch, new (version), and secure:  
  A. Ubuntu  
  B. Debian  
  C. RHEL  
 - There are more solid choices than these, but these are the least debated and least polarized choices. It is a common consensus they're the most reliable and consistent operating systems for headless servers across architectures.  
 - You may notice I did NOT include the versions that are currently new and most supported, that is because that will age like milk. Simply check the OSs on a search browser and find out the newest stable/supported release.  
 A. May also want to mark EoLs of your chosen OS versions if you plan to run a server for multiple years.    
 B.  LTS may be ideal if you intend to "run and forget" a server for a long time (1 year+).  
 C.  Make sure to minimize your install to base unless explicitly needed, this adds unnecessary attack surface area.           
  * IF srvc =! required to run server's purpose: shouldn't be installed.   

- https://messervices.cyber.gouv.fr/documents-guides/linux_configuration-en-v2.pdf is a dense and hollistic source for linux OSes and ideal configuration from 2022, if you want to further your OS knowledge.

 - Use principle of least privilege (polp)   
A. polp states every user/system/service gets the minimum access needed to complete the job.    
B. Many portions of polp require a superuser at first to set things up, then shifts into a usr/srvc with less permissions. For example in a database you may start with postgres as the user but then your admin account is restricted to basic maintenance, and an even lower permission account for the database API account/role.                                                    
C. In a way polp is common sense, but many people will get mad at a finicky program and chmod 777 it. But that is exactly how you can easily break the basic principle and put yourself at risk.   

 - 'Defense in depth' principle states you should have multiple security controls across the entire system. So instead of having just a firewall or OS hardening, you have both of those, input validation on application layer, auth, encryption, MFA, monitoring, logging, security policies, intrusion detection, segmentation, proxies, etc.   
 - Bootloaders must be secured if you don't have a rented/cloud provider or a specific OS 'doing it for you'.   
A. Bootloaders are services/programs that run at the launch of a machine in a sort of 'partitioned sandbox'. They contain (or can have added to them) file system access control, boot options (ie external boot), bypasses of your security points, etc.   
B. Activation of IOMMU on the bootloader protects against arbitrary memory access.        
C. ```[...]=full,force``` can be used to enable all mitigations and lock them in place: for example l1tf=full,force for Intel's L1 Terminal Fault Vuln.   
D. Add these to your bootloader. They fill free pages until use, force PTI, increase difficulty in heap overflow cyberattacks, and block specific attack patterns respectively.    
```
page_poison=on
pti = on
slab_nomerge=yes
slub_debug=FZP
spectre_v2=on
mds=full,nosmt
mce=0
page_alloc.shuffle=1
rng_core.default_quality=500
```
E. General settings to add to your bootloader (ANSSI Guidelines):    
```
# Restrict access to the dmesg buffer (equivalent to
# CONFIG_SECURITY_DMESG_RESTRICT=y)
kernel.dmesg_restrict=1
# Hide kernel addresses in /proc and various other interfaces ,
# including from privileged users
kernel.kptr_restrict=2
# Explicitly specify the process id space supported by the kernel ,
# 65536 being an example value
kernel.pid_max=65536
# Restrict the use of the perf subsystem
kernel.perf_cpu_time_max_percent=1
kernel.perf_event_max_sample_rate=1
# Prohibit unprivileged access to the perf_event_open () system call.
# With a value greater than 2, we impose the possession of
# CAP_SYS_ADMIN , in order to collect the perf events.
kernel.perf_event_paranoid=2
# Activate ASLR
kernel.randomize_va_space=2
# Disable Magic System Request Key combinations
kernel.sysrq=0
# Restrict kernel BPF usage to privileged users
kernel.unprivileged_bpf_disabled=1
# Completely shut down the system if the Linux kernel behaves
# unexpectedly
kernel.panic_on_oops=1
# This option replaced CONFIG_DEBUG_RODATA in the kernel (> = 4.11)
# which was a dependency of CONFIG_DEBUG_KERNEL
CONFIG_STRICT_KERNEL_RWX=y
# CONFIG_ARCH_OPTIONAL_KERNEL_RWX and
# CONFIG_ARCH_HAS_STRICT_KERNEL_RWX are dependencies of
# CONFIG_STRICT_KERNEL_RWX
CONFIG_ARCH_OPTIONAL_KERNEL_RWX=y
CONFIG_ARCH_HAS_STRICT_KERNEL_RWX=y
# Check and report unsafe memory mapping permissions , for
# example , when kernel pages are writable and executable.
# This option is not available on all architectures
# hardware (x86> = 4.4, arm64 > = 4.10, etc.).
CONFIG_DEBUG_WX=y
# Disable the file system used only for debugging. Protecting this file
# system takes additional work.
CONFIG_DEBUG_FS=n
# Starting with kernel version 4.18, these options add
# -fstack -protector -strong at compilation time to strengthen
# the canary stack , it is necessary to have a version of GCC at least
# equal to 4.9.
# Before version 4.18 of the linux kernel , you must use the options
# CONFIG_CC_STACKPROTECTOR and
# CONFIG_CC_STACKPROTECTOR_STRONG
CONFIG_STACKPROTECTOR=y
CONFIG_STACKPROTECTOR_STRONG=y
# Prohibits direct access to physical memory.
# If necessary and only in this case, it is possible to activate a
# strict access to memory , thus limiting its access , with options
# CONFIG_STRICT_DEVMEM=y an
# Detects stack corruption during the call to the scheduler
CONFIG_SCHED_STACK_END_CHECK=y
# Impose a check of the limits of the memory mapping of the process
# at the time of data copies.
CONFIG_HARDENED_USERCOPY=y
# Forbid the return to a verification of the mapping with the allocator if
# the previous option failed.
#CONFIG_HARDENED_USERCOPY_FALLBACK is not set
# Added cover pages between kernel stacks. This protects against the effects
# of edge of stack overflows (this option is not supported on all
# architectures).
CONFIG_VMAP_STACK=y
# Impose exhaustive checks on kernel reference counters (<=5.4)
CONFIG_REFCOUNT_FULL=y
# Check the memory copy actions that could cause corruption
# of structure in the kernel functions str*() and mem*(). This verification is
# performed at compile time and at runtime.
CONFIG_FORTIFY_SOURCE=y
# Disable the dangerous use of ACPI, which can lead to direct writing
# in physical memory.
#CONFIG_ACPI_CUSTOM_METHOD is not set
# Prohibit direct access to kernel memory from userspace (<=5.12)
#CONFIG_DEVKMEM is not set
# Prohibits provision of kernel image layout
#CONFIG_PROC_KCORE is not set
# Disable VDSO backward compatibility , which makes it impossible
# to use ASLR
#CONFIG_COMPAT_VDSO is not set
# Prevent unprivileged users from retrieving kernel addresses
# with dmesg (8)
CONFIG_SECURITY_DMESG_RESTRICT=y
# Activate retpolines which are necessary to protect yourself from Spectre v2
# GCC> = 7.3.0 is required.
CONFIG_RETPOLINE=y
# Disable the vsyscall table. It is no longer required by libc and is a
# potential source of ROP gadgets.
CONFIG_LEGACY_VSYSCALL_NONE=y
CONFIG_LEGACY_VSYSCALL_EMULATE=n
CONFIG_LEGACY_VSYSCALL_XONLY=n
CONFIG_X86_VSYSCALL_EMULATION=n
# Check the authorisation data structures
CONFIG_DEBUG_CREDENTIALS=y
# Check the notifications data structures
CONFIG_DEBUG_NOTIFIERS=y
# Check kernel lists
CONFIG_DEBUG_LIST=y
# Check the kernel Scatter -Gather tables.
CONFIG_DEBUG_SG=y
# Generate a call to BUG() if corruption is detected.
CONFIG_BUG_ON_DATA_CORRUPTION=y
# Randomly position the free block chaining information of the allocator.
CONFIG_SLAB_FREELIST_RANDOM=y
# CONFIG_SLAB is a dependency of CONFIG_SLAB_FREELIST_RANDOM
CONFIG_SLUB=y
# Protects integrity of the allocator 's metadata.
CONFIG_SLAB_FREELIST_HARDENED=y
# Starting with kernel version 4.13, this option disables the merge of
# SLAB caches
CONFIG_SLAB_MERGE_DEFAULT=n
# Activates the checking of the memory allocator structures and resets
# to zero the zones allocated when they are released (requires the
# activation of the page_poison=on memory configuration option
# added to the list of kernel parameters during boot). It is better to use
# the slub_debug memory configuration option added to the list of
# kernel parameters during boot as it allows finer management
# from the debug slub.
CONFIG_SLUB_DEBUG=y
# Clean up memory pages when they are freed.
CONFIG_PAGE_POISONING=y
# Deep cleaning disabled. This option comes at a significant cost;
# however if the performance impact is compatible with the need
# operational of the equipment , it is recommended to activate it. (<=5.10)
CONFIG_PAGE_POISONING_NO_SANITY=y
# The cleaning of the memory pages is carried out with a rewrite
# of zeros in place of the data (<=5.10)
CONFIG_PAGE_POISONING_ZERO=y
# Disable backward compatibility with brk () which makes it impossible to
# implementation of brk () with ASLR.
#CONFIG_COMPAT_BRK is not set
# Activation of module support
CONFIG_MODULES=y
# This option replaced CONFIG_DEBUG_SET_MODULE_RONX
# in the kernel (> = 4.11)
CONFIG_STRICT_MODULE_RWX=y
# This option replaced CONFIG_DEBUG_SET_MODULE_RONX
# in the kernel (> = 4.11)
CONFIG_MODULE_SIG=y
# Prevent loading modules that are unsigned or signed with a key that
# does not belong to us.
CONFIG_MODULE_SIG_FORCE=y
# Activate CONFIG_MODULE_SIG_ALL allows signing all modules
# automatically during the "make modules_install" step, without this option
# modules must be signed manually using the script
# scripts/sign-file. The CONFIG_MODULE_SIG_ALL option
# depends on CONFIG_MODULE_SIG and CONFIG_MODULE_SIG_FORCE ,
# so they must be enabled.
CONFIG_MODULE_SIG_ALL=y
# Module signing uses SHA512 as hash function
CONFIG_MODULE_SIG_SHA512=y
CONFIG_MODULE_SIG_HASH="sha512"
# Specifies the path to the file containing both the private key and its
# X.509 certificate in PEM format used for signing modules ,
# relative to the root of the kernel.
CONFIG_MODULE_SIG_KEY="certs/signing_key.pem"
# Report on the conditions of the call to the macro kernel BUG ()
# and kill the process that initiated the call. Not setting this variable
# may hide a number of critical errors.
CONFIG_BUG=y
# Shut down the system in the event of a critical error to cut short any
# erroneous behavior.
CONFIG_PANIC_ON_OOPS=y
# Prevents restarting of the machine after a panic which cuts short
# any attempted brute force attack. This obviously has a strong impact
# on production servers.
CONFIG_PANIC_TIMEOUT=-1
# Enables the ability to filter system calls made by an application.
CONFIG_SECCOMP=y
# Activate the possibility of using BPF (Berkeley Packet Filter) scripts.
CONFIG_SECCOMP_FILTER=y
# Enables Linux kernel security primitives , required for LSMs.
CONFIG_SECURITY=y
# Active Yama, which allows to simply limit the use of the system call
# ptrace (). Once the security modules used by the system
# have been selected , support for other security modules should be disabled
# in order to reduce the attack surface.
CONFIG_SECURITY_YAMA=y
# Ensure kernel structures associated with LSMs are always mapped
# read-only after system boot. In the event that SELinux is
# used, make sure that CONFIG_SECURITY_SELINUX_DISABLE is not set,
# for this option to be available. It is then no longer possible to
# disable an LSM after the kernel has booted. It is however still
# possible to do this by modifying the kernel command line. For the
# 4.18 kernel the present LSMs are: AppArmor , LoadPin , SELinux , Smack ,
# TOMOYO and Yama.
#CONFIG_SECURITY_WRITABLE_HOOKS is not set
# Enable compiler plugins support (implies the use of GCC).
CONFIG_GCC_PLUGINS=y
# Amplify entropy generation at equipment startup for those
# having inappropriate entropy sources
CONFIG_GCC_PLUGIN_LATENT_ENTROPY=y
# Clean up the contents of the stack at the time of the exit system call.
CONFIG_GCC_PLUGIN_STACKLEAK=y
# Force initialization of structures in memory to avoid data leakage by
# superimposition with an old structure.
CONFIG_GCC_PLUGIN_STRUCTLEAK=y
# Globalization of the previous option in the case of the passage
# of structure by reference between functions if they have uninitialized fields
CONFIG_GCC_PLUGIN_STRUCTLEAK_BYREF_ALL=y
# Build a random memory map for the kernel system structures.
# This option has a strong impact on performance. The option
# CONFIG_GCC_PLUGIN_RANDSTRUCT_PERFORMANCE=y should be used
# instead if this impact is not acceptable.
CONFIG_GCC_PLUGIN_RANDSTRUCT=y
# Used to prevent SYN flood type attacks.
CONFIG_SYN_COOKIES=y
# Prohibits the execution of a new kernel image after reboot.
#CONFIG_KEXEC is not set
# Prohibits switching to hibernation mode, which allows you to
# substitute the image kernel without his knowledge.
#CONFIG_HIBERNATION is not set
# Disable arbitrary binary format support.
#CONFIG_BINFMT_MISC is not set
# Impose the use of modern ptys (devpts) which number and use
# can be controlled
#CONFIG_LEGACY_PTYS is not set
# If module support is not absolutely necessary , it must be
# disabled.
#CONFIG_MODULES is not set
```
F. A Network config is also important for your bootloader, but it is much more based on individual use cases, so you'll have to decide yourself what direction to go.        
G. A portion of kernel compilation options are dependent upon your arch, so this will also be left up to you.    
H. Ensure GRUB cfg has a password.          
I. Enable UEFI secure boot and disable external boot in firmware.     
J. Lock GRUB.    

3 - SSH Setup  

 Create an SSH key to replace your password (general sys advice below):  
  Use modern key types:  
```
   ed25519 preferred  
   rsa only if >= 4096 bits  
```
 Disable legacy algorithms in your sshd cfg  
 Create a sudo user and then disable root login (make the super user also have an ssh key login).  
 Adjust your SSHD cfg (typical setup below, adjust slightly if needed for system constraints):  
 ```
  PasswordAuthentication no  
  PubkeyAuthentication yes  
  PermitRootLogin no  
  MaxAuthTrs 6 #Many people can lower this, but those with employees may need more trs, this is a catch all. The security risk of high try counts is minimal so long as your server is setup properly.  
  LoginGraceTime 15 #15 seconds to login, since you should be logging in near instantly, holding it longer makes unnecessary concurrent TCP connections in the event of an attack.  
  AllowUsers user1 user2 #(or AllowGroups ssh).  
  X11Forwarding no  
  AllowTcpForwarding no #(unless explicitly needed).  
  PermitEmptyPasswords no  
 ```
 Change your SSH port and do a whitelist with iptables or NFT (simple example below:)  
 ```
  iptables -A INPUT -p tcp -s [ip/range to whitelist, do not include brackets] --dport [YOUR SSH PORT HERE. DO NOT INCLUDE BRACKETS] -j ACCEPT  
  iptables -A INPUT -p tcp --dport [YOUR SSH PORT HERE. DO NOT INCLUDE BRACKETS] -j DROP 
 ```
   * Ensure this is at the top of your iptables or nftables priority. AND note this will lock you out if you lose access to the whitelist IP. To fix this you can reboot or add more static whitelist IPs for redundancy prior to entry.   
   * Changing your port isn't imperative but it isn't irrelevant like many "OPSEC gurus" echo. It's a useful 'strainer' for spotting serious precursor activity about to arise, for example it filters out automated mass scanners from logs. You can do this by modifying /etc/ssh/sshd_config (slightly different depending on OS, but usually that path), just uncomment '#Port' then change the number to a valid unused port. Then restart the ssh service. 
 - You should assign services to their own users with lowest needed perms, or user nobody if there is a major portability or automation concern that can't be easily circumvented.   
 A. Adding multiple services to user nobody means a compromise can spread with much less friction to any service on nobody. Ideally user nobody works best if there are a limited number of services on the system and only 1 service utilizes user nobody. Data plane servers are the main example of where user nobody may be acceptable.   
- I'd suggest putting SSH on its own IP address, and that IP shouldn't be more visibly related than it needs to be to your infrastructure, this can assist in dealing with scanning/general probing. This is already mentioned indirectly later in the document, but I'll place it here for prudence.   
- Some people may suggest you should add a rate limit to your ssh port, this does increase security but can be used by bad actors to DoS your ssh port (can't use the srvc).    
  * I would personally recommend doing a rate limit on a per linux user and or IP address basis. If not, a rate limit can often be counterproductive.  
 - Rotate your SSH keys often and make one per user per machine, don't share keys, and remove ghost keys (access no longer needed).  
 - I'd recommend making a basic ssh logging system (i.e. with a packet sniffer that monitors traffic to your ssh port (interface should be differentiated in logs)).     
   * If your CPU/RAM/DISK is more sensitive than your NIC in saturation, avoid packet sniffers for logging.    
 -  Ensure you disable agent fwding.  
```
  Disable agent forwarding   
  AllowAgentForwarding no
```
 - Make "break-glass" access through some way in an emergency. Make sure it is one time use and secure, only accessible (quickly, though) to people who you trust.


4 - Software Security   

 - Ensure you remain up to date with apt update and apt-get upgrade (or yum, depending on OS).  
  A. For Debian/Ubuntu, unattended-upgrades is useful. For RHEL, dnf-automatic is useful in this regard.  
 - Remove unused packages with apt autoremove and systemctl disable <>  
 - Install monitoring software like tcpdump (network traffic monitor) to spot leaks or issues earlr.  
 - Always look for customizability choices in any given software's documentation (for example, openvpn lets you do a great deal of customization in certificates, encryption, and redundancy).  
 - Keep dependencs updated (manually) and pin versions, along with regular audits.  
   * To clarify, I mean you should ensure dependencs don't auto update, but you should auto update security updates when applicable (typically OS related) and pin with alerts for updates from a software when you can't isolate security updates.  
 - Log...and log a lot...AND ROTATE THEM.  
  A. Explore remote systlog/log shipping.  
  B. Hackers will clear any logs they can manipulate, consider this aspect.  
 - Jail/Chroot srvcs whenever possible.  
 - Time sync will need some work on most default setups: NTP, chrony, systemd-timesyncd, etc for logs, auth, etc.  
 - DDoS Protection is a piece of software security I'll need to go deeper into:   
 A. First things first, you want to make sure your firewall follows the logic of whitelisting what you need and dropping everything else.   
 B. A simple internal firewall with a priority system is a good first step:  ssh whitelist, whitelist stateful traffic, whitelist ports, anything else needing a whitelist for your service/use case, DROP INCOMING.  
 C. The MOST IMPORTANT piece of DDoS protection comes in one factor: upstream customizability and capability: For ex, "Placeholder ISP" can shield 400gbps+ on DDoS floods (in the short term) upstream and allow firewall customization for rules appld before they reach your NIC (upstream).  
 D. Assuming you have some type of upstream DDoS firewall with customization, the most important next step is blocking "filtering based DoS attacks":  
 E. Attackers will attempt to exploit your upstream firewall based on its infrastructure. A good way to hinder their research on said datacenter can be to register your own IP block with ARIN.  
   * Note: This option has its own downsides/issues and it isn't a guarantee: DNS analysis, helpdesk social engineering/bribing, etc.  
   - Filtering attacks often hinge on poor whitelisting policies, routing, and or a lack of redundancy relative to your infrastructure:  
    - Use the server's default dns resolvers (resolv.conf) to avoid dns reflection floods against a stateless upstream filtering infrastructure. Floods reflected off an "external dns"'s IP range (ie Google) can cause an upstream firewall to overfilter (drop your dns).               


F. Base IP routing on your upstream filtering weak points (where they drop legitimate traffic or fail to filter). If they struggle with SYN for example, route with failovers and or with UDP being separated.     
    - Most firewall systems apply filters per IP. Use this to make a failover system via null routing upstream, with the service(s) getting an IP pool from some egress points.


G. Load balancing is another important factor: IPs, concurrent services running, disk writing, etc should all have contingencies for null routing/blackholing or migration of traffic.  
    * Though mentioned above, I'd like to specify IPs here, say you have 30 IPv4 addresses on a 1gb NIC, you could be flooded by a mere 33.3mbps to each IP (roughly 1000 to 5000 pps). It is vital to keep your internal rules not 'per-ip', then with a firewall privy to 'spread out' floods.  

H. Minimize calls to other resources. For example, consider peppers for a website's login system. You shouldn't call your vault api every login POST, this makes you vulnerable to DoS. Most developers opt to call it less and store it on the database server's memory, but that isn't important for this. Where you send JSON, your endpoint can get hammered by huge request body floods. The solution to this is limiting the body size to a kb or so in nodejs or whatever your API runs on, but this is also an aside. The point I wanted to make with both examples is that you want to analyze every call to another resource and mitigate risks in some way.        

- Lock app dependencies to prevent catastrophic updates interfering with your 'security profile'.
-   Add abuse throttling at every level. For example don't rely on your ISP to stop bruteforce logins with their DoS rate limits. Likewise, despite using iptables under your ISP's own upstream firewall (and if possible your own upstream firewall cfg too), your app shouldn't rely on either/both ratelimits to mitigate a bruteforce attack, it should have its own filtering too. It is this "layering" of security that keeps things air tight. Just like increasing a lid’s complexity/seal quality will prolong freshness for its product.   
- Sandbox systemd or Linux Namespaces.  
  * This can break apps, check this first before disabling other security measures or troubleshooting.  
  
  A. Systemd can apply primitives: namespaces, mount protections, cgroups, capability bounding, seccomp, syscall filtering, etc.  
  B. Sandboxing often restricts: file sys writes, access to certain paths, divide nodes, creation/privileges, allowable syscalls, and or 'network family hygiene'. These are the failure points.  
  C. Start with a baseline with the service running as a non-root user before sandboxing.  
  D. Run "systemd-analyze security [yourservice.service]" for insight.    
  E. Begin adding constraints and look for errors.  

   ```
   #/etc/systemd/system/service.ex.d/hardening.conf  
   [Service]  
   NoNewPrivileges=yes  
   PrivateTmp=yes  
   ProtectHome=yes  
   ProtectSystem=full  
   PrivateDevices=yes  
   LockPersonality=yes  
   MemoryDenyWriteExecute=yes  
   RestrictSUIDSGID=yes
  ```

F. Once it works with the basics get specific: Use capabilityboundingset=, ambientcapabilities=, systemcallfilter=, restrictnamespaces=, protectsystem=, readwritepaths= , ProtectControlGroups=yes, RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6  
 - Apparmor (Ubuntu/Debian) or SELinux (RHEL) and avoid lowering/compromising security to workaround apparmor/selinux errors.   
  A. Make sure to check the denial/error in logs before surmising an issue from a new patch, you don't wanna chase ghosts.  
  B. Realistically only disable your mandatory access control if debugging/troubleshooting.  
 - Having multiple IPs yields several benefits so long as they're setup correctly.  
  A. Assign externally exposed services to dedicated IPs or its own egress pool if it can be done.  
    * Things like SSH and a web server for example, should be on different IPs.
 
  B. Optimize to null route (upstream, if an option) per IP to maximize redundancy in an attack.  
  C. Use good DNS hygiene: don't use any unnecessary DNS records that associate services together.   
   * For example the reverse dns of a commercial VPN shouldn't be its control plane's API or something.



  D. Apply filtering policies specific to each IP: drop any protocol that the service the IP 'runs' doesn't use.   
  E. Load balancing rules for attacks spread over multiple IPs.  
  F. Carefully monitor and control how traffic moves between these IPs and or interfaces.        




 - ```# systemctl list -units --type service``` lists all services running on a server.
 - ```# find / -type f -perm /111 -exec getcap {} \; 2>/dev/null``` lists EXEs which have at least 1 Linux capability enabled.     
- I may have already said this, but it is very important that you read documentation for software you use. Preferably scaling depth in your research depending on how often you use it and how critical it is: for example if you're making an API, express.js is going to be more important to read up on than Stacer or something. You'll also dramatically increase your retention and interest if you implement(some of) the examples as you read through.   

5 - Ongoing Maintenance/Lifecycle   

- Security updates should be automatic, as stated previously.  
- Schedule non-security updates weekly/monthly (don't do auto update, though). 
- Log update outcomes (failed auto upgrades or reboot-required state).
- Track kernel EOL AND OS EOL, make sure you don't just assume it is the same.  
- Emergency Path protocol: you (and employees, if you have them) need to know what to do (your protocol) when something happens: ssh drops a cve, unauthorized access, actively under attack, etc.  
- Plan for deterioration relative to the rest of cyber security: crypto algs, tls version, ssh key types, file systems, init systems, etc.    
- Log any manual change.  
- Define log retention windows.  
- Local logs should be short but remote ones can be longer.  
- Ensure log integrity (ie who can append and at what stage?).  
- Schedule "check in" days for servers or a set of them.   
- Classify your backup data and put it somewhere safe.  
- Have alerts set if any monitoring software goes down (monitor monitoring software).  
- Incident readiness, same as emergency path protocol but more broad, naturally. Don't let uncertainty or panic motivate you. The written text is in a much calmer mental state that is forward thinking and pragmatic, rely on it for anything preparable, this scales exponentially with employees.   
- If you're renting or have upstream dependencies, ensure you're checking your email and have all notifications on for them.  
- Disk handling/discardment.

A. Depending on your use case, this can vary.   
B. A simple and free solution is to do a double pass over, writing over the disk twice. Advanced forensics can recover this method, though.  
C. Destruction of disks supervised by you after multiple write overs on the disk.  
   * Paradoxically can be more expensive to do it well than to just send it in somewhere due to the stuff needed to actually destroy the data.                                             

D. Certified Data Destruction from a third party after doing your pass overs.  


6 - Misc  
- Depending on priority and budget, ordering pen testing (after performing your own exhaustive pen test, of course) can give clients confidence and lower risk substantially.   
- Infrastructure scaling should ideally partition services/components to a whole, specific server or VM when possible. This segments attacks and is more friendly with scalability.

- Threat Modeling and Attack Analysis  
  A. These two are useful tools once you've got your server setup and are ready to run and perform whatever operation the server is made for.              
  B. At the most basic level, these concepts are intuitive: threat modeling is spotting threats and theories about weak points, and attack surface analysis simply sees what can be attacked.      
  C. Dataflow diagrams are useful tools for both of these to map trust boundaries and how traffic routes throughout the system.    
  D. Many articles exist on these two topics; I'd recommend reading some or watching videos on these whenever you find the time.       
 
 -  This is the end, but the most important thing in security is knowing you don't know everything. No guide can cover every exploit and no one can retain what they read perfectly. So it is important to always challenge assumptions, ask questions, be informed, and stay vigilant. Thank you for taking time to read my manpage on unmanaged servers.   
 - MORE COMING TO THIS SECTION/TBD     


