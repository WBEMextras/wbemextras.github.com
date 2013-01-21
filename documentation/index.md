---
layout: default
title: WBEMextras documentation
---

## Introduction ##

This document will explain the scripts developed to install, upgrade and verify the health check of HP System Insight Management (HPSIM) Insight Remote Support Advanced (IRSA) on HP-UX based systems.

HP-UX systems use next to SNMP to send traps to the HPSIM server, also the WBEM protocols via the so called cimproviders to comminucate with the HPSIM server about their health status. The System Fault Management (SFM) software is used to send hardware failure reports via the HPSIM server and the IRSA plug-in (running on top of the HPSIM server) to the HP Support centre in an automatic way with any user intervention.

However, the HPSIM IRSA software components are known to be very problematic to install and/or upgrade on HP-UX systems because of the many inter-dependencies [HP IRSA Configuration Guide] (http://bizsupport2.austin.hp.com/bc/docs/support/SupportManual/c03052695/c03052695.pdf)
The question may arise why do we need some scripts to install software on HP-UX systems as `/usr/sbin/swinstall` command does this already?

During a migration project of ISEE towards HPSIM on HP-UX systems in 2009 we were faced with various types of problems, which we were able to group into the following sections:

Groups are:
- Software installation problems, e.g. missing pre-requisite patches, or other software components required. Also, during upgrades we were faced with backward requirements which we thought these requirements were already fulfilled.
- Configuration problems whereby in HPSIM it was not possible to make a successful WBEM event subscription, or WEBES subscription did not happen automatically.
- Operational problems which lead to unmonitored HP-UX systems where we were in the opinion everything worked fine. The most common issues were with cimprovider daemons that took 100% of CPUs, or agents that were in a degraded state, or sometimes our non-privileged account (wbem) became locked on the system. All these various types of (small) issues lead to the fact that a HP-UX system cannot send out an alert to the HP SIM server and therefore no automatic incident could be created at HP.

To assist us in getting the HPSIM-IRSA software properly installed we designed a script to accomplish this in a structured way. Upgrading existing HP-UX systems with newer releases of HPSIM-IRSA components was also a challenge and therefore, we wrote also a script to assist in task. To make sure all the software components are installed properly and configured according the rules we wrote a third script to check the HPSIM Healh status on the HP-UX systems.
In the following chapters we will explain in detail all the scripts, software depot requirements, and the usage of them in detail.

## The HP-UX Software Depots ##

The scripts that will help us to install the HPSIM-IRSA software components on HP-UX systems must first be downloaded from either HP-UX Application CD-ROMs or from the HP Software Depot site [HP Software Depot Download site](http://www.software.hp.com)
The HP-UX Operating Systems in scope are HP-UX 11.11, HP-UX 11.23 and HP-UX 11.31. Older versions of HP-UX Operating System are not supported by HPSIM-IRSA anymore. If you wish you can download from the HP Software Depot site each software depot individually. In our depots we collected the following software depots.

It would be nice that the collection of software depots could be made available as one big software depot, e.g. `irsa-1111.depot.gz` and made available for download for usage.

### Software depot for HP-UX 11.11 ###

The versions of the software depots can vary over time. This list was made in January 2012.

<pre>
  B9073BA                               B.11.11.09.02.00.07	HP-UX iCOD Instant Capacity (iCAP)
  EventMonitoring                       A.04.20.11.05		Event Monitoring Service
  FCProvider                            B.11.11.08		CIM/WBEM Provider for Fibre Channel HBAs
  FileSysProvider                       B.11.11.01		HP-UX File System CIM Provider
  IOTreeIndication                      B.11.11.0712		CIM/WBEM Indication Provider for IOTree subsystem
  IOTreeProvider                        B.11.11.06		CIM/WBEM Provider for HPUX IOTree
  LVMProvider                           B.11.11.05		CIM/WBEM Provider for LVM
  OnlineDiag                            B.11.11.21.07		HPUX 11.11 Support Tools Bundle, March 2011
  OpenSSL                               A.00.09.08s.001		Secure Network Communications Protocol
  RS-ACC                                A.05.50.26.007		HP Remote Support Advanced Configuration Collector
  SCSIProvider                          B.11.11.07		CIM/WBEM Provider for SCSI HBA
  SysFaultMgmt                          A.04.04.02.01		HPUX 11.11 System Fault Management
  SysMgmtWeb                            A.2.2.9.3.1		HP-UX Web Based System Management User Interfaces
  UtilProvider                          A.01.08.05.02		HP-UX Utilization Provider
  WBEMP-LAN-00                          B.11.11.0706		LAN Providers for Ethernet LAN interfaces.
  WBEMP-LAN-TEST-00                     B.11.11.03		LAN Provider Test package
  WBEMSvcs                              A.02.07.06		HP WBEM Services for HP-UX
  hpuxwsApache                          B.2.0.59.16		HP-UX Apache-based Web Server
  nParProvider                          B.12.02.07.03		nPartition Provider - HP-UX
  vParProvider                          B.11.11.01.06		vPar Provider - HP-UX
  EMS-KRMonitor                         A.11.11.05		EMS Kernel Resource Monitor
  WBEMextras                            A.01.00.04		HP WBEM Extras for HP-UX
</pre>

The above list contains two depots that are not available on the HP Software Depot site:

1. The `RS-ACC` depots are available from the HPSIM server (or Central Management Server) location `<dir>:\HP\Installers\UC\ACC\HPUX\`, and
2. the `WBEMextras` depot was made by us and will be explained in a seperate chapter.

### Software depot for HP-UX 11.23 ###

The software depots for HP-UX 11.23 contains the following components:

<pre>
  B9073BA                       B.11.23.10.00.00.11	HP-UX iCOD Instant Capacity (iCAP)
  CommonIO                      B.11.23.1102		Common IO Drivers
  EventMonitoring               A.04.20.23.06		Event Monitoring Service
  FCProvider                    B.11.23.1103		CIM/WBEM Provider for Fibre Channel HBAs
  FileSysProvider               B.11.23.0706		HP-UX File System CIM Provider.
  IOTreeIndication              B.11.23.1003		CIM/WBEM Indication Provider for IOTree subsystem
  KernelProviders               B.01.01.02.01		HPUX Kernel Providers
  LVMProvider                   R11.23.009		CIM/WBEM Provider for LVM
  OnlineDiag                    B.11.23.13.05		HPUX 11.23 Support Tools Bundle, December 2009
  OpenSSL                       A.00.09.08s.002		Secure Network Communications Protocol
  RAIDSAProvider                B.11.23.1012		Smart Array RAID Provider
  RS-ACC                        A.05.50.26.007		HP Remote Support Advanced Configuration Collector
  RS-ACC                        A.05.50.26.007		HP Remote Support Advanced Configuration Collector
  SASProvider                   B.11.23.1012		SerialSCSI Provider
  SCSIProvider                  B.11.23.1009		CIM/WBEM Provider for SCSI HBA
  SD                            B.11.23.1009.352	HP Software Distributor
  SysFaultMgmt                  B.07.05.01.02		HPUX System Fault Management
  SysMgmtBASE                   B.00.02.03.03		SysMgmtBASE
  SysMgmtWeb                    A.3.2.0			HP-UX Web Based System Management User Interfaces
  UtilProvider                  A.01.08.05.02		HP-UX Utilization Provider
  WBEMP-LAN-00                  B.11.23.1009		LAN Providers for Ethernet LAN interfaces.
  WBEMSvcs                      A.02.09.08		HP WBEM Services for HP-UX
  WBEMpatches-1123              B.2011.12.13		IRSA patch bundle for HP-UX 11.23
  hpuxwsApache                  B.2.0.59.16		HP-UX Apache-based Web Server
  nParProvider                  B.23.01.07.05		nPartition Provider - HP-UX
  vParProvider                  B.11.23.01.07		vPar Provider - HP-UX
  PHCO_34721                    1.0			killall(1M) cumulative patch
  PHCO_40382                    1.0			Cumulative IO Tree Provider patch
  WBEMextras                    A.01.00.04		HP WBEM Extras for HP-UX
</pre>

The above list contains two depots that are not available on the HP Software Depot site:

1. The `RS-ACC` depots are available from the HPSIM server (or Central Management Server) location `<dir>:\HP\Installers\UC\ACC\HPUX\`, and
2. the `WBEMextras` depot was made by us and will be explained in a seperate chapter, and
3. the `WBEMpatches-1123` are a bunch of patches downloaded from HP Support Center [HP Support Center](https://h20566.www2.hp.com/portal/site/hpsc/public/), but we will explain the creation of this patch bundle seperately.

### Software depot for HP-UX 11.31 ###

The software depots for HP-UX 11.31 contains the following components:

<pre>
  B9073BA                       B.11.31.10.03.00.06	HP-UX iCOD Instant Capacity (iCAP)
  CommonIO                      B.11.31.1109		Common IO Drivers
  EventMonitoring               A.04.20.31.07		Event Monitoring Service
  OnlineDiag                    B.11.31.17.04		HPUX 11.31 Support Tools Bundle, Mar 2011
  OpenSSL                       A.00.09.08s.003		Secure Network Communications Protocol
  RS-ACC                        A.05.50.26.007		HP Remote Support Advanced Configuration Collector
  SysMgmtPlus                   A.06.00.04.01		HP-UX SMH Supplemental Functionality
  SysMgmtWeb                    A.3.2.2			HP-UX Web Based System Management User Interfaces
  UtilProvider                  A.01.08.06.02		HP-UX Utilization Provider
  WBEMMgmtBundle                C.02.01			WBEMMgmtBundle
  hpuxws22Apache                B.2.2.15.10		HP-UX Apache-based Web Server
  hpuxws22Tomcat                B.5.5.34.02		HP-UX Tomcat-based Servlet Engine
  hpuxws22Webmin                A.1.070.13		HP-UX Webmin-based Admin
  PHCO_36032                    1.0			killall(1M) cumulative patch
  PHCO_36038                    1.0			esmd(1M) cumulative patch
  PHCO_40087                    1.0			Event Management (EVM) cumulative patch
  PHCO_41122                    1.0			IOTreeModule Provider Patch
  PHCO_41483                    1.0			cumulative libipmimsg patch
  SysMgmtBase                   B.00.02.04.09		HP-UX Common System Management Enablers
  WBEMextras                    A.01.00.04		HP WBEM Extras for HP-UX
</pre>

The above list contains two depots that are not available on the HP Software Depot site:

1. The `RS-ACC` depots are available from the HPSIM server (or Central Management Server) location `<dir>:\HP\Installers\UC\ACC\HPUX\`, and
2. the `WBEMextras` depot was made by us and will be explained in a seperate chapter

### Copying the software depots ###

The best place to copy the software depots is on the HP-UX Ignite server and foresee a file system with plenty of space (5 GBytes). Decide on a file system path name, e.g. `/var/opt/ignite/depots/GLOBAL/irsa` and create the path as follow: 

<pre>
$ mkdir -m 755 -p /var/opt/ignite/depots/GLOBAL/irsa
</pre>

Under this path we can +swcopy+ our depot files downloaded from HP, e.g. for HP-UX 11.11 execute the following `swcopy` command and verify with +swlist+ if the depot was successfully copied: 

<pre>
$ swcopy -x enforce_dependencies=false -x mount_all_filesystems=false -x autoselect_dependencies=false \
         -s /depots/irsa-1111.depot \* @ /var/opt/ignite/depots/GLOBAL/irsa/11.11
$ swlist -s /var/opt/ignite/depots/GLOBAL/irsa/11.11
  B9073BA                               B.11.11.09.02.00.07	HP-UX iCOD Instant Capacity (iCAP)
  EventMonitoring                       A.04.20.11.05		Event Monitoring Service
  FCProvider                            B.11.11.08		CIM/WBEM Provider for Fibre Channel HBAs
  FileSysProvider                       B.11.11.01		HP-UX File System CIM Provider
  IOTreeIndication                      B.11.11.0712		CIM/WBEM Indication Provider for IOTree subsystem
  IOTreeProvider                        B.11.11.06		CIM/WBEM Provider for HPUX IOTree
  LVMProvider                           B.11.11.05		CIM/WBEM Provider for LVM
  OnlineDiag                            B.11.11.21.07		HPUX 11.11 Support Tools Bundle, March 2011
  OpenSSL                               A.00.09.08s.001		Secure Network Communications Protocol
  RS-ACC                                A.05.50.26.007		HP Remote Support Advanced Configuration Collector
  SCSIProvider                          B.11.11.07		CIM/WBEM Provider for SCSI HBA
  SysFaultMgmt                          A.04.04.02.01		HPUX 11.11 System Fault Management
  SysMgmtWeb                            A.2.2.9.3.1		HP-UX Web Based System Management User Interfaces
  UtilProvider                          A.01.08.05.02		HP-UX Utilization Provider
  WBEMP-LAN-00                          B.11.11.0706		LAN Providers for Ethernet LAN interfaces.
  WBEMP-LAN-TEST-00                     B.11.11.03		LAN Provider Test package
  WBEMSvcs                              A.02.07.06		HP WBEM Services for HP-UX
  hpuxwsApache                          B.2.0.59.16		HP-UX Apache-based Web Server
  nParProvider                          B.12.02.07.03		nPartition Provider - HP-UX
  vParProvider                          B.11.11.01.06		vPar Provider - HP-UX
  EMS-KRMonitor                         A.11.11.05		EMS Kernel Resource Monitor
  WBEMextras                            A.01.00.04		HP WBEM Extras for HP-UX
</pre>

We repeat these commands for HP-UX 11.23 and HP-UX 11.31 software depots.


### Creation of WBEMpatches-1123 patch depot ###

Login on the HP Support center (registration is required, but free of charge) and select *patch management*, *find a specfic patch* and type in the following list of patches in the "search test" box, also select from the pull-down menus *Search by Patch ID* and *OS revision* "HP-UX 11.23", then finally click on the *Search* button:

- PHSS_41495
- PHKL_34795
- PHKL_40918
- PHCO_41860
- PHSS_35055
- PHSS_41422
- PHKL_31500
- PHSS_36870
- PHKL_36288
- PHSS_42043

The patches will be shown on the screen and select the most recent ones and click on the *Add to my patch list* button. The patches can be downloaded by clicking on the *Download Selected* button and select the *Download GZIP Package* option and then the patch bundle will be downloaded and saved as `hpux_11.23_<random nr>.tgz`.

The following steps must be executed on an HP-UX system, the Operating System version is not relevant for these steps. We will extract the patches and create a patch depot of it:

<pre>
$ swlist -s ./depot 
  WBEMpatches-1123      B.2012.01.27   IRSA patch bundle for HP-UX 11.23
</pre>

The `$PWD/depot` is a directory containing all the patches, but we prefer to have a portable depot (is like a `tar` archive) which can be easily moved to another HP-UX system via scp, e-mail or whatever means of transport. To accomplish this execute the following command:

<pre>
$ swpackage -d $PWD/WBEMpatches-1123.depot -x target_type=tape -s $PWD/depot WBEMpatches-1123
       * Tape #1: CRC-32 checksum &amp; size: 948628547 155484160
$ cksum WBEMpatches-1123.depot
948628547 155484160 WBEMpatches-1123.depot
</pre>

This patch bundle will afterwards be copied into the IRSA software depot of HP-UX 11.23, e.g.

<pre>
swcopy -x enforce_dependencies=false -x mount_all_filesystems=false -x autoselect_dependencies=false \
       -s $PWD/WBEMpatches/WBEMpatches-1123.depot \* @ /var/opt/ignite/depots/GLOBAL/irsa/11.23
</pre>

### Maintaining the software depots ###

Nothing is perfect, so at least on a quartely basis updates will be released by HP. Therefore, it is important to keep these software depots up to date. If needed, ask assistance at HP (they have trained consultants in-house who can do this task for you).
It is key not to forget to keep the software depots in good health as these will be installed on the HP-UX managed nodes through out your organization. To avoid repeating the same exercises over and over again make sure that the source depots, on the Ignite/UX server, are in good shape.

## The HPSIM configuration file HPSIM_irsa.conf ##

The configuration file `/usr/local/etc/HPSIM_irsa.conf`, which can be placed in advance or will be created by the `HPSIM-Check-RSP-readiness.sh` script (if it didn't exist yet). 

The important variables are:

- *WbemUser=wbem* is the non-privileged user who will be used for cimprovider queries
- *mailusr=* is the mail destination who will be receiving the output logging. The output is always saved locally (see further).
- *SimServer=sim-server* contains the fully qualified domain name of the HPSIM server (also known as the Central Managed Server).
- *MaxTestDelay=0* is delay variable used before sending test events to the HPSIM server (could be useful when doing lots of systems in batch). However, the default value is 0 seconds.
- *dlog=/var/adm/install-logs* contains the directory where the logging should be saved into.
- *IUXSERVER=ignite-ux* contains the fully qualified domain name of the Ignite/UX server of software depot server.
- *baseDepo=/var/opt/ignite/depots/GLOBAL/irsa* is the location where we put our software on the IUXSERVER. Be careful, the Operating System version should be added to the baseDepo name, e.g. for HP-UX 11.31 the real path becomes `${baseDepo}/${os}`
- *ENCPW=6u2CMymnCznQo* is the encrypted password of the wbem account. To create a new password use the command `openssl passwd -crypt` and copy-paste the output in the configuration file.
- *HpsmhAdminGroup=hpsmh* contains the group name of the HP System Management Homepage. The wbem account will have this as a secondary group and as such can be used to login at the HP System Management Homepage interface with administrator rights. This useful to exchange SSL certificates between HPSMH and HPSIM.

When we remove all comments from the configuration file we see something like the following:

<pre>
grep -v \# /usr/local/etc/HPSIM_irsa.conf | sed -e '/^$/d'
WbemUser=wbem
mailusr=root
SimServer=sim-server
MaxTestDelay=0
dlog=/var/adm/install-logs
IUXSERVER=iux-server
baseDepo=/var/opt/ignite/depots/irsa
ENCPW="6u2CMymnCznQo"
</pre>

## The SIM Installation and Loggings (wbemextras) Scripts ##

The scripts were developed to install, configure, upgrade and check the HPSIM IRSA related software on the HP-UX managed nodes. See it as an aid in getting these installation and configuration tasks done in a simple way. The only thing we need are the software depots which we already have installed (or copied) on our central Ignite/UX server and the first script `HPSIM-Check-RSP-readiness.sh` and its accompying configuration file (albeit not required to have). 

We will discuss all the scripts in detail, but will start with the most import script and that is `HPSIM-Check-RSP-readiness.sh` and secondly we will discuss the configuration file _HPSIM_irsa.conf_ in more detail. Afterwards, we will go over the `HPSIM-Upgrade-RSP.sh` script and `HPSIM-Healthcheck.sh` scripts. We still have one special script `restart_cim_sfm.sh` which will be bundle in the WBEMextras depot and which is initiated via crontab.

### The HPSIM-Check-RSP-readiness.sh script ###

The easiest way to explain the `HPSIM-Check-RSP-readiness.sh` script is via the help output.

The short usage output (*-?* option):

<pre>
./HPSIM-Check-RSP-readiness.sh -?
Usage: HPSIM-Check-RSP-readiness.sh [-vhpi] [-u WbemUser] [-g HpsmhAdminGroup] [-d IP:path] \
       [-m email1,email2] [-c conf-file>] [-s SimServer]
</pre>

Some important knowledge you need to understand. The HPSIM WBEM communication between the HPSIM server (also known as the Central management Server, or abbreviated CMS) and the HP-UX managed node is done in a secure way via the Secure Socket Layer (SSL) and via Secure Shell (SSH). In particular when we use SSH we preferrably use a non-privelge user (non-root) and that is where the *WbemUser* is used for (the *-u* option).

Once we start configuration part the *WbemUser* will be created (if not yet existing) with a default password (hpinvent) or use option *-p* to be prompted for a password.

Option *-g* might be required if you want the _WbemUser_ to be added to a second group *hpsmh* (the HP system management homepage group) which is needed to grant the *WbemUser* administrator's rights within the HP system management homepage interface). In some organizations the administrator's group may be different.

We already know the usage output of the `HPSIM-Check-RSP-readiness.sh` script, but we can see more if we use the *-h* option instead.

The help output (-h option)

<pre>
$ ./HPSIM-Check-RSP-readiness.sh -h
NAME
  HPSIM-Check-RSP-readiness.sh - Install and configure IRSS/IRSA software on HP-UX 11i systems
SYNOPSIS
  HPSIM-Check-RSP-readiness.sh [ options ]
DESCRIPTION
  This script will check the pre-requisites, install all required software
  pieces, does the needed configuration and send/save a detailed installed
  report. The script knows two modes (preview and installation).
OPTIONS
  -h
        Print this man page.
  -i
        Run HPSIM-Check-RSP-readiness.sh in "installation" mode. Be careful, this will install software!
        Default is preview mode and is harmless as nothing will be installed nor configured.
  -u WbemUser
        Non-priviledge account to use with IRSS/IRSA WBEM protocol (default wbem).
  -g HpsmhAdminGroup
        The HP System Management Homepage Admin Group (default hpsmh).
  -m email1,email2...
        When this option is used, an  email notification is sent when an error
        occurs.  Use this option with a valid SMTP email address.
  -d [IP address or FQDN of Software Depot server]:/Absolute/path/to/base/depot
        Example: -d 10.0.0.1:/var/opt/ignite/depots/GLOBAL/rsp/pre-req
        The actual software depots are then located under:
          B.11.11 : /var/opt/ignite/depots/GLOBAL/rsp/pre-req/11.11
          B.11.23 : /var/opt/ignite/depots/GLOBAL/rsp/pre-req/11.23
          B.11.31 : /var/opt/ignite/depots/GLOBAL/rsp/pre-req/11.31
        However, -d /cdrom/rsp/pre-req is also valid where same rules apply as above.
  -p
        Prompt for a password for the WBEM user (non-priviledge user).
        Default password is "hpinvent" (without the double quotes).
  -c path/configuration-file
        Will store a configuration file as specified which can be used when we
        run this script again. However, other scripts will benefit from this too:
        - HPSIM-HealthCheck.sh
        - HPSIM-Upgrade-RSP.sh
        - restart_cim_sfm.sh
        Default location/name is [ /usr/local/etc/HPSIM_irsa.conf ]
  -s HPSIM Server
        The HP SIM Server address (FQDN or IP address) where $(hostname) will be defined.
  -v
        Prints the version of HPSIM-Check-RSP-readiness.sh.
EXAMPLES
    HPSIM-Check-RSP-readiness.sh -d 10.1.2.3:/var/opt/ignite/depots/irsa
        Run HPSIM-Check-RSP-readiness.sh in preview mode only and will give a status update.
    HPSIM-Check-RSP-readiness.sh -i
        Run HPSIM-Check-RSP-readiness.sh in installation mode and use default values for
        IP address of Ignite server (or SD server) and software depot path
IMPLEMENTATION
  version       Id: HPSIM-Check-RSP-readiness.sh $
  Revision      UNKNOWN
  Author        Gratien D'haese
  Release Date  19-Jan-2012
</pre>

The *-c* option is used to point to a configuration file which will overrule the command arguments. If we do not use the *-c* the script will create a configuration file `/usr/local/etc/HPSIM_irsa.conf` with the arguments given or with the default values. We find it a best practice to prepare a configuration file on-front to avoid mistakes and to guarantee consistency across the different scripts. We will discuss the configuration file a bit later.

The *-d* option defines the path where to software is stored and it combines the Ignite/UX server address (FQDN or IP address) and the absolute path to the software depots (without the HP-UX operating system version number). E.g.  `10.1.2.3:/var/opt/ignite/depots/irsa` can be used as `swlist -s 10.1.2.3:/var/opt/ignite/depots/irsa/*11.11*`

The *-m* option defines the mail destination (more then one can be defined) which will get the report of the installation and configuration details.

The *-s* option defines the HPSIM server address (FQDN or IP address) which is not used in this script, but which will be recorded in the configuration file for later usage by the other scripts.

We can use the script with command arguments (without the *-i* option means "preview" mode) - see following example run on HP-UX 11.11. It we would have created a configuration file in front then the script name would have been sufficient.

Preview run on HP-UX 11.11

<pre>
$ ./HPSIM-Check-RSP-readiness.sh -u wbem -p -m gdhaese -d ignite-ux:/var/opt/ignite/depots/GLOBAL/irsa \
  -g se -s sim-server
Enter the secret password for user wbem (do not forget it!)
Password:
Verifying - Password:
  Installation Script: HPSIM-Check-RSP-readiness.sh
        Ignite Server: ignite-ux
           OS Release: 11.11
                Model: 9000/800/L2000-44
    Installation Host: hpux01
    Installation User: root
    Installation Date: 2012-02-01 @ 10:47:18
     Installation Log: /var/adm/install-logs/HPSIM-Check-RSP-readiness.2012-02-01.104718.scriptlog
 ** Running in preview mode.
 ** HP-UX 11.11 support is OK
 ** Patch Level is OK
 ** Corrupt filesets - none were found - OK
 ** ISEEPlatform was not found - OK
 ** Current SysFaultMgmt version is OK
 ** OpenSSL version is OK
 ** WBEMService is OK
 ** OnlineDiag version is OK
 ** Stop the HP System Mgmt Homepage daemon
 ** hpuxwsApache release is OK
 ** hpuxws22Apache release not found - N/A
 ** System Management Homepage version is OK
 ** Start the HP System Mgmt Homepage daemon
 ** No IRSA patch bundle for HP-UX 11.11 required
 ** System Fault Management (SFM) is OK
 ** WBEMMgmtBundle is not required on 11.11 - N/A
 ** Install HP Remote Support Advanced Configuration Collector
 ** nParProvider version is OK
 ** vParProvider version is OK
 ** (Re-)Configure all CIM Providers installed on this system
 ** To view or run the installation script - check /tmp/RSP_readiness_hpux01.install
 ** To view or run the config script - check /tmp/RSP_readiness_hpux01.config
 ** There were no errors detected.
</pre>

There are a few points to know about this script before we dig deeper. The `HPSIM-Check-RSP-readiness.sh` script knowns two modes of operations, a preview and an installation mode. The preview mode, which is the default, will not install nor configure the HP-UX managed node in anyway. It will create two other scripts

- `/tmp/RSP_readiness_$(hostname).install`
- `/tmp/RSP_readiness_$(hostname).config`

The above mentioned scripts can be viewed and analyzed if the proposed solution are ok. 
The `/tmp/RSP_readiness_$(hostname).install` script will and shall be different each time you run (or re-run) the `HPSIM-Check-RSP-readiness.sh`. We can see the content of `/tmp/RSP_readiness_$(hostname).install` with the `cat` command:

<pre>
$ cat RSP_readiness_hpux01.install
#!/usr/bin/ksh
{
# all actions run in "preview" mode!!
echo \# Test 1 : HP-UX B.11.11 supported - OK
echo \# Test 2 : Patch Level - OK
echo \# Test 3 : Corrupt filesets found - N/A
echo \# Test 4 : ISEEPlatform - OK
echo \# Test 5 : Unsupported System Fault Mgt. \(SFM\) found - OK
echo \# Test 6 : OpenSSL - OK
echo \# Test 7 : WBEMservices - OK
echo \# Test 8 : OnlineDiag - OK
echo \# Test 9 : Stop System Mgmt Homepage
 /sbin/init.d/hpsmh stop
echo \# Test 10 : Apache v2.0 - OK
echo \# Test 11 : Apache v2.2 - N/A
echo \# Test 12 : SysMgmtWeb - OK
echo \# Test 13 : Start System Mgmt Homepage
 /sbin/init.d/hpsmh start
echo \# Test 14 : HP SIM/IRSA Patches - OK
echo \# Test 15 : System Fault Mgt - OK
 /usr/sbin/swlist -l fileset -a state SysFaultMgmt | grep -v -E '\#|configured' | grep 'installed'
 if [ $? -eq 0 ]; then
 /usr/sbin/swconfig -vp -x mount_all_filesystems=false -x reconfigure=true SysFaultMgmt
 fi
echo \# Test 16 : WBEMMgmtBundle - N/A
echo \# Test 17 : Remote Support Adv Conf Collector
 /usr/sbin/swinstall -vp -x enforce_dependencies=false -x mount_all_filesystems=false -s ingnite-ux:/var/opt/ignite/depots/GLOBAL/irsa/11.11 RS-ACC
echo \# Test 18 : nParProvider - OK
echo \# Test 19 : vParProvider - OK
echo \# Test 20 : WBEMextras
# Install WBEMextras
 /usr/sbin/swinstall -vp -x reinstall=false -x mount_all_filesystems=false -s ignite-ux:/var/opt/ignite/depots/GLOBAL/irsa/11.11 WBEMextras
echo \# Test 21 : Configure the CIM Providers
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false FCProvider
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false FileSysProvider
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false IOTreeIndication
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false IOTreeProvider
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false LVMProvider
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false SCSIProvider
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false UtilProvider
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false WBEMP-LAN-00
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false nParProvider
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false vParProvider
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false CM-Provider-MOF
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false OPS-Provider-MOF
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false WBEMP-LAN
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false SW-DIST
 /usr/sbin/swconfig -vp -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false SFM-CORE
} > /var/adm/install-logs/RSP_readiness_hpux01.inst-install.scriptlog 2>&1
</pre>

The second created script is `/tmp/RSP_readiness_$(hostname).config` which contains the configuration part for WBEM and System Fault Management (SFM).

<pre>
$ cat RSP_readiness_hpux01.config
#!/usr/bin/ksh
{
export PATH=/usr/sbin:/usr/bin:/opt/samba/bin:/opt/ansic/bin:/usr/ccs/bin:/usr/contrib/bin:/opt/hparray/bin:\
/opt/nettladm/bin:/opt/upgrade/bin:/opt/fcms/bin:/opt/pd/bin:/opt/resmon/bin:/opt/gnome/bin:/opt/perf/bin:\
/usr/bin/X11:/usr/contrib/bin/X11:/opt/graphics/common/bin:/opt/prm/bin:/usr/sbin/diag/contrib:/opt/mx/bin:\
/opt/sec_mgmt/bastille/bin:/opt/ignite/bin:/usr/local/bin:/usr/sbin:/opt/ssh/bin:/opt/OV/bin/OpC:/opt/OV/bin:\
/opt/hpnpl//bin:/opt/mozilla:/opt/hpsmh/bin:/opt/langtools/bin:/opt/imake/bin:/opt/sfm/bin:/opt/wbem/bin:\
/opt/wbem/sbin:/opt/cfg2html:/usr/sbin:/sbin:/sbin:/home/root:/usr/bin:/usr/sbin:/sbin:/opt/wbem/bin:/opt/wbem/sbin
# swconfig EMS-Core - need to find the latest release to configure (move from install to here)
r=`swlist -l product | grep -i EMS-core | tail -1 | awk '{print $2}'`
swconfig -x autoselect_dependencies=false -x reconfigure=true -x mount_all_filesystems=false EMS-Core,r=$r
echo
# List of active CIM Providers
echo \# List of active CIM Providers
cimprovider -l -s
echo
# Check Event Monitoring
echo \# Check Event Monitoring
echo q | /etc/opt/resmon/lbin/monconfig | grep -E 'EMS|STM'
# Hardware monitors always use EMS
echo \# Hardware monitors always use EMS
echo
# check special wbem account (wbem) for monitoring with HP SIM
# grep ^wbem /etc/passwd
echo \# grep ^wbem /etc/passwd
grep "^wbem" /etc/passwd 2>&amp;1
[ $? -ne 0 ] &amp;&amp; {
        # Account wbem does not exist - creating one
        /usr/sbin/useradd -g users -G se wbem
        /usr/sam/lbin/usermod.sam -F -p ARbwd5UKVpJQs wbem
        /usr/lbin/modprpw -m exptm=0,lftm=0 wbem
        /usr/lbin/modprpw -v wbem
        }
echo
# Print current CIM configuration
echo \# Print current CIM configuration
cimconfig -l -p
cimconfig -s enableSubscriptionsForNonprivilegedUsers=true -p
cimconfig -s enableNamespaceAuthorization=true -p
echo
# Stop cimserver
echo \# Stop cimserver
cimserver -s
echo \# Add secondary se group to the wbem account
/usr/sbin/usermod -G se wbem
echo
# Start cimserver
echo \# Start cimserver
cimserver
echo
# Check cimconfig
echo \# Check cimconfig
cimconfig -l -c
echo
# Add the needed CIM authorizations
echo \# Add the needed CIM authorizations
cimauth -l | grep wbem | grep -q "root/cimv2" || cimauth -a -u wbem -n root/cimv2 -R -W
cimauth -l | grep wbem | grep -q "root/PG_InterOp" || cimauth -a -u wbem -n root/PG_InterOp -R -W
cimauth -l | grep wbem | grep -q "root/PG_Internal" || cimauth -a -u wbem -n root/PG_Internal -R -W
cimauth -l | grep wbem | grep -q "root/cimv2/npar" || cimauth -a -u wbem -n root/cimv2/npar -R -W
cimauth -l | grep wbem | grep -q "root/cimv2/vpar" || cimauth -a -u wbem -n root/cimv2/vpar -R -W
cimauth -l | grep wbem | grep -q "root/cimv2/hpvm" || cimauth -a -u wbem -n root/cimv2/hpvm -R -W
# List the CIM authorizations
echo \# List the CIM authorizations
cimauth -l
echo
ls -l /var/opt/wbem/repository
# Check if SysFaultMgt processes are running:
echo \# Check if SysFaultMgt processes are running:
ps -ef | grep sfmdb | grep -v grep
echo
# Set System Management Homepage to start on boot and add se group to authorized users:
echo \# Set System Management Homepage to start on boot and add se group to authorized users:
/opt/hpsmh/lbin/hpsmh stop
cat >/opt/hpsmh/conf.common/smhpd.xml &lt;&lt;EOF
&lt;?xml version="1.0" encoding="UTF-8"?>
&lt;system-management-homepage>
&lt;admin-group>se&lt;/admin-group>
&lt;operator-group>&lt;/operator-group>
&lt;user-group>&lt;/user-group>
&lt;allow-default-os-admin>True&lt;/allow-default-os-admin>
&lt;anonymous-access>False&lt;/anonymous-access>
&lt;localaccess-enabled>False&lt;/localaccess-enabled>
&lt;localaccess-type>Anonymous&lt;/localaccess-type>
&lt;trustmode>TrustByCert&lt;/trustmode>
&lt;xenamelist>&lt;/xenamelist>
&lt;ip-restricted-logins>False&lt;/ip-restricted-logins>
&lt;ip-restricted-include>&lt;/ip-restricted-include>
&lt;ip-restricted-exclude>&lt;/ip-restricted-exclude>
&lt;ip-binding>False&lt;/ip-binding>
&lt;ip-binding-list>&lt;/ip-binding-list>
&lt;/system-management-homepage>
EOF
chmod 444 /opt/hpsmh/conf.common/smhpd.xml
/opt/hpsmh/bin/smhstartconfig -a off -b on
/opt/hpsmh/lbin/hpsmh start
/opt/hpsmh/bin/smhstartconfig
# writing the ConfFile if needed
if [ ! -f /usr/local/etc/HPSIM_irsa.conf ]; then
echo \# Writing a fresh /usr/local/etc/HPSIM_irsa.conf file, which contains:
echo
[ ! -d /usr/local/etc ] &amp;&amp; mkdir -p -m 755 /usr/local/etc
cat > /usr/local/etc/HPSIM_irsa.conf &lt;&lt;EOF
# Configuration file is read by (if available)
#       - HPSIM-Check-RSP-readiness.sh
#       - HPSIM-Upgrade-RSP.sh
#       - HPSIM-HealthCheck.sh
#       - restart_cim_sfm.sh
#################################################
# Default location of this file is:
#       /usr/local/etc/HPSIM_irsa.conf
# but may be overruled with the '-c' argument
#
#################################################
#       Variables available in this config file
#       have default settings in each script too
#       and may be overruled via command arguments
#       (don't worry if conf file is not found...)
#       Use the '-h' for help with each script
#################################################
# The WBEM user used for HP SIM purposes
# WbemUser=wbem
WbemUser=wbem
# The mail recipients to whom an output report will
# be send - default is none
# mailusr="root"
# mailusr="root,someuser@corporation.com"
mailusr=gdhaese
# The HP SIM server FQDN (no default for this one!)
# SimServer=HPSIM_FQDN
SimServer=sim-server
# The maximum Test Delay in seconds for sending
# out a test event (only used by HPSIM-HealthCheck.sh)
# MaxTestDelay=0
MaxTestDelay=0
# The logging directory where our log files will be kept
# dlog=/var/adm/install-logs    (default)
dlog=/var/adm/install-logs
# The Ignite/UX or SD server where our HP-UX depots are kept
# IUXSERVER=FQDN
IUXSERVER=ignite-ux
# The location of the base HPSIM/IRSA depots
# baseDepo=/var/opt/ignite/depots/irsa
# without 11.11, 11.23 or 11.31 sub-depots names
baseDepo=/var/opt/ignite/depots/irsa
# The encrypted password of the WbemUser
# Variable only used by HPSIM-Check-RSP-readiness.sh script
# An easy way to produce such crypt password is with "openssl passwd -crypt", or
# HPSIM-Check-RSP-readiness.sh -p will prompt for a new password
# ENCPW="6u2CMymnCznQo"         # default password is "hpinvent" (without the double quotes)
ENCPW="6u2CMymnCznQo"
# The HP System Management Homepage Admin Group (hpsmh is default setting)
# The WbemUser will belong to this secondary group to allow access to HP SMH
HpsmhAdminGroup="hpsmh"
EOF
cat /usr/local/etc/HPSIM_irsa.conf
fi
} > /var/adm/install-logs/RSP_readiness_hpux01.con-config.scriptlog 2>&amp;1
</pre>

If you read through the config script you will notice that the `/usr/local/etc/HPSIM_irsa.conf` will be created if did not yet exists.
Furthermore, in preview mode, the two scripts `/tmp/RSP_readiness_$(hostname).install` and `/tmp/RSP_readiness_$(hostname).config`, will not be executed automatically. In installation mode these will be executed automatically. So, after running `HPSIM-Check-RSP-readiness.sh` in preview mode (without *-i* option) the choice is yours to run the two home-brew scripts. E.g. to evaluate does it make sense or not?

<pre>
$ /tmp/RSP_readiness_hpux01.install
$ cat /var/adm/install-logs/RSP_readiness_hpux01.inst-install.scriptlog
# Test 1 : HP-UX B.11.11 supported - OK
# Test 2 : Patch Level - OK
# Test 3 : Corrupt filesets found - N/A
# Test 4 : ISEEPlatform - OK
# Test 5 : Unsupported System Fault Mgt. (SFM) found - OK
# Test 6 : OpenSSL - OK
# Test 7 : WBEMservices - OK
# Test 8 : OnlineDiag - OK
# Test 9 : Stop System Mgmt Homepage
The System Management HomePage server is already stopped.
# Test 10 : Apache v2.0 - OK
# Test 11 : Apache v2.2 - N/A
# Test 12 : SysMgmtWeb - OK
# Test 13 : Start System Mgmt Homepage
The System Management HomePage server has been started successfully.
The System Management HomePage timeout monitor is currently disabled.
# Test 14 : HP SIM/IRSA Patches - OK
# Test 15 : System Fault Mgt - OK
# Test 16 : WBEMMgmtBundle - N/A
# Test 17 : Remote Support Adv Conf Collector
...
</pre>

Installation run on HP-UX 11.11

When we feel confident that no additional patches are required (that require a reboot - Test 2), and that there are no corrupt filesets on your system present (before installing new software it is better to start with a healhty system - Test 3) we may re-run the `HPSIM-Check-RSP-readiness.sh` script, but now with the *-i* option (installation mode):

<pre>
$ ./HPSIM-Check-RSP-readiness.sh -u wbem -p -m gdhaese -d ignite-ux:/var/opt/ignite/depots/irsa \
  -g se -s sim-server -i
Enter the secret password for user wbem (do not forget it!)
Password:
Verifying - Password:
  Installation Script: HPSIM-Check-RSP-readiness.sh
        Ignite Server: ignite-ux
           OS Release: 11.11
                Model: 9000/800/L2000-44
    Installation Host: hpux01
    Installation User: root
    Installation Date: 2012-02-03 @ 11:26:22
     Installation Log: /var/adm/install-logs/HPSIM-Check-RSP-readiness.2012-02-03.112622.scriptlog
 ** Running installation mode!
 ** HP-UX 11.11 support is OK
 ** Patch Level is OK
 ** Corrupt filesets - none were found - OK
 ** ISEEPlatform was not found - OK
 ** Current SysFaultMgmt version is OK
 ** OpenSSL version is OK
 ** WBEMService is OK
 ** OnlineDiag version is OK
 ** Stop the HP System Mgmt Homepage daemon
 ** hpuxwsApache release is OK
 ** hpuxws22Apache release not found - N/A
 ** System Management Homepage version is OK
 ** Start the HP System Mgmt Homepage daemon
 ** No IRSA patch bundle for HP-UX 11.11 required
 ** System Fault Management (SFM) is OK
 ** WBEMMgmtBundle is not required on 11.11 - N/A
 ** Install HP Remote Support Advanced Configuration Collector
 ** nParProvider version is OK
 ** vParProvider version is OK
 ** (Re-)Configure all CIM Providers installed on this system
 ** Running /tmp/RSP_readiness_hpux01.install...
 ** Analyzing the install log file for errors
==> ERROR:   "hpux01:/":  1 configure or unconfigure scripts failed.
==> ERROR:   More information may be found in the agent logfile using the
==> Run command "swjob -a log hpux01-1982 @ hpux01:/".
=======  02/03/12 11:53:38 MET  BEGIN configure AGENT SESSION
         (pid=5905) (jobid=hpux01-1982)
       * Agent session started for user "root@hpux01". (pid=5905)
       * Beginning Analysis Phase.
       * Target:           hpux01:/
       * Target logfile:   hpux01:/var/adm/sw/swagent.log
       * Reading source for file information.
       * Summary of Analysis Phase:
       * 21 of 21 filesets had no Errors or Warnings.
       * The Analysis Phase succeeded.
       * Beginning the Configure Execution Phase.
       * Filesets:         21
       * Configuring EMT...
       * Creating the EMT database...
ERROR:   Failed to create the EMT database.
ERROR:   Could not install the EMT database. EMT configure failed.
NOTE:    EVWEB install failed.
NOTE:    Registering of SFM Provider Module.
NOTE:    SFM Provider Module registration completed.
       * Running config clean command /usr/lbin/sw/config_clean.
       * Summary of Execution Phase:
ERROR:       Installed     SFM-CORE.EVWEB_COREPA,l=/,r=A.04.04.02
ERROR:   1 of 21 filesets had Errors.
       * 20 of 21 filesets had no Errors or Warnings.
ERROR:   The Execution Phase had errors.  See the above output for
         details.
=======  02/03/12 11:56:57 MET  END configure AGENT SESSION (pid=5905)
         (jobid=hpux01-1982)
 ** Running /tmp/RSP_readiness_hpux01.config...
 ** Analyzing the config log file for errors
 ** No errors found in config log file.
 ** There were several errors detected (see details above or in the log files)
</pre>

Hum, an error occured (*Failed to create the EMT database*), which means that the SFM database was not populated correctly (in our case because the database was probably not stopped at the moment of upgrade). This is a good oppertunity to introduce the HPSIM health check script `/usr/local/bin/HPSIM-HealthCheck.sh`. We can just run without additional parameters, because the configuration file `/usr/local/etc/HPSIM_irsa.conf` was created during the execution of script `HPSIM-Check-RSP-readiness.sh` in installation mode (option *-i*).

### The /usr/local/bin/HPSIM-HealthCheck.sh script ###

The purpose of the `HPSIM-HealthCheck.sh` script is to verify if WBEM related software components are properly configured, and to check some security related issues. Furthermore, it also checks the configuration part of HPSIM at the HP-UX managed node and report the errors, if any. It will send a test event to HPSIM server and beyond via the IRSA plug-in (at the HPSIM server) to HP back-end, but only if it finds valid WBEM/WEBES subscriptions. To verify if the test event arrived you need to login on the HPSIM console and check the events tab of system you're testing (in this particular case hpux01).

<pre>
$ /usr/local/bin/HPSIM-HealthCheck.sh
  -> Reading configuration file /usr/local/etc/HPSIM_irsa.conf
 --------------------------------------------------------------------------------
               Script: HPSIM-HealthCheck.sh
         Managed Node: hpux01
 System Serial Number: USA000000
       Executing User: root
        HP SIM Server: sim-server
         WBEM account: wbem
     Mail Destination: gdhaese
                 Date: Fri Feb  3 13:17:18 MET 2012
                  Log: /var/adm/log/HPSIM-HealthCheck.scriptlog
 --------------------------------------------------------------------------------
  ** System hpux01 runs HP-UX B.11.11                                                : [  OK  ]
  ** SysFaultMgmt (A.04.04.02.01) is properly installed                              : [  OK  ]
  ** WBEMSvcs (A.02.07.06) is properly installed                                     : [  OK  ]
  ** Older filesets may jeopardize proper working                                    : [FAILED]
 --------------------------------------------------------------------------------
  -> WARNING: Found some older filesets - please investigate (it may be OK)
openssl         /1.0.0e/A.00.09.08s.001/
SysMgmtHomepage         /A.2.2.9.2/A.2.2.9.3.1/
 --------------------------------------------------------------------------------
  ** EMS Monitors ( A.04.20.11.05 ) are enabled                                      : [  OK  ]
  ** Directory permissions of /opt/hpsmh (555)                                       : [  OK  ]
  ** Directory permissions of /etc/opt/hp/sslshare (555)                             : [  OK  ]
  ** File permissions of /stand/bootconf (644)                                       : [  OK  ]
  ** File permissions of /etc/resolv.conf (644)                                      : [  OK  ]
  ** File permissions of /var/opt/wbem/cimserver_current.conf (644)                  : [  OK  ]
  ** Check integrity of file /etc/pam.conf                                           : [  OK  ]
  ** A non-privileged user can login via ssh                                         : [  OK  ]
  ** HP System Mgmt Homepage is running                                              : [  OK  ]
  ** The "wbem" user exists and has a valid password                                 : [  OK  ]
  ** The "wbem" user is authorized to do CIM work                                    : [  OK  ]
  ** The CIMON port 5989 is in LISTEN mode                                           : [  OK  ]
  ** All CIM Providers are enabled and usable                                        : [FAILED]
 --------------------------------------------------------------------------------
  -> WARNING: SFMProviderModule                       Stopped
  ->  when "Degraded" then disbale/enable the povider, e.g.
  ->          cimprovider -d -m SFMProviderModule
  ->          cimprovider -e -m SFMProviderModule
 --------------------------------------------------------------------------------
  ** Is enable Subscriptions For Nonprivileged Users "true"                          : [  OK  ]
  ** The /var file system is still below 90% usage                                   : [  OK  ]
  ** HP SIM Server sim-server is reachable                                           : [  OK  ]
  ** The WEBES port on sim-server is accepting a SSL connection                      : [  OK  ]
  ** Valid HPSIM subscription for system hpux01                                      : [  OK  ]
  ** Valid HPWEBES subscription for system hpux01                                    : [  OK  ]
  ** Sending a test event (delay 0 seconds)                                          : [  OK  ]
 --------------------------------------------------------------------------------
  -> Login with your admin account on the HP SIM server sim-server
  -> and check if for system hpux01 a critical (type 4) event arrived
 --------------------------------------------------------------------------------
  ** List of external HP SIM/WEBES subscription for system hpux01                    : [  OK  ]
 --------------------------------------------------------------------------------
 --------------------------------------------------------------------------------
  ** List most recent events for system hpux01                                       : [  OK  ]
 --------------------------------------------------------------------------------
An error occured while executing the request
 --------------------------------------------------------------------------------
  ** Display relevant cimserver messages of today                                    : [  OK  ]
 --------------------------------------------------------------------------------
Feb  3 11:41:53 hpux01 cimserver[23202]: PGS10213: Provider (HPUXFCHBAIndicationProvider) is no 
longer serving subscription (root/cimv2 HP_General Filter@1_V1, root/cimv2 localhost/CIMListener
/EMArchiveConsumer) in namespace root/cimv2.
</pre>

Above output proofs the finding we saw during the installation (or upgrade) of System Fault Management software which failed to create the EMT database as the SFMProviderModule is not running. Also, pay attention to the section *List most recent events for system hpux01* as the output returned an error ("An error occured while executing the request"). To remediaze this we can do different things, but the easiest way it running the `/usr/local/bin/HPSIM-Upgrade-RSP.sh`

### The /usr/local/bin/HPSIM-Upgrade-RSP.sh script ###

The purpose of the `HPSIM-Upgrade-RSP.sh` script to upgrade and install missing (new) components at any time, but it can also be used to fix issues introduced by the `HPSIM-Check-RSP-readiness.sh` script as it will try to re-configure failed software (before and after any software installation). The script has also two modes (preview and installation), and a help is available (option *-h* which is very similar to the `HPSIM-Check-RSP-readiness.sh` script). However, only the option *-i* is required as we have the configuration file (`/usr/local/etc/HPSIM_irsa.conf`) in place.

The output is like:

<pre>
$ ./HPSIM-Upgrade-RSP.sh -i
 ** Reading configuration file /usr/local/etc/HPSIM_irsa.conf
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
  Installation Script: HPSIM-Upgrade-RSP.sh
        Ignite Server: ignite-ux
           OS Release: 11.11
                Model: 9000/800/L2000-44
    Installation Host: hpux01
    Installation User: root
    Installation Date: 2012-02-03 @ 15:17:47
     Installation Log: /var/adm/install-logs/HPSIM-Upgrade-RSP.2012-02-03.151747.scriptlog
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 ** Running installation mode!
 ** Before we start check the software status of installed software:
 ** All software is properly configured.
 -----------------------------------------------------------------------------------------------
  ** Bundle B9073BA (HP-UX iCOD Instant Capacity (iCAP)) was not found on this system: [ SKIP ]
  ** Bundle EventMonitoring with version A.04.20.11.05 is up-to-date                 : [  N/A ]
  ** Bundle FCProvider with version B.11.11.08 is up-to-date                         : [  N/A ]
  ** Bundle FileSysProvider with version B.11.11.01 is up-to-date                    : [  N/A ]
  ** Bundle IOTreeIndication with version B.11.11.0712 is up-to-date                 : [  N/A ]
  ** Bundle IOTreeProvider with version B.11.11.06 is up-to-date                     : [  N/A ]
  ** Bundle LVMProvider with version B.11.11.05 is up-to-date                        : [  N/A ]
  ** Bundle OnlineDiag with version B.11.11.21.07 is up-to-date                      : [  N/A ]
  ** Bundle OpenSSL with version A.00.09.08s.001 is up-to-date                       : [  N/A ]
  ** Bundle RS-ACC with version A.05.50.26.007 is up-to-date                         : [  N/A ]
  ** Bundle SCSIProvider with version B.11.11.07 is up-to-date                       : [  N/A ]
  ** Bundle SysFaultMgmt with version A.04.04.02.01 is up-to-date                    : [  N/A ]
  ** Bundle SysMgmtWeb with version A.2.2.9.3.1 is up-to-date                        : [  N/A ]
  ** Bundle UtilProvider with version A.01.08.05.02 is up-to-date                    : [  N/A ]
  ** Bundle WBEMP-LAN-00 with version B.11.11.0706 is up-to-date                     : [  N/A ]
  ** Bundle WBEMP-LAN-TEST-00 with version B.11.11.03 is up-to-date                  : [  N/A ]
  ** Bundle WBEMSvcs with version A.02.07.06 is up-to-date                           : [  N/A ]
  ** Bundle hpuxwsApache with version B.2.0.59.16 is up-to-date                      : [  N/A ]
  ** Bundle nParProvider with version B.12.02.07.03 is up-to-date                    : [  N/A ]
  ** Bundle vParProvider with version B.11.11.01.06 is up-to-date                    : [  N/A ]
  ** Bundle EMS-KRMonitor with version A.11.11.05 is up-to-date                      : [  N/A ]
  ** Bundle WBEMextras (HP WBEM Extras for HP-UX) is missing on on this system       : [  OK  ]
 ** Fri Feb  3 15:18:08 MET 2012 - Installing WBEMextras A.01.00.04 HP WBEM Extras for HP-UX
=======  02/03/12 15:18:08 MET  BEGIN swinstall SESSION
         (non-interactive) (jobid=hpux01-1991)
       * Session started for user "root@hpux01".
       * Beginning Selection
       * Target connection succeeded for "hpux01:/".
       * Source connection succeeded for
         "ignite-ux:/var/opt/ignite/depots/irsa/11.11".
       * Source:
         ignite-ux:/var/opt/ignite/depots/irsa/11.11
       * Targets:                hpux01:/
       * Software selections:
             WBEMextras.HPSIM_IRSA_scripts,r=A.01.00.04,a=S800_HPUX_11,v=GPL,fr=A.01.00.04,fa=S800_HPUX_11
             WBEMextras.Restart_cim_sfm,r=A.01.00.04,a=S800_HPUX_11,v=GPL,fr=A.01.00.04,fa=S800_HPUX_11
       * Selection succeeded.
       * Beginning Analysis and Execution
       * Session selections have been saved in the file
         "/.root/.sw/sessions/swinstall.last".
       * "hpux01:/":  There will be no attempt to mount filesystems
         that appear in the filesystem table.
       * The execution phase succeeded for "hpux01:/".
       * Analysis and Execution succeeded.
NOTE:    More information may be found in the agent logfile using the
         command "swjob -a log hpux01-1991 @ hpux01:/".
=======  02/03/12 15:19:30 MET  END swinstall SESSION (non-interactive)
         (jobid=hpux01-1991)
 ** No errors detected during installation of WBEMextras
 -----------------------------------------------------------------------------------------------
 ** Are there software components which are still in 'installed' state?
 ** All software is properly configured.
 -----------------------------------------------------------------------------------------------
 ** The active cimproviders are:
MODULE                                  STATUS
OperatingSystemModule                   OK
ComputerSystemModule                    OK
ProcessModule                           OK
IPProviderModule                        OK
HP_iCAPProviderModule                   OK
HP_GiCAPProviderModule                  OK
SDProviderModule                        OK
HPUXFCCSProviderModule                  OK
HPUXFCIndicationProviderModule          OK
HPUXFCProviderModule                    OK
FSProviderModule                        OK
HPUXIOTreeIndicationProviderModule      OK
IOTreeModule                            OK
HPUXLVMProviderModule                   OK
HPUXSCSICSProviderModule                OK
HPUXSCSIProviderModule                  OK
HP_UtilizationProviderModule            OK
HP_NParProviderModule                   OK
HP_VParProviderModule                   OK
HPUXLANProviderModule                   OK
HPUXLANCSProviderModule                 OK
HPUXLANIndicationProviderModule         OK
EMSHAProviderModule                     OK
SFMProviderModule                       OK

 ** Send a test event (simulate memory or cpu failures):
Contacting Registrar on hpux01
NAME:   /system/events/memory
DESCRIPTION:    System Memory Monitor
This resource monitors events for system memory.  Event monitoring
requests are created using the Monitoring Request Manager.  Monitoring
requests to detect changes in device status are created using the
Peripheral Status Monitor (psmmon(1m)) and Event Monitoring Service (EMS).
For more information see the monitor man page, (dm_memory(1m)).
TYPE:   /system/events/memory is a Resource Class.
There is one resource configured below /system/events/memory:
Resource Class
        /system/events/memory/8
Finding resource name associated with monitor dm_memory.
Found resource name /system/events/memory
associated with monitor dm_memory.
Creating test file /var/stm/config/tools/monitor/dm_memory.test
for monitor dm_memory.
Performing resls on resource name /system/events/memory
for monitor dm_memory to cause generation of test event.
###############################################################################################
 ** There were no errors detected.
###############################################################################################
</pre>

After, the upgrade we better re-run the `HPSIM-HealthCheck.sh` script again to verify if all is fine.

### The /usr/local/bin/cleanup_subscriptions.sh script ###

If you migrate a managed node from one HPSIM server to another and configure it correctly with the new SIM server then you will have multiple HPSIM/HPWEBES subscriptions. To cleanup old subscription we provided a script `cleanup_subscriptions.sh`.

The usage is quite simple:
<pre>
/usr/local/bin/cleanup_subscriptions.sh -h
Usage: cleanup_subscriptions.sh [-s HPSIM-Server] [-m <mail1,mail2>] [-c Config file] [-hd]
-s: The HPSIM server (IP address or FQDN).
-m: The mail recipients seperated by comma.
-c: The config file for arguments.
-h: This help message.
-d: Debug mode (safe mode)

cleanup_subscriptions.sh run without any switch will use the following default values:
-s sim-server -m root

Purpose is to delete obsolete HPSIM and WEBES subscriptions on this system.
</pre>

The option `-d` is very helpful to run it in debug mode, then you will see what it will try to do (if you run the script with `-d` argument).

### The WBEMextras (HP WBEM Extras for HP-UX) depot ###

The WBEMextras software depot contains the following scripts which we already know:

- `/usr/local/bin/HPSIM-HealthCheck.sh`
- `/usr/local/bin/HPSIM-Upgrade-RSP.sh`
- `/usr/local/bin/cleanup_subscriptions.sh`

and, one new script `/usr/local/bin/restart_cim_sfm.sh` which has a correseponding crontab entry:

<pre>
6,21,36,51 * * * * /usr/local/bin/restart_cim_sfm.sh  > /dev/null 2>&1
</pre>

The script `/usr/local/bin/restart_cim_sfm.sh` will do some checks on a regular basis of:

- is the *WbemUser* created? If not, then this HP-UX managed node cannot properly be discovered by the HPSIM server, and we assume that we did not yet run the script `HPSIM-Check-RSP-readiness.sh`
- is the *WbemUser* locked or not? If locked then unlock the user account, otherwise, the HPSIM server cannot do discovery and other inventory sweeps on the HP-UX managed node.
- is the *WbemUser* expired? If yes, then we need to reset this account.
- is the cimserver process running on the HP-UX managed node? If not, then start it up. If yes, check if the process runs longer then 1 day, then force a restart to prevent CPU bottlenecks, unnneeded memory consumption.
- Enable any disbaled cim provider agent process we encounter, unless the status is 'stopped' then we do not enable it again (as most likely we stopped it on purpose) 

From experience we know that the `restart_cim_sfm.sh` script increased the stability of the cimserver and cim provider agent processes on the HP-UX managed node a lot.

## Frequently Asked Question (FAQ) ##


*ERROR: System _ignite-ux_ is not reachable via ping from _hpux01_* <br />
 Make sure _ignite-ux_ is a valid hostname and it must be reachable, otherwise, it makes no sense to use this script. Be aware, if we install from the localhost the Ignite/ux server may be named _localhost_


## References ##
The references used in this document are listed below.

- [HP Software Depot Download site](http://www.software.hp.com)
- [HP Support Center](https://h20566.www2.hp.com/portal/site/hpsc/public/)
- [HP IRSA Configuration Guide](http://bizsupport2.austin.hp.com/bc/docs/support/SupportManual/c03052695/c03052695.pdf)
