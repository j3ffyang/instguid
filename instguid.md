```
Unix, AIX and Linux

  / /  (_)__  __ ____  __
 / /__/ / _ \/ // /\ \/ /
/____/_/_//_/\_,_/ /_/\_\

###############################################################################

AIX System Admin

AIX Tools and Util (iostat and vmstat filesed, etc.

$bootlist -m normal -o cd0 hdisk0       #change the boot order from hdisk0 to cd0.

lslpp -w /usr/bin/man	#find out its fileset.
lslpp -l bos.rte.libc	#AIX version check

lslpp -l bos.rte	# Runtime level

AIX install
	inutoc /path/inst.images	# to create toc for install
        installp -aX -d device_path X11.adt.lib X11.adt.motif bos.adt.base

	# Install fixes
	# download into /usr/sys/inst.images
	cd /usr/sys/inst.images
	gzip -d -c 510006.v1.tar.gz | tar -xvf -
	inutoc /usr/sys/inst.images
	installp -acgXd /usr/sys/inst.images bos.rte.install
	smitty update_all
	oslevel -r	# check the maintenance level

AIX security mode install TCB (trusted computer base)
        ls -le	#list if there is a plus '+' at the end of file, indicating TCB installed

manpage	man install - man page install manpage fileset
                bos.html.en_US
                bos.html.en_US.cmds
                #bos.html.en_US.nav
                bos.html.en_US.topnav

vmstat and iostat filesets      bos.acct
AIX debug fileset               bos.adt.debug

lssrc -a | grep dhcp
	install bos.net.tcpip

lsattr -El mem0		#aix mem memory checking.
lsattr -E -l sys0	# You can check the current setting of maxuproc with the command:  

chdev -l sys0 -a maxuproc='nn'	To change maxuproc, use the command: where nn is the new integer value of maxuproc.  

	# mkdev for ethernet
	mkdev -l en0

	# lsdev for ethernet card
	lsdev -C | grep en

	# cfgmgr -> Check the missing fileset/ driver
	cfgmgr
	cfgmgr: 0514-621 WARNING: The following device packages are required for
       		device support but are not currently installed.
		devices.pci.ethernet:devices.pci.1410ff01:devices.pci.86802912:
		devices.pci.pciclass.020000

socket buffer
	no -a | grep tcp
	no -a | grep sb_max
	no -o sb_max= xxx 	# change socket buffer

aix stop process
	stopsrc -s snmpd -> to stop SNMP
	comment out the snmpd line in the /etc/rc.tcpip file

samba on aix, mount to remote windows
	installp bos.cifs_fs
	mount -v cifs -n <hostname>/userid/passwd /<remote_filesystem> /local

Debug / trouble shooting
	#Dump analysis and error report
	errpt -a
	strings _ core | grep _=	# core analysis
	dbx /usr/HTTPServer/bin/httpd core
		# Available param
		where
		t
		at
		map

# mount an ISO on AIX
	mklv -y cdlv rootvg ${num_of_pps}
	dd if=/path/file.iso of=/dev/cdlv
	mount -v cdrfs /dev/cdlv /mnt/point

................................................................................
###############################################################################

AIX Monitor (data collected/ selected in System Assessment service in 2000)
	- lslpp -l output
	- errpt output
	- ps auxww

	- vmstat ( capture output every 10 sec for 15 mins for 1 day )
	Make sure the day this is captured represents an average load day.
	- iostat ( capture output every 10 sec for 15 mins for 1 day )
	Make sure that the day this is captured represents an average load day.

	These 2 lines can be placed in the crontab and then removed after the data has been collected.
	/usr/bin/vmstat 10 90 >> /tmp/vmstat.out 	#interval 10 sec for 90 secs
	/usr/bin/iostat 10 90 >> /tmp/iostat.out	#interval 10 sec for 90 secs

$vmstat <time interval> <iteration> > <filename>	 
$iostat <time interval> <iterations> > <filename>
................................................................................

Unix System Performance Tuning and Monitoring

CPU-> Mem-> Disk/ IO-> Network
................................................................................
	System Performance Tuning and Monitoring Flowchart

                       /^\
     Yes             /     \
+--------------+   /         \
| Optimize     <-< Is the sys  >
| reschedule   |   \CPU bound/
| repriortize  |     \     /
| sar & vmstat |       \ /
| time         |        | No
| tprof & ps   |        |
| nice & renice|        v
| schedtune    |       /^\               /^\
+--------------+     /     \           /     \      Yes
                   /         \  No   /         \ +------------+
               +-<Is the system>-> <Is the system| Read-ahead |
           Yes |   \MEM bound/       Disk bound? |write-behind|
               v     \     /           \    /    | I/O pacing |
+--------------+       \ /               \/      | iostat     |
| vmstat       |                     +--------+  | filemon    |
| ps           |                     | netstat|  | fileplace  |
| lsps         |                     | nfsstat|  | lslv       |
| svmon        |                     +--------+  +------------+
| rmss         |                        /^\
| vmtune       |        Yes           /     \
+--------------+   +------------+   /         \
                   | parameter  <-<Is the system>        
                   | nfs tuning |  Network bound   
                   | add mem    |     \     /      
                   | reschedule |       \ /
                   +------------+
................................................................................
###############################################################################

Memory Performance Tuning and Monitoring (Mem Perf)
sar	(ref: AIX Certification Guide: Perf Tuning and Monitor. Page 52)
	syntax: sar <interval> <times>	 
	$sar -P ALL 2 1		#CPU utilization/ performance
	$sar 1 10 = $sar -u 1 10 #capture cpu util 10 times per 1 second.
	$sar -a 1 10	#how many times per second several of the sys file access routunes had been called.
	$sar -c 1 10	#system calls.
	$sar -d = $iostat
	$sar -q	1 10	#queue statistics
	$sar -r	1 10	#paging statistics
	$sar -v 1 10	#status of the process, kernel- thread, i- node, and file table.
	$sar -y 1 10	#status of tty dev.

svmon	to list of the top mem users. Individual perspective.
	-P	display mem usage statistics for processes pid1 ... pidN.

emstat	performance issue when migrate to a Power PC srv from an old POWER src.

ps au	a snapshot available in time to look at a processes' average use of mem.

vmstat	(ref: AIX Certification Guide: Perf Tuning and Monitor. Page 61). Average perspective.
	$vmstat -f	#how many forks since system startup
	$vmstat hdisk1
	Occasional small numbers of page pi and page po are normal.

	kthr	kernel thread state changed per second over the sampling interval.
	-r 	# of kernel threads placed in run queue. value shoule be < 5
	-b	# of kernel threads place in wait queue. near to 0 (zero)
	Memory Info about the usage of virtual and real mem. A Page is 4096 bytes.
	-avm	active virtual pages => $lsps -a (how to know page file/ pagefile size on AIX)
	-fre	size of free list
	Page	Info about page faults and paging activity.
		These are averaged over the interval and given in units per sec.
	Ports Listening	-an
	-re	page input/ output list
	-pi	pages paged in from paging space
	-po 	pages paged out to paging space
	-fr	pages freed (page replacement)
	-sr	pages scanned by page- replacement algorithm
	-cy	clock cycles by page-replacement algorithm
	Faults	Trap and interrupt rate averages per second over the sampling interval.
	-in	dev interrupts
	-sy	system calls
	-cs	kernel thread contect swtiches
	CPU	breakdown of percentage usage of CPU time.
	-us	user time
	-sy	system time
	-id	cpu idle time
	-wa	cpu cycles to determine that the curent process is waiting and there is pending disk input/ output.
	Disk xfer

$vmstat -s	write to the standard output the contents of the sum structure.

ps / perf/ performance
	column	value
	C	Recent used CPU time for process
	TIME	Total CPU time used by process since it started
	%CPU

	C column
	#ps -ef | sort +3 -r | head -n 5
	+3: 3rd column/ C column. -r: reversed. head -n 5: display first 5 lines

	TIME column
	#ps -e | head -n 1; ps -e| grep -v "TIME|0:"|sort +2b -3 -n -r|head -n 10

	CPU column
	#ps auxwww | head -n 5
	#ps gu | head -n1; ps gu|egrep -v "CPU|kproc" |sort +2b -3 -n -r |head -n 5
	ps -axf -> list detailed cmd.

	RSS column
	#ps av |sort +6 -r |head -n 5

	%MEM column
	#ps au | head -n 1; ps au |egrep -v "RSS"|sort +3 -r |head -n 5
	#ps gv|head -n 1;ps gv|egrep -v "RSS" | sort +6b -7 -n -r |head -n 5

	ps haxo 'size' | (tr '\n' +; echo 0) | bc

ps to check memory usage
	# rsz= physical mem size
	ps -e -o rsz,pid,cmd --sort rsz

	# vsz= virtual mem size
	ps -e -o vsz,pid,cmd --sort vsz

	# awk '{ t... }' = sum of all number
	ps -e -o rsz | awk '{ t += $1 } END { print t }'


SMP	symmetrical multiprocessor

................................................................................

Disk Performance Tuning and Monitoring (Disk Perf)
	if large background job interfering with interactive response time, activate I/O pacing.
	a small # of files are being read over and over again-> consider whether additional real mem would allow those
	files to be buffered more effectively.
	if iostat cmd indicates I/O activity is not distributed among the sys disk drives, and the util of one or more disk
	drives is often 40- 50% or more, -> consider reorg fs.
	if workload access patterm is random, -> adding disks and distributing the randomlu accessed files across more drives.

Disk- Physical volume level report
	#filemon -o /tmp/filemonLF.out -O pv
Disk- Virtual mem level report
	#filemon -o /tmp/filemonLF.out -O vm
................................................................................

General Recommendations on Disk, I/O performance. (IO)
Logical volume org for highest perf.
	allocate hot LVs to different PVs to reduce disk contention.
	spread hot LVs across multiple PVs so that parallel access is possible.
	place the hottest LVs in the center of PVs, the moderate LVs in the
	middle of PVs, and the coldest LVs on edges of PVs.
	mirroring can improve perf for read- intensive applications but, as
	writes need to be performed several times, can impact the perf of other
	applications.
	make the LV contiguous to reduce access time.
	set inter- policy to max. this will spread each logical volume across
	as many physical volumes as possible, allowing reads and writes to be
	shared among several physical vols.
	place frequently used logical vol close together to reduce the seek time.
	set write verify to NO.
Logical vol striping
File system relate perf issues.
	create an additional log logical vol to separate the log og the most
	active file system from the default log. this will increase parallel
	resource usage.  

An lslv usage scenario: determine if hot file systems are better located on a
	physical drive or spread across multiple physocal drives.  

lslv	LVM perf analysis using lslv. this cmd uses mainly cpu time.

lspv	determine which disk or set of disk is experiencing contention on a SCSI bus.

filemon usage scenario:
	determine if hot files are local or remote.
	determine if paging space dominates disk util.
	look for heavy physical vol util. determine if the type of drive or SCSI adapter causing a bottleneck.

trcstop	if the filemon cmd is invoked, run trcstop to stop the cmd os that the filemon reports can be generated.

iostat	-d	disk utilization report
	-t	TTY and CPU usage.
	%iowait + %tm_act metrics provided by the iostat report is used to initially determine if a sys is I/O (IO) bound.
fileplace usage scenario:
	determine if the application perf a lot of synchronous file IO.
	look for file fragmentation. determine if the hot files are heavily frgmented.
Paging space related disk perf issue
	never add more than one paging space on the same physical vol.
	reorg or add paging space on the same physical vol.
................................................................................

Network Performance Tuning and Monitoring (NW Perf)

Adapter transmit and receive queue tuning
	#lsattr -El ent0
	#ifconfig en0 detach
	#chdev -l ent0 -a tx_que_size=128
	#ifconfig en0 up
	#netstat -v
	* 2 parameters should be checked (page 155, CertGuide AIX Perf Sys Tune)
	- Max Packets on S/W Transmit Queue. This is the max
	# of outgoing packets ever queued to the sw xanmit queue. An indication
	  of an inadequate queue size is if max
	# xansnuts queued equals the current queue size tx_que_size. This
	  indicates that the queue was full at some point.
	- S/W Transmit Queue Overflow. The
	# of outgoing packets that have overflowed the sw xansmit que. A value
	  other than zero indicates that the same actions
	# needed if the Max Packets on S/W Xansmit Que reaches the tx_que_size
  	  should be taken. The xansmit queue size has to be increased.

###############################################################################

AIX/ aix/ tcpip tuning
 You must be root to change the values. Use the "no -a" command to list all
settings and the command below to set the values:

no -o sack=1
no -o rfc1323=1
no -o tcp_sendspace=524176
no -o tcp_recvspace=524176
no -o sb_max=1048352
no -o tcp_mssdflt=1448
- or -
no -o tcp_pmtu_discover=1

Be aware that the changes are lost after a reboot. Add the command to an init
script like tcp.local or use the -p option of the no command on AIX 5.2
systems (if you did not migrate from AIX 5.1).

###############################################################################
###############################################################################

ENV env environment

Unix id command/ cmd
	0		root. ALL permission,
	1- 100		some permission
	101+-> 65535	no special permission. Normal user

Unix SUID/ suid, SGID/ sgid
	-r-Sr--r-x 1	root	system 	234423 	Oct 20 15:29 back_shell	#Have group permission w/o exe
	drwxr-sr-x 2 	root	system	123	Oct 30 10:20 mydir	#have group perm w/ execute

Unix sticky	-r-xr-xr-t
	chmod a+t <dir>	#generic unix. will not work in AIX
	chmod +t <dir>	#AIX cmd

Unix device
	tty- hard wired terminal
	pts- pseudo terminal

Linux Profile/profile / Environment Setting (env)
	/etc/bashrc 	=> system wide aliases and functions; 	
	/etc/profile	=> system wide environment stuff and startup programs 	
	In AIX, they're /etc/environment and /etc/profile

	/etc/bashrc  and ~/.bashrc			# bashrc will be read first and always read. Both login shell and nonlogin shell will read.
	/etc/profile and ~/.bash_profile		# Only login shell read

	/etc/skel/ stores sys files for being copied to new created user home
	ie. /etc/skel/.profile to set EDITOR

	$HOME/.bashrc contains user aliases and functions; 	
	$HOME/.bash_profile contains user environment stuff and startup programs

Linux .profile / PROFILE/ Profile sample
	# .profile

	USERNAME="root"
	PATH=$PATH:/usr/local/bin
	BASH_ENV=$HOME/.bashrc
	HOSTNAME=Greatwall

	export USERNAME BASH_ENV PATH

	# In AIX, $HOSTNAME is the fully qualified hostname
	HOSTNAME=`hostname -s`
	PS1='$LOGNAME@$HOSTNAME $PWD \$ '

	# SuSE konsole
	export PS1=$PS1"\[\e]0;\H:\w\a\]"

	stty erase ^?	# backspace redefine

Disable / disable pc speaker / audio
	echo "set bell-style none" >> /etc/inputrc

$HOME/.inputrc contains key bindings and other bits.

Linux Shell Change/ Selection
	$chsh		#change shell
	$echo $SHELL	#check current used shell.

###############################################################################
###############################################################################

Linux command/ cmd usage / linux command

script	# record scripting in shell	# recording replay
	script -a filename	# append

	script -t 2> tutorial.timing -a tutorial.session # record w/ timing
	scriptreplay tutorial.timing tutorial.session	 # replay

cvs / CVS
	# login
	cvs -d:pserver:anonymous@cvs.sourceforge.net:/cvsroot/open1x login
	# retrieve the source
	cvs -d:pserver:anonymous@cvs.sourceforge.net:/cvsroot/open1x get xsupplicant

Linux editor	=> kate / vi / jedit

Command line ftp through ftp | http
	wget -t3 -c -r -l 10 http://somewhere
	# -t= retry, -c=continue, -r=recursive, -l=depth

	wget --no-directories --passive -m ftp://sunsite.ch/redhat-updates/7.2/en/os/i386/*.rpm

Linux common cmd/ Linux command:  
	wall-> send msg to all clients
	last		/var/log/wtmp
	lastlog		/var/log/lastlog
	who		/var/log/utmp
	lastb		/var/log/btmp	bad attemp login
	ac		/var/log/wtmp	connect time
	dump-utmp	Converts the raw data from utmp or wtmp into ascii.
	ftpwho		display all active ftp users
	ftpcount	current # of users logged in to the sys. and the max # allowed.
	ftpshut		shutdown ftp server. /etc/shutmsg created.
			remove /etc/shutmsg to restart ftp server

	top-> s1: 1 second interval. M: sort by mem. P: sort by CPU. T: time and fo: get help.

	md5sum /bin/ps-> checksum rpm's md5sum

	file /bin/ps	# unix file cmd/ file command. ie. file /etc/security/lastlog to tell file type.
	kill -HUP <pid> = daemon restart

Linux text file converted between win/dos to unix
	From Unix to DOS:
	recode lat1..ibmpc file.txt
	From DOS to Unix:
	recode ibmpc..lat1 file.txt

Unix fuser cmd/command	fuser (objective: find out which appl owns port
	AIX
	fuser /fs/file			-> list pid
	fuser -u /file/system		-> list pid/ uidA
	fuser -kxuc /file/system	-> terminate all process using file.

	Linux
	fuser -kimuv /file/system	-> terminate all process using file.

	/sbin/fuser 22/tcp -> output is	# port/ service/ pid
	22/tcp:                907  7643

	fuser -n tcp|udp -v <port>[,<remote address>[,<remote port>]
	fuser -n tcp -v 22

	ps -xa | grep '907' -> outout is
	 907 ?        S      0:01 /usr/local/sbin/sshd2
	7710 pts/0    S      0:00 grep 907

	ps -el		-> this gives correct start time only if the process was started before 24hrs.
			For all the process which has elapsed 24 hrs, it just displays the number of days.

Linux lsof 	Track network connection
	lsof <file_system_name>	# find processes blocking umount
	lsof /path/file		# find users of a specific open file
	lsof -i:<port#>		# find particular nw connect
	lsof -iTCP		# show only TCP (works the same for UDP)
		lsof -iTCP@aaa.bbb.ccc:ftp-data
	lsof -i@192.168.1.1	# show connections to a specific host
	lsof -i:sunrpc		lsof -i:telnet
	lsof -i | grep LISTEN	# same to ESTABLISHED
	lsof -p <pid>		# list all appl running over such daemon
	lsof -c <first_char_of_cmd_that_interests_u>
				# list all appl running
	lsof -c syslog
	lsof +d /var/log	# associates open files with their processes. +D is recursive.
	lsof -F 		# format the output of list
	lsof -u <login> / <uid>	# files open by a specific login
	lsof -N			# list open NFS files

lsof advanced usage
	lsof -a -u userid -i @1.1.1.1
		# Using -a to combine search terms, like says, "show me everything running as daniel connected to 1.1.1.1"

	kill -HUP `lsof -t -c sshd`
		# Using -t and -c options to HUP processes
	kill -9 `lsof -t -u userid`
		# to kill all processes owned by userid
	lsof +L1
		# to show all open files that have a link count less than 1.
		# indicative of a cracker trying to hide something


Linux log rotate
	$less /etc/logrotate.conf
	$man logrotate
	ie. Weekly rotate 51 #how many weeks are there in one year.
	ie. Monthly rotate 12 #

Linux date/ cal (display how many days of today in this year.)
	cal -j
	date +%j

Linux Calendar/ calendar	cal mm yyyy

Linux print configurator
	/usr/sbin/printtool

	# printer on ubuntu 11.10
	system-config-printer > Add > Internet Printing Protocol > ipp://9.123.142.17:631/ipp > Forward > Generic (recommended) > PCL 6/PCL XL (recommended) > Generic PCL 6/PCL XL Printer Foomatmatic/pxlcolor [en](recommended)

Unix ls command
	ls -ltr 	# will list the most recent updated files at the bottom
	ls -la		# to list all detail w/ user+ group
	ls -ln		# to list all detail w/ userid + groupid

	ls -i /usr 	# list inode
	ls -dl /usr	# list directories


Command line internet
	text internet	$lynx	#(shift+o -> options)
			$links

Zip/zip/unzip	jar -x -> to unzip
		jar -t -> to list the content.
		unzip file.zip
bz2		tar xjf *.tar.bz2	or bunzip2 *.bz2

rar		unrar x file.rar	# extract rar file

iso		mkisofs -o $IMAGE.iso $ORIG_FILE


###############################################################################
###############################################################################


Linux system management/ system mgmt

cpu temperature / thermal
       cat /proc/acpi/thermal_zone/THMO/
       <setting not supported>
       cooling mode:   critical
       <polling disabled>
       state:                   ok
       temperature:             60 C
       critical (S5):           255 C


logging > strace -ff -F -tt -v -o /tmp/passwd-trace.log -s 102400 passwd "user" # increase tracing level upon passwd command

hostname set in RedHat
	edit /etc/sysconfig/network
	hostname -s	-> short host
	hostname -d	-> list domain

Linux control center	gnomecc-> gnome		kontrol-panel-> kde

Linux desktop switch	exec gnome-session
			exec startkde

Linux IPC/ipc/ipcs/ipcrm 	# related to db2
	Symptom is db2instl can't db2start and db2stop force-> SQL10003 and SQLSTATE=57011
	-> Virtual resource is not enough
	su - db2inst1-> kill -9 -1-> ipcs | grep db2-> ipcs-> ipcrm shm xxxxx-> ipcs
	kill -9 -1 (minus one) -> kill shm (shared mem)

Linux cpu/ mem check & partition setting
	cat /proc/cpuinfo
	cat /proc/meminfo
	cat /proc/partitions

Linux mem memory by pid
	ps -o pmem <pid>

Linux Memory (mem)
	$memprof	#in gui

	add below in /etc/lilo.conf to recognize the added mem
	append="mem=???MB"

hard drive / hard disk / hdd performance
	# Get the current status of hard drive
	hdparm -Tt /dev/hda

	# Set up
	hdparm -c1 -d1 -m16 /dev/hda
	-c1		# turn on 32bit i/o on pci bus
	-d1		# enable dma
	-d1 -X66	# enable ultradma transters
	-m16		# multicount on/ off
	-t		# test timing bufferred disk reads
	-T		# test timing buffer- cache reads

	-A1		# Enables the auto-readahead feature of the drive
	-a64		# tells the drive how far to read ahead.
	-X69		# UDMA 5 ATA 100
	-M254		# Full speed

file fragment
	filefrag <file>	# list frangmented extension's quantity

Linux mouseconfig/ mouse config
	/usr/sbin/mouseconfig
	/etc/sysconfig/mouse

SuSE/ suse mouse configuration in cmnline/ command line
	SaX2 /dev/input/mice -p PS/2
	SaX2 /dev/mouse -p PS/2

Linux xwindow mouse setting
	xset mouse 8 2	# mouse pointer acceleration

Linux swap file
	#dd if=/dev/zero of=/swapfile bs=1024 count=65536
	mkswap /swapfile 65536
	sync
	swapon /swapfile

	swap should be equal to twice your computer's RAM, or 32MB, whichever
	amount is larger, but no larger than 2GB.

	# To create a swap partition
	mkswap /dev/hdb2
	swapon /dev/hdb2
	add the following in /etc/fstab
		/dev/hdb2	swap	swap	defaults	0 0
	# to verify -> cat /proc/swaps

	# To create a swap file
	dd if=/dev/zero of=/swapfile bs=1024 count=65536
	mkswap /swapfile
	swapon /swapfile
	add the following in /etc/fstab
		/swapfile	swap	swap	default		0 0

	# to remove a swap space
	swapoff /dev/hdb2
	remove its entry in /etc/fstab
	then umount, remove the filesystem or file

linux disk quota
	create a partition, such /dev/hda7 = /home
	edit /etc/fstab -> replace "defaults" entry with "usrquota"
	mount -o remount /home
	touch /home/aquota.user
	quotacheck -c /home
	quotaon /home
	edquota user_A

disk raid
	fdisk /dev/hda -> add 4 news partitions ->
	use "t" to convert filesystem id to "fd", which is raid auto/ RAID AUTO

	touch /etc/raidtab
		***********************************************
		raiddev /dev/md0
		        raid-level              5
		        nr-raid-disks           4
		        chunk-size              32
		        persistent-superblock   1
		        parity-algorithm        left-symmetric
		        device          /dev//hda9
		        raid-disk       0
		        device          /dev//hda10
		        raid-disk       1
		        device          /dev//hda11
		        raid-disk       2
		        device          /dev//hda12
		        raid-disk       3
		***********************************************

	mkraid /dev/md0		# start array
	raidstart /dev/md0	# manually activate
	watch cat /proc/mdstat	# watch the array building progress
	mke2fs -j -b 4096 -R stride=8 /dev/md0
		# format the raid fs. 4096x8=32 in chunk-size in /etc/raidtab
	mount /dev/md0 /<mount_point>
	lsraid -A -a /dev/md0	# display info on state of raid
	add entry in /etc/fstab	# if you want to mount it when boot
	cat /proc/mdstat	# check the array building
	raidsetfaulty /dev/md0 /dev/hda11	# simulate faulty on hda11
	/var/log/messages and output of /proc/mdstat
	raidstart /dev/md0, then raidhotadd /dev/md0 /dev/hda11


Linux Disk Usage/ disk usage
	df -h -T -l
	df -T /dev/hdaX

disk mgmt disk management > convert ext2 to ext3
	/sbin/tune2fs -j /dev/vg0/pool

lvm / logical volume manager
	vgscan			# initilize LVM configuration
	fdisk /dev/hda		# create a new partition for vg
	pvcreate /dev/hdaX	# initialize LVM partitions
	pvdisplay		# display

	vgcreate <VG_NAME> /dev/hdaX	# create a volume group using the default 4mb extent size
	vgdisplay

	lvcreate -L 40M -n <LV_NAME> <VG_NAME>	# create a small logical volume
	lvdisplay /dev/<VG_NAME>/<LV_NAME>
	mke2fs -j /dev/<VG_NAME>/<LV_NAME>	# format that filesystem
	mount /dev/<VG_NAME>/<LV_NAME>	/<MOUNT_POINT>

	# change lv status, usage
	lvscan

	# extend vol group
	vgextend <VG_NAME> /dev/hdaN		# extend VG to another partition

lvm resize (umount lvm first)
	e2fsck -f /dev/vg0/LogVol00 # fsck on the filesystem
	lvextend -L +100M /dev/vg0/lvXXX
	lvreduce --size -100M /dev/vg0/lvX (!! data on reduced part will get lost !!)

	e2fsck -f /dev/vg0/LogVol00 # Do another fsck on the
	resize2fs /dev/vg0/LogVol00 # resize the fs to nn GB; to commit

lvm remove /LVM remove
	remove any /etc/fstab entries you might have setup
	umount /dev/<VG>/<LV>
	lvremove /dev/<VG>/<LV>
	vgchange -an <VG>	# deactivate the VG

vg remove
	vgremove <VG>

change vg status
	vgchange -a y

AIX filesystem / file system
	# To determine if jfs2 being used
	lsfs -v jfs2	# The cmd returns NO output if it finds standard fs.

Linux Core Dump analysis / core dump / coredump
	core | grep _= 	# core analysis	# aix
	strings core-> find which cmd causes the crash

	gdb /filesystem/cmd core-> find which lib causes the crash

Linux Kernel Source (kernel src rpm)
	$cd /mnt/cdrom/RPMS	(linux installation CD)
	$rpm -ivh kernelsrc.rpm -> install rpm
	$cd /usr/src/linux	(you will find this file system)

	# Creatae an emergency bootdisk
	mkbootdisk -device /dev/fd0 2.4.7-10

	# find kernel level
	uname -a
	SPident		# on SuSE to find out the FixPack level

Linux Kernel Recompiling/ compile -> /usr/src/linux-version/  
	make menuconfig (make xconfig, make config)
	make dep - checks dependencies
	make clean - cleans up old .o, .a files, and so forth		 
	make bzImage - compile.  
		     - create kernel /usr/src/linux-version/arch/i386/boot
	make modules; make modules_install

	or
	make menuconfig	 
	make dep clean bzImage modules modules_install

	then
	cp arch/i386/boot/bzImage /boot/vmlinuz-2.2.15-x.0-xxxx
	cp System.map /boot/System.map-2.2.15-x.0-xxxx
	vi /etc/lilo.conf -> add the image
	lilo -v -v

Linux updatedb/ slocate		/etc/cron.daily/slocate.cron
	# initialize the db -> slocate -c -u (create + update)
	# to find the file  -> slocate myfile

Linux lib*.so lib.so
	updatedb-> locate libntdll.so-> put the output into /etc/ld.so.conf
	-> ldconfig -v

Linux module probe
	lsmod 		-> list all the loaded module
	rmmod ipchains	-> remove module ipchains
	modprobe 3c509  -> load module 3c509
	insmod eepro100	-> install module
		or edit /etc/module.conf -> alias eth0 eepro100
	The network drivers locate /lib/modules/2.4.7-10/kernel/drivers/net
	# after modifying /etc/modules.conf, run to refresh
	depmod -a

Linux RPM/ rpm mgmt/ management signature
	# rpm package signature check
	gpg --import /mnt/cdrom/RPM-GPG-KEY
	rpm -K <pachage> -> MD5 checksum
	rpm --checksig passwd-0.64.1-1.i386.rpm
	passwd-0.64.1-1.i386 md5 gpg OK

	# verify rpm's integrity
	rpm -V `rpm -qa` or rpm -V `rpm -qa | grep cmd`
		net-tools-> telnet rpm
		procps-> /bin/ps rpm
	rpm --verify `rpm -qa`
	for j in `rpm -qa`; do-> echo $j-> rpm --verify $j-> done

	# verify rpm with ignoring file attibute. List unsat
	rpm -Va --nofiles

	# rpm remove
	rpm -e --allmatches packages
	rpm remove using --allmatches parameter
	rpm -e --allmatches libstdc++-4.0.0-1 --nodeps

	# receive the error
	error: %preun(VMwareWorkstation-4.5.2-8848) scriptlet failed, exit status 1
	# solution
	rpm -e --nodeps --nopreun xyz

	!# rpm db recovery #
	rm -fr /var/lib/rpm/__db*
	db_verify /var/lib/rpm/Packages
	/usr/bin/rpmdb --rebuilddb
	/usr/lib/rpmdb/i386-redhat-linux/redhat ->
		all rpm db location. backup them up before changing

	rpm -ivh ftp://path/file.rpm	# Install from ftp site
	rpm --aid			# install all dependent rpm automatically when detecting required
					# must rpmdb-*.rpm

	rpm -qf /filesys/cmd 		# check 'cmd' owned by which RPM
	rpm -qa | grep cmd 		# if cmd rpm is installed
	rpm -qa gpg-pubkey		# list the imported gpg pubkey
	rpm -q --requires mozilla-M18 	# query the requirement of one rpm
	rpm -qil xxx			# where the rpm is installed and package information.
	rpm -qil <rpm> --changelog	# list all changelog
	rpm -qi <rpm>			# last installed, size, all info.
	rpm -qi <rpm> --scripts		# preinstall, postinstall scripts
	rpm -qp <rpm> --requires	# dependency list
		      --provides	# capability provided by package
		      --changelog	# changelog

rpm build
	# building from a source RPM (SRPM) to create a *.spec
	# usually in /usr/src/redhat/
	rpm -i somepackage-1.0-1.src.rpm

	# build the RPM with *.spec, then find RPM in RPMS/i386
	cd /*/SPECS
	rpmbuild --bb somepackage.spec

	# building form a source archive
	rpmbuild -tb somepackage-1.0.tar.gz

yum	# update kernel
	yum -y update kernel

yum	disable rhn > edit /etc/yum/pluginconf.d/rhnplugin.conf > enable=0

yum clean dbcache

yum repo build
	yum install createrepo
	cp -var $AL_RPM $ACTUAL_REPO_DIR
	createrepo $ACTUAL_REPO_DIR

yum mount an ISO
	mkdir -p /mnt/iso/{1,2,3}
	mount -o loop file.iso /mnt/iso/1
	cd /mnt/iso
	createrepo .
	yum clean all
	vi /etc/yum.repos.d/iso.repo
		baseurl=file:///mnt/iso
		enabled=1

yum suse repo build on rhel
	On RHEL, Install & configure vsftpd. Place all SuSE dvd under /var/ftp/pub/sles103_repo/suse/
	createrepo /var/ftp/pub/sles103_repo/suse/

yum clean all
yum -d10 check-update

Inittab change		/etc/inittab-> id:x:initdefault:  
			(x=kde w/ network multi users w/o graphic)
	/sbin/init q

Linux chkconfig-> change runlevel.
	#chkconfig --list -> list all on- services. ie. chkconfig --level 2345 ssh on -> start ssh in runlevel 2,3,4,5
	#chkconfig telnet on -> to enable telnet service

	chkconfig --add <service_name> 	# add to check service
	cp <service_name> /etc/rc.d/init.d/ -> cd /etc/rc.d/init.d/
	chmod u+x <service_name>
	<service_name> must contains following 2 lines
		#!/bin/bash
		# chkconfig: 3 56 50
		# description: nothing
	chkconfig --del <service_name> 	# delete checked service

Linux at / AT
	$ at 14:00	-> man at. schedule a cmd by at.

History, key in		$cat /root/.bash_history
	HISTFILESIZE environmental variable. To increase it,
	put "export HISTFILESIZE=xxxx" in your .bashrc file.

	in ubuntu, place "export HISTSIZE=5000" into ~/.bashrc

Restore/ recovery boot manager/ bootmgr	$lilo -u /dev/hda

Linux undelete and delete (Recover)	# egrep -200 'string1.+string2' /dev/hda3 > /mnt/dos/barrie
					strings /mnt/dos/barrie | string2

Linux crontab sample file
	SHELL=/bin/sh
	# mail any output to `paul', no matter whose crontab this is
	MAILTO=paul
	#
	# run five minutes after midnight, every day
	5 0 * * *       $HOME/bin/daily.job >> $HOME/tmp/out 2>&1
	# run at 2:15pm on the first of every month -- output mailed to paul
	15 14 1 * *     $HOME/bin/monthly
	# run at 10 pm on weekdays, annoy Joe
	0 22 * * 1-5   mail -s "It's 10pm" joe%Joe,%%Where are your kids?%
	23 0-23/2 * * * echo "run 23 minutes after midnight, 2am, 4am ..., everyday"
	5 4 * * sun     echo "run at 5 after 4 every sunday"

	min 	hour 	day 	month 	weekday
	0-59	0-23	1-31	1-12	0-6

manual page/ manpage/ man page
	man logrotate | col -b > /tmp/logrotate.man
	# remove !@#$%^.  comparing w/ man logrotate > /tmp/logrotate.man

uuid UUID > 	blkid

Linux system management/ system mgmt

###############################################################################
###############################################################################

Linux network management / network mgmt / nw mgmt

TCP/IP layer / tcpip reference model

 	Application layer		-> web
	client and server programs

	Transport layer			-> program-program msg delivery
	tcp and udp protocols and service ports

	Internet/Network layer		-> source-destination computer msg delivery
	ip packets, ip addr and icmp msg

	subnet layer
	cable, wire, microwave, radio

    IP vs UDP vs TCP (ip vs udp vs tcp)
	IP -> datagram, data forward
	UDP -> parallel w/ TCP, on top of IP, packed in IP. No connection.
	TCP -> same as UDP. Connection required. Verify connection always. Busy traffic. ie. Telnet. Photo call.

ip
	show
	ip addr show
	ip link show

	enable a NIC
	ip link set eth0 up

	set ip addr
	ip address add 192.168.0.77 dev eth0 -> ifconfig eth0 192.168.0.77
	ip addr add 192.168.0.77/24 broadcast 192.168.0.255 dev eth0 -> ifconfig eth0 192.168.0.77 netmask 255.255.255.0 broadcast 192.168.0.255

	delete an ip
	ip addr del 192.168.0.77/24 dev eth0

	add alias interface
	ip addr add 10.0.0.1/8 dev eth0 label eth0:1 -> ifconfig eth0:1 10.0.0.1/8

	arp protocol
	arp -i eth0 -s 192.168.0.1 00:11:22:33:44:55
	ip neigh add 192.168.0.1 lladdr 00:11:22:33:44:55 nud permanent dev eth0

	switch arp resolution off on one device
	ip link set dev eth0 arp off -> ifconfig -arp eth0


add/ delete routing table
	route add default gw <ip>
	route delete default gw <ip>
	route -n	# -n = number, IP

	route add -net 192.168.100.0 netmask 255.255.255.0 gw 172.16.0.1

# add route policy
	vi /etc/sysconfig/network-scripts/ifup-post
	ip route del default
	ip route add 9.0.0.0/8 via 9.115.78.1 dev br1
	ip route add 172.16.0.0/16 via 172.16.27.1 dev br0

netstat interface	$netstat -in -t
			#netstat -a | grep pts
			#netstat -tap -> tell you who owns the processes.

    * LISTEN?The socket is listening for incoming connection.
    * ESTABLISHED?The socket has an established connection.
    * SYN_SENT?The socket is actively attempting to establish a connection.
    * SYN_RECV?A connection request has been received from the network.
    * TIME_WAIT?The socket is waiting after close to handle packets still in the network.
    * FIN_WAIT1?The socket is closed, and the connection is shutting down.
    * FIN_WAIT2?The connection is closed and the socket is waiting for a shutdown from the remote end.
    * CLOSE_WAIT?The remote end has shut down, and it is waiting for the socket to close.
    * CLOSED?The socket is not being used.

	netstat -an -> the output is
	Active Internet connections (servers and established)
	Proto Recv-Q Send-Q Local Address           Foreign Address         State
	tcp        0    268 24.101.233.38:22        199.246.40.54:13694     ESTABLISHED
	tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
	udp        0      0 0.0.0.0:67              0.0.0.0:*

	netstat -a -p -A inet

	# Search for open TCP network ports
	netstat -apNlt

	# Search for open UDP network ports
	netstat -apNlu

Linux wu-ftp/ FTP anonymous turnoff
	Turn off anonymous ftp access (if you have to have ftp at all) by editing /etc/ftpaccess
	You will see a line :

	class   all   real,guest,anonymous  *
	Just remove the words guest and anonymous from the line.

	another way to permanently remove anonymous ftp
	userdel ftp
	rpm -qa | grep anonftp

tcp wrapper
	Restricting access to local users with TCP wrappers

	The tcpd program is configured using two files: /etc/hosts.allow and
	/etc/hosts.deny. These files have lines of the form:

	daemon_list : client_list [ : shell_command ]

	Access is granted or denied in the following order. The search stops at the first match:

    * Access is granted when a match is found in /etc/hosts.allow
    * Access is denied when a match is found in /etc/hosts.deny
    * Access is granted if nothing matches

	For example, to allow telnet access only to our internal network, we
	start by setting policy (reject all connections with a source other
	than localhost) in /etc/hosts.deny:

	in.telnetd: ALL EXCEPT LOCAL

Linux Telnet Connection Refused.
	Uncomment or Add in.telnetd:ALL:ALLOW in /etc/hosts.allow
	eg. enable some service like telnet, ssh, etc-> add below lines in hosts.allow
	sshd:ALL
	telnetd:ALL
	ftp:ALL
	in.telnetd:ALL:ALLOW
	ALL:192.168.0/255.0

	Then make sure below is added in hosts.deny
	ALL:ALL

Linux Telnet Login/login text. message of the day
	$vi /etc/issue -> text added in this file will be displayed when system starts on screen
	$vi /etc/issue.net -> usually copied from above file. The text added will display on telnet.
	/etc/motd -> msg of the day.	 

Linux NFS mount config		
	exportfs -ra	# apply the /etc/exports change
	vi /etc/exports, add ->		/mnt/cdrom	192.168.1.1(ro,no_root_squash)
	# for AIX, 	 add -> 	/mount/path	*(insecure)

	# on server, need to restart nfs service
	# On SuSE, /etc/init.d/nfsserver restart
	# on client, need to start portmap

Linux Mount/mount mgmt
	force to umount	$fuser -c /cdrom		(in AIX)
			$kill -9 xxx <- /cdrom pid

			$fuser -kimuv /mnt/cdrom	(in RedHat)

			$fuser -k /cdrom/oracle8i	(in Sun)
			$fuser -c /cdrom/oracle8i -> this might cause cdrom unavailable until reboot machine.
	#mount -o remount,rw /usr -> remount.  

	showmount <hostname> or <ip_address>
	showmount -e <hostname>		# list /etc/exports

Linux connect to Windows as Samba client/ samba client/ smbclient

	smbclient -L windows_IP -U windows_username
	mount -t smbfs //windows_IP/shared_drv /mnt/samba -o username=win_usr

Linux Share (Samba/ samba/ smb) -> for windows user
	vi /etc/samba/smb.conf

[global]
    workgroup = MYGROUP
    server string = Samba Server Version %v
    log file = /var/log/samba/log.%m
    max log size = 50
    security = user
    passdb = tdbsam
[homes]
    comment = Home Directories
    browseable =no
    writable = yes

	../smbpasswd user_id
	$/usr/bin/smb restart

	c:\net use \\jyang\tech /user:db2inst1	(in windows)

	Encrypt doc	/usr/doc/samba-2.0.6/docs/textdocs/ENCRYPTION.txt

/etc/resolv.conf -> this file is auto generated after dhcpcd eth1. copied on apr 11. 2002
	domain mtmk.phub.net.cable.rogers.com
	nameserver 24.153.22.195
	nameserver 24.153.23.66
	search mtmk.phub.net.cable.rogers.com

network resolvconf resolv.conf on Ubuntu ubuntu
	nmcli dev list iface eth0 | grep IP4.DNS
	sudo dpkg-reconfigure resolvconf or sudo ln -sf ../run/resolvconf/resolv.conf /etc/resolv.conf

ngrep - packet monitor
	ngrep port 22 and src host <ssh_client_ip> and dst host <ssh_server_ip>
	 ngrep -q -t -wi "<string_to_search>" port 22

vlan
	vconfig add eth0 100
	ip link set dev eth0.100 up
	ethtool eth0.100
	ip addr add 10.10.0.10/24 dev eth0.100
	ip add show dev eth0.100

vxlan
	1. Create vxlan device
	  # ip li add vxlan0 type vxlan id 42 group 239.1.1.1 dev eth1

	This creates a new device (vxlan0). The device uses the
	the multicast group 239.1.1.1 over eth1 to handle packets where
	no entry is in the forwarding table.

	2. Delete vxlan device
	  # ip link delete vxlan0

	3. Show vxlan info
	  # ip -d link show vxlan0

	It is possible to create, destroy and display the vxlan
	forwarding table using the new bridge command.

	1. Create forwarding table entry
	  # bridge fdb add to 00:17:42:8a:b4:05 dst 192.19.0.2 dev vxlan0

	2. Delete forwarding table entry
	  # bridge fdb delete 00:17:42:8a:b4:05 dev vxlan0

	3. Show forwarding table
	  # bridge fdb show dev vxlan0

disable multicast_snooping on neutron's physical box when running Neutron neutron in VM
	echo "0" > /sys/devices/virtual/net/br1/bridge/multicast_snooping

tailf virsh dumpxml guest_id | grep console
	tailf `virsh dumpxml 170 | grep console | cut -d'=' -f2 | cut -d"'" -f2 | awk 'NR==1{print $1}'`

create bridging/ tap devices with tunctl and openvpn
	openvpn --mktun --dev tap0

On all compute nodes, run:
	brctl addbr br.4090
	ip link add vxlan4090 type vxlan id 4090 group 239.1.1.1 dev eth0
	ip link set vxlan4090 up
	ip link set br.4090 up
	brctl addif br.4090 vxlan4090
Then the VMs using br.4090 on all compute nodes can connect each other.

add bridge (scenario = bridge & bond are missing in xml when kvm vm crashes)
	brctl addbr br.550
	ifconfig br.550 up
	vconfig add bond0.550
	brctl addif br.550 bond0.550
	ip link set bond0.550 up

add bridge
	brctl addbr br0
	brctl stop br0 off
	brctl addif br0 eth0	!!! you're going to lose the connection !!!
	ifconfig eth0 down
	ifconfig eth0 0.0.0.0 up
	ifconfig br0 10.0.3.120 up
	Then modify /etc/sysconfig/network-scripts/ifcfg-eth0 & br0

###############################################################################
###############################################################################

network debug
	symptom:
	1. storage-1/ 2 host boxes are both working.
	2. storage-2 can ping storage-1 and all kernel service VMs on storage-1.
	3. storage-2 can NOT ping any VMs on storage-2

	bonding setting
	eth0+ eth2 = bond0
	eth1+ eth3 = bond1 -> br0

--------
[root@storage-2 ~]# brctl show all
bridge name	bridge id		STP enabled	interfaces
br0		8000.5cf3fce203da	no		bond1
							vnet0
							vnet1
							vnet2
							vnet3
							vnet4
							vnet5
--------

--------
	eth3 (in bond1) is active

[root@storage-2 ~]# cat /proc/net/bonding/bond1
Ethernet Channel Bonding Driver: v3.6.0 (September 26, 2009)

Bonding Mode: fault-tolerance (active-backup)
Primary Slave: None
Currently Active Slave: eth3
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 1
Permanent HW addr: 5c:f3:fc:e2:03:da
Slave queue ID: 0

Slave Interface: eth3
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 5c:f3:fc:3b:5b:7a
Slave queue ID: 0
--------

--------
assign an IP to br0 then use br0 to ping target VM on storage-2
[root@storage-2 ~]# ifconfig br0
br0       Link encap:Ethernet  HWaddr 5C:F3:FC:E2:03:DA  
          inet addr:172.30.11.100  Bcast:172.30.11.255  Mask:255.255.255.0
	  ...

[root@storage-2 ~]# ping -I br0 zookeeper-2
PING zookeeper-2 (172.30.11.21) from 172.30.11.100 br0: 56(84) bytes of data.
^C
--------

--------
	check eth3 connectivity
[root@storage-2 ~]# ethtool eth3
Settings for eth3:
	Supported ports: [ TP ]
	Supported link modes:   10baseT/Half 10baseT/Full
	                        100baseT/Half 100baseT/Full
	                        1000baseT/Full
	Supports auto-negotiation: Yes
	Advertised link modes:  10baseT/Half 10baseT/Full
	                        100baseT/Half 100baseT/Full
	                        1000baseT/Full
	Advertised pause frame use: No
	Advertised auto-negotiation: Yes
	Speed: 1000Mb/s
	Duplex: Full
	Port: Twisted Pair
	PHYAD: 1
	Transceiver: internal
	Auto-negotiation: on
	MDI-X: Unknown
	Supports Wake-on: g
	Wake-on: g
	Link detected: yes
--------

--------
	activate eth1 in bond1		ifenslave -c bond1 eth1
	dettach eth3 in bond1		ifenslave -d bond1 eth3
	bind eth1/ eth3 in bond1	ifenslave bond1 eth1 eth3

[root@storage-2 ~]# cat /proc/net/bonding/bond1
Ethernet Channel Bonding Driver: v3.6.0 (September 26, 2009)

Bonding Mode: fault-tolerance (active-backup)
Primary Slave: None
Currently Active Slave: eth1
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 1
Permanent HW addr: 5c:f3:fc:e2:03:da
Slave queue ID: 0
--------

conclusion: eth3 doesn't have correct link even it's connected

ethtool
	find device driver and firmware info
	ethtool -i eth0

	find factory- default MAC addr
	ethtool -P eth0



###############################################################################
###############################################################################

Linux daemon management/ service management/ server management

linux named / dns / bind
	vi /etc/named.conf -> add the following

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
zone "10.168.192.in-addr.arps" in {
        type master;
        file "192.168.10.zone";
};

zone "gimlet.co.uk" in {
        type master;
        file "gimlet.co.uk.zone";
};
vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv

	touch /var/named/192.168.10.zone
	ln -s /var/named/192.168.10.zone /var/named/gimlet.co.uk.zone

	vi /var/named/192.168.10.zone ->
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
$TTL 1W
@               IN SOA          gimlet.co.uk.   root.gimlet.co.uk. (
                                42              ; serial (d. adams)
                                2D              ; refresh
                                4H              ; retry
                                6W              ; expiry
                                1W )            ; minimum

                IN NS           gimlet.co.uk.
1               IN PTR          localhost.
wclinux         IN      A       192.168.10.17
db2linux        IN      A       192.168.10.18
vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv


Linux securing e-mail
	#mailq	-> check mail queue
	#sendmail -q -> send mail pending in queue.
	mail -f ~/mbox	# read the saved mail in mailbox

	/etc/sendmail.cf -> comment out the line containing: O Daemon Port Options=Port=smtp...

linux qmail	how to hide domain name
	echo linuxaid.com.cn > /var/qmail/control/defaulthost
	chmod 644 /var/qmail/control/defaulthost

sendmail configuration
	install rpms of sendmail, sendmail-cf, m4, procmail
	vi /etc/mail/sendmail.mc -> Comment out the line below by prepending it w/ "dnl ", like so:
	dnl DAEMON_OPTIONS(`Port=smtp,Addr=127.0.0.1, Name=MTA')

	cp /etc/mail/sendmail.cf /etc/mail/sendmail.cf.orig
	cp /etc/mail/sendmail.mc /etc/mail/sendmail.mc.orig

	m4 /etc/mail/sendmail.mc > /etc/mail/sendmail.cf

sendmail debug -> sendmail -d0 < /dev/null

Linux mail command/ mail cmd sendmail cmd
	mail -s subject -c cclist -b bcclist toaddr < somefile

	send an attachment in sendmail
	uuencode /path/file | sendmail -s "subject" user@host.org

Linux sendmail server setup procedure.
	vi /etc/sendmail.cf-> 	Cwlocalhost torolab.ibm.com
				Djheaventemple.torolab.ibm.com
				DS -> maybe null. If this server sits in DMZ, this must be set for forward
	*****-> Comment out line starting: O DaemonPortOptions=Port=smtp,Addr=127.0.0.1, Name=MTA

	vi /etc/mail/access-> add torolab.ibm.com RELAY
	cd /etc/mail vi mailertable > add torolab.ibm.com smtp:mail.torolab.ibm.com > make
	-> service sendmail restart
	cd /etc/xinetd.d-> vi ipop3-> edit disable= no-> service xinetd restart

	vi /var/named/named.heaventemple.torolab.ibm.com->
		$TTL    86400
		@       IN      SOA     heaventemple.torolab.ibm.com. root.heaventemple.torolab.ibm.com.  (
		                                      1997022700 ; Serial
		                                      28800      ; Refresh
		                                      14400      ; Retry
		                                      3600000    ; Expire
		                                      86400 )    ; Minimum
		        IN      NS      heaventemple.torolab.ibm.com.
		        IN      MX 10   heaventemple.torolab.ibm.com.
		fw      IN      A       heaventemple.torolab.ibm.com

		1       IN      PTR     localhost.
	-> service named restart

--------------------------------------------------------------------------------
	telnet smtp 25
	Mail server manual test procedure
	>>> HELO sun.tuc.noao.edu
	250 noao.edu Hello sun.tuc.noao.edu., pleased to meet you

	>>> MAIL From:<rstevens@sun.tuc.noao.edu>
	250 <rstevens@sun.tuc.noao.edu>... Sender ok

	>>> RCPT To:<rstevens@noao.edu>
	250 <rstevens@noao.edu>... Recipient ok

	>>> DATA
	354 Enter mail, end with a period '.'  on a line by itself

	>>> .
	250 Mail accepted

	>>> QUIT
	221 noao.edu delivering mail
--------------------------------------------------------------------------------

postfix	check verion
	postconf mail_version


postfix A Quick- Start Procedure
	edit /etc/postfix/main.cf -> myhostname = penguinsecurity.net
	myorigin = $hostname
	mydestination = $myhostname, localhost.$mydomain, $mydomain
	uncomment home_mailbox = Maildir/ # if using IMAP

	postqueue -p	-> check mails in q
	postalias aliases	-> aliases
	postmap access		-> access
	postconf alias_maps	-> locate alias maps

	postfix reload | start | stop

postfix rpm configuration. switch MTA/ mta (using source code, don't need this)
	redhat-switch-mail
	alternatives --set mta /usr/sbin/sendmail.postfix
	install from rpm -> redhat-switchmail-nox
	postconf	-> list all set param
	postconf -d	-> list default param
	postconf -n	-> list diff one

postfix anti- spam
	Anti-UCE
	/etc/postfix/sample-smtpd.cf
	smtpd_recipient_limit	-> default = 250
	smtpd_recipient_restrictions
		check_recipient_access = hash:/etc/postfix/access
		create /etc/postfix/access -> refer man 5 access
		run postmap hash:/etc/postfix/access
	smtpd_client_restrictions
		reject_maps_rbl
		reject_unknown_client
	maps_rbl_domains -> refer to blackholes.mail-abuse.org


squirrelmail configuration
	unzip the zip file in /PATH/squirrelmail-version/
	PATH=/opt
	ln -s /PATH/squirrelmail $apache_docroot/webmail
	cd /PATH/squirrelmail/config
	perl conf.pl
		change all localhost to IP

	vi /PATH/squirrelmail/config/config.php	# default NOT need to change
	default_sub_of_inbox=true	# for courier
	default_folder_prefix=		<- nothing
	trash_folder=INBOX.Trash
	sent_folder=INBOX.Sent
	draft_folder=INBOX.Drafts

	# security / Security
	mkdir attach
	chown -R nobody.nobody data + attach
	# make sure data+ attach are outside of www tree

	# php.ini may be locate /usr/local/php/lib
	vi php.ini -> register_global=Off -> security issue
		   -> file_uploads = On   -> send email out

	# upload file size/ attachment size
	edit $PHP/lib/php.ini ->memory_limit (larger than other 2 params)
				post_max_size
				upload_max_filesize
	check httpd.conf ->	LimitRequestBody	20480000

courier-imap
	su - non-root before compile
	rpm -Uvh openssl-devel
	./configure --prefix=/usr/local/courier-imap<version> \
		--without-ipv6 --enable-unicode \
		--with-redhat --with-ssl
	# default path=/usr/lib/courier-imap
	make; make install; make install-configure

	# run this in each user who need mail
	su - thatuser -> cd ~
	$courier-imap/bin/maildirmake Maildir	# create maildir
	su - root -> change the default login to nologin for all mail users.

	$courier-imap/libexec/imapd.rc start | stop

	autostart courier-imap
	ln -s $courier-imap/libexec/imapd.rc /etc/init.d/courier-imapd
	add
	# chkconfig: - 81 30
	# description: courier-imapd
	then
	chkconfig --level 345 courier-imapd on

courier-imap	-> virtual account
	# convert OS passwd to imap. must run each time when new user comes in.
	touch /etc/userdb -> chmod 700 chown root.root
	$courier-imap/sbin/pw2userdb > /etc/userdb	#convertpasswd
	$courier-imap/sbin/makeuserdb			#compile
	chmod 600 /etc/userdb

	courier-authdaemon restart
	courier-imap restart


spamassassin / spam assassin
	export LANG=en_US
	perl -MCPAN -e shell
	install Mail::SpamAssassin


pop3 / pop / dovecot
	yum install dovecot
	edit /etc/dovecot.conf >

	# Protocols we want to be serving:
	# imap imaps pop3 pop3s
	protocols = imap pop3

	/etc/init.d/dovecot restart

	telnet localhost 110
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
+OK dovecot ready.

# telnet localhost 143
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
* OK dovecot ready.


zlib	http://www.gzip.org/zlib
	http://www.devside.net/web/server/linux/zlib
	make clean -> ./configure --prefix=PATH -> make -> make install
	or ./configure -s -> make -> make test -> make install
	cp zutil.* /usr/include
	add PATH/lib in /etc/ld.so.conf -> ldconfig

openssl	# make sure zlib is install and added in libpath, run ldconfig
	./config --prefix=/usr/local/openssl shared zlib-dynamic
	./config -t
	make -> make test -> make install
	add /usr/local/openssl/lib to /etc/ld.so.conf -> ldconfig
	openssl version

apache / httpd
	./configure --prefix=/usr/local/apache2052 \
		--enable-mods-shared=most \
		--enable-rewrite --enable-spelling \
		--enable-ssl \
		--with-z=/usr/lib \
		--with-ssl=/usr/local/openssl
		--enable-so
	make ; make install

	# enable ssl
	mkdir /usr/local/apache2/conf/ssl.crt + ssl.key
	openssl req -new -out server.csr
	openssl rsa < privkey.pem > server.key
	openssl x509 -in server.csr -out server.crt -req -signkey server.key -days 365
	mv server.crt /usr/local/apache2/conf/ssl.crt
	mv server.key /usr/local/apache2/conf/ssl.key

php	pre-requisite	-> flex, bison, libjpeg-devel, gd-devel, zlib-devel, libxml2
	./configure --prefix=/usr/local/php \
	--with-apxs2=/usr/local/apache2/bin/apxs \
	--with-openssl=/usr/local/openssl \
	--with-mysql=/usr/local/mysql \
	--with-gd=/usr \
	--with-jpeg-dir=/usr/lib \
	--with-zlib=/usr \
	--enable-mbstring
	--with-pdflib
	--with-tiff-dir=/usr/lib
	--with-png-dir=/usr/lib		(require libpng-devel, libtiff-devel)
	make; make install

	$source/php.ini-recommended -> /usr/local/php/lib/php.ini

	configure with apache -> edit httpd.conf
	LoadModule php4_module modules/libphp4.so
	DirectoryIndex index.html index.html.var index.php
	AddType application/x-httpd-php	.php
	AddType application/x-httpd-php-source	.phps

	# enable mbstring
	uncomment ;extension=php_mbstring.dll

	# php logging / log level	http://ca.php.net/errorfunc
	# reduce loglevel in php.ini	# ~ = not
	error_reporting = E_ALL & ~E_NOTICE & ~E_STRICT

gd	http://www.boutell.com/gd/
	install libpng-*+dev, libjpeg*+dev, freetype*+dev
	./configure --with-png=/usr/lib \
		--with-freetype=/usr/lib \
		--with-jpeg=/usr/lib

jpeg	ftp://ftp.uu.net/graphics/jpeg/
	or install from rpm of both libjpeg and libjpeg-devel
	default install ./configure -> install in /usr/lib
	ldconfig -> load lib

mysql
	download binary package -> don't need compile -> place to /usr/local
	-> ln -s full_name mysql -> cd support-files -> cp my-medium.cnf $mysql/data/my.cnf
	-> make sure client/ server's socket's path is correct & same -> socket=/usr/local/mysql/data

	cd mysql -> scripts/mysql_install_db
	chown -R root .
	chown -R mysql data
	chgrp -R mysql .
	bin/mysqld_safe --user=mysql &

	mysqladmin -uroot -p shutdown

	# refresh the security control
	mysqladmin reload -uuser -ppassword

	# mysql log
	$LOG=$MYSQL_HOME/data/$HOSTNAME.err

	http://www.yolinux.com/TUTORIALS/LinuxTutorialMySQL.html
	https://penguinsecurity.net/txt/mysql.txt
	# contains more info on compilation

	don't forget installing php-mysql rpm

	# configure mysql
	mysql -uuser -p
	use mysql;
	update user set password=password('xyz');
	delete from user where user='';

	# After 4.1. Passwd hashing changed.
	UPDATE mysql.user SET Password = OLD_PASSWORD('newpwd') ->
	-> WHERE Host = 'some_host' AND User = 'some_user';
	FLUSH PRIVILEGES;
	http://dev.mysql.com/doc/mysql/en/Old_client.html
	http://dev.mysql.com/doc/mysql/en/Password_hashing.html


vsftpd
	modify default anonymous path in vsftpd.conf
	mkdir -p /var/ftp/pub/upload; chown 777 /var/ftp/pub/upload

jakarta-tomcat	-> cd $CATALINA_HOME/conf -> vi tomcat-users.xml
	-> <user username="tomcat" password="password" roles="admin,manager"/>
	-> restart tomcat
	put CATALINA_OPTS=-Dfile.encoding=ISO-8859-1

opencms	-> read install.html -> in mysql, grant all on *.* to <user>;
	-> use mysql; -> update user set host='fqhostname' where host='%';
	-> flush privileges;

gallery
	tar -xzvf -> mv gallery $web -> cd $web/gallery/ -> touch config.php .htaccess
	-> chmod 0777 config.php .htaccess -> chmod 0755 setup -> mkdir albums
	-> chmod 0777 albums -> http://hostname/setup/index.php -> ... -> ./secure.sh

gallery upgrade
- split source by date
- backup
- erase duplicated
- remove backup, recreate newer but smaller backup
- sort for upload and keep less than 36 items in each folder
- find . -type f -name "*.jpg" -exec mogrify -resize 800 {} \;
- tar -czvf foo.tar.gz bar
- upload tarball
- remove resized folder and backup
- clean up compact flash



Linux linuxconf	http://www.solucorp.qc.ca/
	linuxconf --text	force to launch linuxconf in text mode


dhcp client 	$dhcpcd -h hostname -D -H eth0 	#bind with hostname when getting dhcp ip.
		$dhcpcd -k	#to restart dhcpcd
		$dhcpcd -k eth1	#stop dhcpcd on eth1
		dhclient ethX

----------------------------------------------------------------
# dhclient.conf
#
# Configuration file for ISC dhclient (see 'man dhclient.conf')
#
interface "eth0" {
send host-name "summerpalace.domain.com";
}

send dhcp-lease-time 3600;
prepend domain-name-servers 127.0.0.1;
request subnet-mask, broadcast-address, time-offset, routers, domain-name,
domain-name-servers,
host-name;
require subnet-mask, domain-name-servers;
----------------------------------------------------------------

dhcp srv/ dhcpd config		/etc/dhcpd.config as below (RedHat)

	Sample /etc/dhcpd.conf on TP755CD
	# Put this file in /etc
	# Touch /var/db/dhcpd.leases
	# According to man dhcpd.lease

	# Modified 12/29/02. Requested by dhcpd v3.0
	ddns-update-style ad-hoc;

	default-lease-time 86400;
	max-lease-time 604800;
	option subnet-mask 255.255.255.0;
	option broadcast-address 192.168.1.255;

	# Modify ip if routers address change.
	option routers 192.168.1.1;
	# option domain-name-servers 204.101.251.1, 204.101.251.2, 24.153.22.195, 24.153.23.66;
	option domain-name-servers 24.153.22.195, 24.153.23.66;
	#option domain-name "penguinsecurity.net";
        	subnet 192.168.1.0 netmask 255.255.255.0 {
        	range 192.168.1.11 192.168.1.30;
        }

Linux/Unix socks service/sockd.
	Adv: transparent after initial setup, good logging, very secure
	Disadv: doesn't work well / udp. doesn't work at all w/ icmp. client needs dns/DNS. slower than NAT*
	www.socks.nec.com
	www.inet.no/dante

	sample /etc/sockd.conf
		logoutput: syslog
		internal: 192.168.5.1 port = 1080
		external: 8.4.113.5
		method: username none
		user.privileged: root
		user.notprivileged: nobody
		user.libwrap: nobody
		connecttimeout: 30
		iotimeout: 86400
		client pass {
		        from: 192.168.5.0/24 to: 0.0.0.0/0
		        log: connect
		}
		block {
		        from: 0.0.0.0/0 to: 127.0.0.0/8
		        log: connect error
		}
		pass {
		        from: 192.168.5.0/24 to: 0.0.0.0/0
		        protocol: tcp udp

	sockify application
		in /etc/socks.conf
		route {
			from: 0.0.0.0/0 to: 192.168.1.1/24 via: direct
		}
		route {
			from: 0.0.0.0/0 to: 0.0.0.0/0 via: 192.168.1.1 port=1080
			protocol: udp tcp
			proxyprotocol: socks_v4 socks_v5
			method: none
		}
	eg.
		#socksify telnet www.ibm.com 80
		#socksify lynx
	to socksify all application that use shared lib:
		#export LD_PRELOAD="libdsocks.so"
		#telnet www.ibm.com 80

Linux apache http/proxy.
	Adv: proxy does DNS lookup. based on user, server and client ip, can do transparent caching.
	Disadv: only works for specific protocols. slower than NAT.  
		/etc/http/logs
	www.apache.org- apache
	www.tis.com- TIS firewall
	squid.nlanr.net- Squid

	sample /etc/httpd/conf/httpd.conf
		Listen 192.168.1.1:8080
		LoadModule proxy_module		modules/libproxy.so
		AddModule mod_proxy.c
		<IfModule mod_proxy.c>
		ProxyRequests On
		<Directory proxy:*>
			Order deny, allow
			Deby from all
			Allow from 192.168.0.0/24
		</Directory>
		CacheRoot "/var/cache/httpd"
		CacheSize 5
		CacheGcInterval 4
		CacheMaxExpire 24
		CacheLastModifiedFactor 0.1
		CacheDefaultExpire 1
		NoCache a_domain.com another_domain.edu xxx.com
		</IfModule>

squid3 squid proxy sample /etc/squid3/squid.conf
	acl localnet src 10.0.0.0/24
	http_access allow localnet
	http_port 0.0.0.0:3128


Linux apache/ httpd/ http server test, configtest
	# limit upload file size at 10M
	LimitRequestBody 10240000

	In Unix, $IBMHTTPServer/bin/apachectl configtest

	In Micro$oft Windows
		$IBMHTTPServer\apache -t
		$IBMHTTPServer\apache -help to get help list

	# resolve ip in http log
	logresolve [ -s filename ] [ -c ] <access_log> access_log.new
	logresolve -c <ip_list.txt> host_resolved.txt
	or in httpd.conf
	HostnameLookups [ on | off ]

	user authentication
	./htpasswd -c /usr/local/apache2/passwords user_id -> generate passwords file containing userid:encrypted_passwd
	vi .htaccess ->
		AuthName "Secret Documents"
		AuthType Basic
		AuthUserFile /usr/local/apache2/passwords
		require valid-user
	edit httpd.conf -> add
		<Directory "/path_to_secure">
		AllowOverride AuthConfig

apache / ihs / ibm http server / performance / perf
	http://httpd.apache.org/docs/mod/core.html#startservers
	http://httpd.apache.org/docs/misc/perf-tuning.html
	# ibm http server parameter setting
	http://publib.boulder.ibm.com/infocenter/wsphelp/index.jsp?topic=/com.ibm.websphere.nd.doc/info/ae/ae/rprf_webserverparameters.html
	# ibm http server caching
	http://www-306.ibm.com/software/webservers/httpservers/doc/v136/ibm/9attune.htm

apache / html redirect / forward url
	 <meta http-equiv="Refresh" content="0;URL=http://wherever_you_want">

apache / virtualhost / redirect
         NameVirtualHost *:80
         <virtualhost *:80>
         ServerName spirit125.watson.ibm.com
         # if your current config has RewriteRules in the base server config,
         # move them into this vhost
         </virtualhost>

         <virtualhost *:80>
         ServerName smallblue4.watson.ibm.com
         Redirect / http://spirit125.watson.ibm.com/smallblue
         </virtualhost>

         Or append to httpd.conf:

         RewriteEngine on
         RewriteCond %{HTTP_HOST} =smallblue4.watson.ibm.com
         RewriteRule ^/(.*) http://spirit125.watson.ibm.com/smallblue$1

apache / php redirect / php forward url
	<?PHP $URL="http://the_destination_url";
	header("Location:$URL");
	?>

apache / httpd.conf
	debug all virtual host configuration
	/usr/local/apache2/bin/httpd -S

apache / rewriterule
	# force all transaction through SSL.
        RewriteEngine   On
        RewriteRule     ^/webmail$      https://www.penguinsecurity.net/webmail [R,L]

	# redirect a URL
	RewriteEngine	On
	RewriteRule	^/$	http://somewhereelse.org/path/somefile

	# convert to https/ ssl
	RewriteRule	^/(.*)	https://%PSERVER_NAME}/$1	[R,L]

	rewrite module flags ->
	http://httpd.apache.org/docs/1.3/mod/mod_rewrite.html
	R = redirect
	L = last

	rewrite guide
	http://httpd.apache.org/docs/2.0/misc/rewriteguide.html

LDAP / OpenLDAP / ldap

	command line
	ldapsearch -vx -H ldaps://bluepages.ibm.com -b "ou=bluepages,o=ibm.com" "(email=who@us.ibm.com)" -L

	strace -e file ldapsearch -vx -H ldaps://bluepages.ibm.com -b "ou=bluepages,o=ibm.com" "(email=who@us.ibm.com)" -L

	Configuring Apahce/IHS to Authenticate against Enterprise Directory

	Base System:
	RedHat 7 with patches
	Apache and Apache-devel
	apache-devel-1.3.14-3
	apache-1.3.14-3
	Openldap
	openldap-1.2.11-15
	openldap-devel-1.2.11-15
	openldap-clients-1.2.11-15

	Needed:
		1. mod_auth_ldap source code - Install to /tmp/
		2. cd to the dir in /tmp where the mod_auth_ldap source was expanded to
		3. execute: /usr/sbin/apxs -lldap -llber -lresolv -a -i -c mod_auth_ldap.c
		4. Edit httpd.conf to make sure the following lines were added at the end of the respective sections:
			LoadModule ldap_auth_module modules/mod_auth_ldap.so
			AddModule mod_auth_ldap.c
		5. Execute from command line or put in startup script: stunnel -d 127.0.0.1:636 -r bluepages.ibm.com:636 -c
		Now you have an ssl tunnel between 127.0.0.1 port 636 and bluepages.ibm.com port 636
		6. Protect the Locations that you want similiar to the following:

	AllowOverride None
	Options None
	Satisfy all
	AuthName Blueorg
	AuthType basic
	LDAP_Server 127.0.0.1 #LDAP Server to talk to
	LDAP_Port 636 #Port to use, default ssl port
	Base_DN "ou=bluepages,o=ibm.com" #Base DN of LDAP tree to use
	UID_Attr mail # attribute to match username filed from 401 challenge to
	Require valid-user


Linux snmpd / snmp
	snmpwalk -v 1 localhost somerandomstring system

Linux Time Syncronize in a group server
	add timed daemon into /etc/rc.d/rc.local on each system and the machine starts  the daemon
	with -M option will be treated as master time sources by others.  

time / ntp client tcp/udp:123
	place following cmd into crontab -e
	0 2 * * * /usr/sbin/ntpdate -s -b -p 8 -u 129.132.2.21

      -b Force the time to be stepped using the settimeofday() system call,
	 rather than slewed (default) using the adjtime() system call. This
	 option should be used when called from a startup file at boot time.

      -p samples Specify the number of samples to be acquired from each
	 server as the integer samples, with values from 1 to 8 inclusive.
	 The default is 4.

      -s Divert logging output from the standard output (default) to the
	 system syslog facility. This is designed primarily for convenience
	 of cron scripts.

      -u Direct ntpdate to use an unprivileged port or outgoing packets. 	         
	 This is most useful when behind a firewall that blocks incoming
	 traffic to privileged ports, and you want to synchronise with hosts
	 beyond the firewall. Note that the -d option always uses unprivileged
	 ports.

	# http://portal.suse.com/sdb/en/2002/02/xntp.html
	ntpdate ntp1.ptb.de

svscan is started from init. add following entry in /etc/inittab
	SV:123456:respawn:/command/svscanboot
	# all daemons in /service/* will be started by this.
	# this daemons in /service/ is symbolink created.

ntp server
      # Look at the Startup Script in /etc/rc.d/init.d/ntpd

    start() {
            # Adjust time to make life easy for ntpd
            if [ -f /etc/ntp/step-tickers ]; then
                 echo -n $"Synchronizing with time server: "
                 /usr/sbin/ntpdate -s -b -p 8 -u \
                 `/bin/sed -e 's/#.*//' /etc/ntp/step-tickers`
                  success
                 echo
            fi
            # Start daemons.
            echo -n $"Starting $prog: "
            daemon ntpd
            RETVAL=$?
            echo
            [ $RETVAL -eq 0 ] && touch /var/lock/subsys/ntpd
            return $RETVAL
    }

      # Insert swisstime.ethz.ch or more NTP Servers to /etc/ntp/step-tickers

    129.132.2.21

      # Edit the configuration file /etc/ntp.conf

    server 127.127.1.0  # local clock
    server 129.132.2.21 # swisstime.ethz.ch (stratum 1)
    driftfile /etc/ntp/drift
    multicastclient     # listen on default 224.0.1.1
    broadcastdelay 0.008

timezone / time zone
	cd /usr/share/zoneinfo/Americas -> cp Montreal /etc/localtime
	/usr/bin/rdate -s time.nist.gov -> hwclock --systohc


ddclient	http://www.aei.ca/~pmatulis/pub/dyndns.html

	steps to set up ddclient

	   1. edit the ddclient configuration file
	   2. test the client
	   3. decide on launching strategy

	1. edit the ddclient configuration file

	The configuration file for ddclient is /etc/ddclient.conf.

	syslog=yes                              # log update msgs to syslog
	mail=root                               # mail all msgs to root
	mail-failure=root                       # mail failed update msgs to root
	pid=/var/run/ddclient.pid               # record PID in file.

	use=web
	web=checkip.dyndns.org
	login=pmatulis                                  # default login
	password=********                               # default password
	backupmx=yes	                                # host is primary MX?
	wildcard=yes	                                # add wildcard CNAME?

	server=members.dyndns.org,      \
	protocol=dyndns2                \
	pmatulis.dyndns.org

	2. test the client

	Invoking ddclient in this way will show us some of the values it is aware of:
	$ ddclient -daemon=0 -query

	Let us run the client for the first time. I have decided to turn on as many
	messages as possible for this testing phase:
	$ ddclient -daemon=0 -debug -verbose -noquiet

	checking /var/log/messages should determine whether everything worked or not.
	Feb 9 22:52:52 veritas ddclient[24339]: SUCCESS: updating pmatulis.dyndns.org: good: IP address set to 216.209.130.107

vmware / VMWare -> err msg: cannot rm /var/run/vmware/<user> when boot
	+++ rc.sysinit  2003-11-17 19:08:09.000000000 +0100
	@@ -647,7 +647,8 @@
        if [ -d "$afile" ]; then
           case "$afile" in
                */news|*/mon)   ;;
-               */sudo|*/vmware)        rm -f $afile/*/* ;;
+               */sudo) rm -f $afile/*/* ;;
+               */vmware)       rm -rf $afile/*/* ;;
                *)              rm -f $afile/* ;;
           esac
        else

=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-


###############################################################################
###############################################################################

Linux Security/ linux security

disable selinux
	permissive	echo 0 > /selinux/enforce
	permanent	@ /etc/selinux/config
			change SELINUX=enforcing to SELINUX=permissive or disabled
	check status	getenforce	sestatus

Unix hack/ hacking scenario. Do as root
	cp /usr/bin/ksh /tmp/ls
	chmod u+s /tmp/ls
	su - non_root_user
	/tmp/ls		-> grant access as root

openssl
	# generate an ssl cert signing request (csr) -> send csr file to CA
	openssl req -new -out filename.csr -keyout privkey.pem
	openssl rsa -in foo.key >> cert.pem

	openssl rsa -in privkey.pem -out server.key

	# create a self- signed ssl cert
	openssl req -new -x509 -days 365 -out filename.crt -keyout privkey.pem
	openssl x509 -in server.csr -out server.crt -req -signkey server.key -days 365
	## mv server.crt $apache/conf/ssl.crt
	## mv server.key $apache/conf/ssl.key

	# set up a CA / ca
	ssl/misc/CA.pl -newca
	ssl/misc/CA.pl -newewq	# create a cert
	ssl/misc/CA.pl -sign	# sign by CA

	# convert ssl cert from der to pem format
	openssl x509 -inform der -in filename -out filename.pem

        # create digests of a file, which can be used to verify that a file
        # hasnot been tampered with:
        $ echo "test file" > foo.txt
        $ openssl dgst -md5 foo.txt
        MD5(foo.txt)= b05403212c66bdc8ccc597fedf6cd5fe

        $ openssl dgst -sha1 foo.txt
        SHA1(foo.txt)= 0181d93fee60b818e3f92e470ea97a2aff4ca56a

        =-=-
        # encrypt
        $ openssl enc -aes-256-cbc -salt -in foo.txt -out foo.enc

        enter aes-256-cbc encryption password:

        Verifying - enter aes-256-cbc encryption password:

        $ file foo.enc

        foo.enc: data

        $ cat foo.enc

        Salted__yvi{!e????i"Yt?;(Ѱ e%
        $ openssl enc -d -aes-256-cbc -in foo.enc

        enter aes-256-cbc decryption password:

        test file
        =-=-


-----------------------------------------------------------------------------------------
openssl openssl openssl openssl openssl openssl
http://shib.kuleuven.be/docs/ssl_commands.shtml

generate a new private key and matching Certificate Signing Request (eg to send to a commercial CA)
	openssl req -out MYCSR.csr -pubkey -new -keyout MYKEY.key
	add -nodes to create an unencrypted private key
	add -config <openssl.cnf> if your config file has not been set in the environment

decrypt private key
	openssl rsa -in MYKEY.key >> MYKEY-NOCRYPT.key

generate a certificate siging request for an existing private key
	openssl req -out MYCSR.csr -key MYKEY.key -new

generate a certificate signing request based on an existing x509 certificate
	openssl x509 -x509toreq -in MYCRT.crt -out MYCSR.csr -signkey MYKEY.key

create self-signed certificate (can be used to sign other certificates)
	openssl req -x509 -new -out MYCERT.crt -keyout MYKEY.key -days 365

sign a Certificate Signing Request
	openssl x509 -req -in MYCSR.csr -CA MY-CA-CERT.crt -CAkey MY-CA-KEY.key -CAcreateserial -out MYCERT.crt -days 365
	-days has to be less than the validity of the CA certificate


convert DER (.crt .cer .der) to PEM
	openssl x509 -inform der -in MYCERT.cer -out MYCERT.pem

convert PEM to DER
	openssl x509 -outform der -in MYCERT.pem -out MYCERT.der

convert PKCS#12 (.pfx .p12) to PEM containing both private key and certificates
	openssl pkcs12 -in KEYSTORE.pfx -out KEYSTORE.pem -nodes
	add -nocerts for private key only; add -nokeys for certificates only

convert (add) a seperate key and certificate to a new keystore of type PKCS#12
	openssl pkcs12 -export -in MYCERT.crt -inkey MYKEY.key -out KEYSTORE.p12 -name "tomcat"

convert (add) a seperate key and certificate to a new keystore of type PKCS#12 for use with a server that should send the chain too (eg Tomcat)
	openssl pkcs12 -export -in MYCERT.crt -inkey MYKEY.key -out KEYSTORE.p12 -name "tomcat" -CAfile MY-CA-CERT.crt -caname myCA -chain

you can repeat the combination of "-CAfile" and "-caname" for each intermediate certificate


check a private key
	openssl rsa -in MYKEY.key -check
	add -noout to not disclose the key

check a Certificate Signing Request
	openssl req -text -noout -verify -in MYCSR.csr

check a certificate
	openssl x509 -in MYCERT.crt -text -noout

check a PKCS#12 keystore
	openssl pkcs12 -info -in KEYSTORE.p12

check a trust chain of a certificate
	openssl verify -CAfile MYCHAINFILE.pem -verbose MYCERT.crt

trust chain is in directory (hash format): replace -CAfile with -CApath /path/to/CAchainDir/
	to check for server usage: -purpose sslserver
	to check for client usage: -purpose sslient


debug an SSL connection [server doesn't require certificate authentication]
	openssl s_client -connect idp.example.be:443

debug an SSL connection with mutual certificate authentication
	openssl s_client -connect idp.example.be:8443 -CAfile MY-CA-CERT.crt -cert MYCERT.crt -key MYKEY.key

trust chain is in directory (hash format): replace -CAfile with -CApath /path/to/CAchainDir/
send the starttls command (smtp or pop3 style): -starttls smtp or -starttls pop3


keytool

keytool does not support management of private keys inside a keystore. You need to use another tool for that. If you are using the JKS format, that means you need another java-based tool. extkeytool from the Shibboleth distribution can do this.

Create an empty keystore
	keytool -genkey -alias foo -keystore truststore.jks
	keytool -delete -alias foo -keystore truststore.jks

Generate a private key and an initial certificate as a JKS keystore
	keytool -genkey -keyalg RSA -alias "selfsigned" -keystore KEYSTORE.jks -storepass "secret" -validity 360

you can also pass the data for the DN of the certificate as command-line parameters: -dname "CN=${pki-cn}, OU=${pki-ou}, O=${pki-o}, L=${pki-l}, S=${pki-s}, C=${pki-c}"

Generate a secret key that can be used for symmetric encryption. For this to work, you need to make use of a JCEKS keystore.
	keytool -genseckey -alias "secret_key" -keystore KEYSTORE.jks -storepass "secret" -storetype "JCEKS"

Generate a Certificate Signing Request for a key in a JKS keystore
	keytool -certreq -v -alias "selfsigned" -keystore KEYSTORE.jks -storepass "secret" -file MYCSR.csr

Import a (signed) certificate into a JKS keystore
	keytool -import -keystore KEYSTORE.jks -storepass "secret" -file MYCERT.crt

add a public certificate to a JKS keystore, eg the JVM truststore
	keytool -import -trustcacerts -alias "sensible-name-for-ca" -file CAcert.crt -keystore MYSTORE.jks

	If the JVM truststore contains your certificate or the certificate of the root CA that signed your certificate, then the JVM will trust and thus might accept your certificate. The default truststore already contains the root certificates of most commonly used sommercial CA's. Use this command to add another certificate for trust:

keytool -import -trustcacerts -alias "sensible-name-for-ca" -file CAcert.crt -keystore $JAVA_HOME/lib/security/cacerts
	the default password of the Java truststore is "changeit".
	if $JAVA_HOME is set to the root of the JDK, then the truststore is it $JAVA_HOME/jre/lib/security/cacerts
keytool does NOT support adding trust certificates to a PKCS12 keystore (which is very unfortunate but probably a good move to promote JKS)

delete a public certificate from a JAVA keystore (JKS; eg JVM truststore)
	keytool -delete -alias "sensible-name-for-ca" -keystore $JAVA_HOME/lib/security/cacerts
	the default password of the Java truststore is "changeit".
	if $JAVA_HOME is set to the root of the JDK, then the truststore is it $JAVA_HOME/jre/lib/security/cacerts

List the certificates inside a keystore
	keytool -list -v -keystore KEYSTORE.jks
	-storetype pkcs12 can be used

Get information about a stand-alone certificate
	keytool -printcert -v -file MYCERT.crt

Convert a JKS file to PKCS12 format (Java 1.6.x and above)
	keytool -importkeystore -srckeystore KEYSTORE.jks -destkeystore KEYSTORE.p12 -srcstoretype JKS -deststoretype PKCS12 -srcstorepass mysecret -deststorepass mysecret -srcalias myalias -destalias myalias -srckeypass mykeypass -destkeypass mykeypass -noprompt


certutil

Add a PKCS12 to a windows certificate store
	certutil -p secret -importpfx KEYSTORE.p12
-----------------------------------------------------------------------------------------


chkrootkit -> make sense -> ./chkrootkit

gnupg / gpg / pgp
	# generate a key. use DSA and ElGamal
	gpg --gen-key		

	gpg --list-secret-keys	# list a keyring
	gpg --list-public-keys		
	gpg --list-keys

	# generate a revocation certificate
	gpg --output revoke.asc --gen-revoke mykey

	# export a public key
	gpg --output alice.gpg --export alice@domain.org
	gpg --armor --export alice@domain.org	# into ASCII-armored format
	# export a private key
	gpg --armor --export-secret-keys mykey_id

	# import a public key
	gpg --import blake.gpg
	gpg --edit-key blake@domain.org
	Command> fpr	# fingerprint
	Command> sign	# or Command> trust
	Command> passwd	# create a throwaway passphase

	gpg --import --allow-secret-key-import keyfile	# if sec key in keyfile

	# encrypt by using recipient's public key
	gpg --output doc.gpg --encrypt --recipient blake@domain.org doc
	# encrypt a dir
	tar cf - /dir | gpg -c > files.tar.gpg
	tar cf - /dir | gpg -e > files.tar.gpg	# key-based encrypt
	find /dir -type f -exec gpg -e '{}' \;	# encrypt each file separately

	# decrypt by using private key
	gpg --output doc     --decrypt doc.gpg

	# make and verify signatures. signed by private key
	gpg --output doc.sig --sign doc
	gpg --output doc --decrypt doc.sig

	# clearsigned doc. in ASCII-armored signature
	gpg --clearsign doc

	# detached signatures. separate file and signature
	gpg --output doc.sig --detach-sig doc
	gpg --verify doc.sig doc

	# manage keypair
	gpg --edit-key blake@domain.org
	Command> toggle	# switch public/ private key
	Command> check	# verify the integrity

	# add/ delete key
	gpg --edit-key uid@domain.org
	Command> addkey	# or delkey

	gpg --delete-secret-key XA1592FF
	gpg --delete-key	XXXXXXXX

	# revoke key
	Command> revkey

	# set a default key from a sec key
	edit ~/.gnupg/gnupg.conf -> default-key	# define the default key

	# distribute key to a keyserver
	gpg --keyserver servername_or_ip --send-keys key_id
	wwwkeys.pgp.net
	www.keyserver.net
	pgp.mit.edu

	# obtain keys from a keyserver
	gpg --keyserver keyserver --revc-keys key_id
	gpg --keyserver keyserver --search-keys string_to_match

encrypted file system / encrypt
        /sbin/modprobe cryptoloop
        /sbin/modprobe blowfish
        dd if=/dev/zero of=secure bs=1k count=665600
        losetup -e blowfish /dev/loop0 secure
        Password:
        mkfs -t ext2 /dev/loop0 665600
        mount -t ext2 /dev/loop0 /mnt/loop
        umount /dev/loop0
        losetup -d /dev/loop0
        sync

encrypt file system / cryptsetup
	cryptsetup --verify-passphrase luksFormat /dev/sdc1 -c aes -s 256 -h sha256
	cryptsetup luksOpen /dev/sdc1 <ENCRYPTION_PARTITION_NAME>
	pv -tpreb /dev/zero | dd of=/dev/mapper/nebula bs=128M
	mkfs.ext4 /dev/mapper/nebula -m 1 -O dir_index,filetype,sparse_super

	dmsetup remove /dev/mapper/nebula

mount encrypt / crypt disk / luks
	mounted disk
	/dev/mapper/luks-931b8221-2851-4e51-8919-0d4f7634be3b on /media/xxx/pool type ext4 (rw,nosuid,nodev,uhelper=udisks2)

	mount /dev/mapper/<ENCRYPTION_PART_NAME> /media/<USR_NAME>/<ENCRYPTION_PART_NAME>

	umount /media/xxx/pool
	udisksctl mount -b /dev/mapper/luks-931b8221-2851-4e51-8919-0d4f7634be3b

PAM control / pam control
	login time control	/etc/pam.d/login -> add 'auth required /lib/security/pam_time.so
	change ftp to passwd change shell /etc/pam.d/ftp -> # pam_shells.so

	specify who can su -> in /etc/pam.d/su, uncomment
		auth       required     /lib/security/pam_wheel.so use_uid
	# add the uid into wheel group in /etc/group

	root-memeber group can do su only. put root-users into root-member grp
		auth       sufficient   /lib/security/pam_stack.so service=root-members
		auth       required     /lib/security/pam_deny.so

	/etc/pam.d/passwd -> control passwd length while passwd change
	#password   required    /lib/security/pam_stack.so service=system-auth
	password    required    /lib/security/pam_cracklib.so minlen=12
	password    required    /lib/security/pam_unix.so        use_authtok md5

	# revoke / disable user when max failure telnet login attempts reached
	edit /etc/pam.d/login -> add
	auth       required     pam_tally.so onerr=fail no_magic_root
	account    required     /lib/security/pam_tally.so deny=2 reset no_magic_root
	# add above 2 lines into sshd
	# to reset
	/sbin/pam_tally --user somebody --reset

sudo	-> visudo -> # Uncomment to allow people in group wheel to runa all command
	%wheel ALL=(ALL)	ALL
	# then edit /etc/group -> wheel:x:10:root,someuser

Linux interface specific options are in
	/proc/sys/net/ipv4/conf/<interface-name>

Linux Firewall/ firewall machine setup. Filesystem/ partition size
	/ 190, /boot 3, /var 30  /home 1  /usr 250  /tmp 20  swap 36 -> actual usage in tp755cd
	/ 1500 /boot 50 /var 400 /home 100 /usr 4500 /tmp 300 /opt 1500 -> penguinsecurity
Linux firewall log. put below line into /etc/syslog.conf-> service syslogd restart
	kern.info /var/log/firewall
	# all error and warning msg logged
	*.warn; *.err	/var/log/errmsg

Linux ipmasq/ ipmasqerading (6.2 and 7.2 default enabled)
	echo 1 > /proc/sys/net/ipv4/ip_forward
	vi /etc/sysctl.conf -> net.ipv4.ip_forward = 1

root ftp / ftp by root
	edit /etc/ftpusers -> comment out root
	edit /etc/ftpaccess -> add
		allow-uid root
		allow-gid root

limit login session	# limits some user telnet login
	/etc/security/limits.conf -> maxlogins ...
	also need to add the following line in /etc/pam.d/login
	session	required	/lib/security/pam_limits.so

/etc/login.defs		# user login control / user control
	PASS_MAX_DAYS	max # of days a passwd is valid
	PASS_MIN_DAYS	min days allowed between passwd changes
	UID_MIN		min value for auto uid selection
	GID_MIN		min valye for auto gid selection
	PASS_MIN_LEN	min acceptable passwd length. this does NOT work. it's
			superseded by PAM "pam_cracklib". See pam_crasklib
			param "minlen" /lib/security/pam_cracklib.so

	/etc/default/useradd	GROUP	default group
				HOME	default user home location
				INACTIVE	max # of days after a passwd
						expired that a yser can change
				EXPIRE	expire date in format yyyy-mm-dd
				SHELL	default shell
				SKEL	default profile dir

	when a new user account is created with useradd, in /etc/passwd and
	/etc/shadow, some setting are recorded as following:
	/etc/passwd:
	<username>:x:UID_MIN+:GROUP:<GECOS>:HOME/<username>:SHELL

	/etc/shadow:
	<username>:<password>:<date>:PASS_MIN_DAYS:PASS_MAX_DAYS:PASS_WARN_AGE:INACTIVE:EXPIRE:

	useradd -c "hadoop" -m -s "/bin/bash" hadoop
	-c	name of user
	-m	create /home/hadoop
	-s 	default shell

	usermod -a -G sudo user	# add user into sudo group

root telnet / telnet by root	# allow root telnet
	edit /etc/securetty -> add pts/0 , pts/1, pts/2, pts/3, pts/4

Linux user login time control
	/etc/security/time.conf -> define what, where, who and when
	/etc/pam.d/login -> add 'auth required /lib/security/pam_time.so

Linux change shell to /usr/bin/passwd
	either add /usr/bin/passwd into /etc/shells
	or /etc/pam.d/ftp -> # pam_shells.so  -> comment out

Linux tripwire
	# Initialize
	cd /etc/tripwire
	twadmin --generate-keys --site-keyfile ./site.key
	twadmin --generate-keys --local-keyfile ./$HOSTNAME-local.key
	twadmin --create-cfgfile --cfgfile tw.cfg --site-keyfile site.key twcfg.txt	# create tw.cfg
	twadmin --create-polfile --cfgfile tw.cfg --site-keyfile site.key twpol.txt	# create tw.pol
	chmod 600 *	# remove *.txt
	tripwire --init --cfgfile tw.cfg --polfile tw.pol --site-keyfile site.key --local-keyfile $HOSTNAME-local.key

	# Print the database
	twprint -m d

	# Verify
	tripwire --check

	# Update
	tripwire --update --twrfile /var/lib/tripwire/report/hostname-time.twr

	# test tripwire sent through email
	tripwire --test --email your@email.address

	# get the plain texted pol file from encrypted file
	# twpol.txt can be removed because it was encrypted into tw.cfg
	twadmin -m p > twpol.txt

	# modify /etc/cron.daily/tripwire-check
	test -f /etc/tripwire/tw.cfg &&  /usr/sbin/tripwire --check | mail -s TripwireCheckLog root@localhost

	# auto report. put it into crontab -e
	/usr/sbin/tripwire -m c | mail -s "Tripwire from HOST" root@localhost
	cat above into /usr/local/bin/runtw.sh -> chmod u+x runtw.sh ->
	crontab -e -> add
	1 6 * * * /usr/local/bin/runtw.sh

	# modify twpol.txt
	...
	# Something I want to monitor
	(
	  rulename = "Something I want to monitor"
	  serverity = $(SIG_HI)
	)
	{
	  /filesystem/being_monitored	-> $(SEC_BIN) (recurse = 10);
	}
	...

	# The option fo SEC_*. Look for definition in twpol.txt
	SEC_BIN	= binary
	SEC_CONFIG = configuration file

rpm integrity check
	rpm -V net-tools	/bin/telnet
	rpm -V procps		/bin/ps
	rpm -V shadow-utils 	lastlog, chage, gpasswd
	rpm -V file-utils	/bin/ls
	rpm -V SysVinit		last, wall, etc
	rpm -V sh-utils		who
	rpm -V util-linux	login
	rpm -V rpm
	rpm -V setup		/etc/passwd, group

Linux nessus
	nmap cmd line	nmap -sT -O localhost

	nessus-mkcert	# create cert
	nessus-adduser
	nessus-fetch --register xxxx-xxxx-xxxx-xxxx-xxxx
	nessus-fetch --plugins
	cd /var/lib/nessus ; mkdir plugins ; cd plugins; tar -xzvf all-2.0.tar.gz
	cd /var/lib/nessus ; touch nessus-services
	nessusd -D	# start daemon

Linux Ethereal/ ethereal
	http://www.ethereal.com/distribution/ -> download to /download
	cd /usr/src-> tar -xzvf /download/ethereal-version.tar.gz
	cd ethereal* -> ./configure --prefix=/usr
	make -> make install -> ethereal &

	# capture UDP but NOT DNS
	edit you filter, add expr, select UDP, select Souce or Dest Port, select != at right panel, type 53.

Linux tcpdump (AIX uses iptrace)
	tcpdump -i eth0 -lnx -> n=no DNS
	tcpdump output:
    Hexadecimal  Binary				        Meaning
    ---- ----    -------- -------- -------- --------    ---------------------------------------------------
    4500 0054    01000101 00000000 00000000 01010100    VERS=4, HLEN=5, Service=00, Total length=0054 (Hex)
    0172 0000    00000001 01110010 00000000 00000000    ID=0172, FLG=0, FO=0
    .... ....

	# capture all UDP, but NOT DNS.
	tcpdump -i eth1 'proto UDP and (port not 53)'

capture vlan / VLAN / vLAN tagged traffic
	tcpdump -Uw - | tcpdump -i eth0 -en -r - vlan 20

windump (install winPcap as prerequisite)
        windump -D      # list all available captured NIC. Wireless doesn't seemto be supported.
windump sample
        windump -i 3 -lnx host 60.191.123.155   # n= no DNS & on specific IPwith NIC 3
        windump host bamse and host cartman and udp     # capture udp between 2 hosts
        windump -v -n "icmp[0]=8 or icmp[0]=0"  # capture icmp echo req and echo reply msg. n= don't resolve ip to names

xauth | xauthority | to avoid Xlib: connection to ":0.0" refused by server > Xlib: No protocol specified > Error: Can't open display: :0.0
	xauth -f ~source_user/.Xauthority extract - :0 | xauth merge -

tap tun	http://en.wikipedia.org/wiki/TUN/TAP
TAP (as in network tap) simulates a link layer device and it operates with layer 2 packets such as Ethernet frames. TUN (as in network TUNnel) simulates a network layer device and it operates with layer 3 packets such as IP packets. TAP is used to create a network bridge, while TUN is used with routing.

ssh / SSH / sshd performance
	@ /etc/ssh/sshd_config
	GSSAPIAuthentication no
	UseDNS no

Linux SSH/ssh. to prevent xauth fails between machine w/ xwin and machine w/o xwin, do below 2 steps
	$unset DISPLAY
	$ssh -x destinationhost

Linux ssh keys setup http://www.arches.uga.edu/~pkeck/ssh/
	on local machine:
	ssh-keygen -t dsa
	scp ~/.ssh/id_dsa.pub 192.168.1.1:.ssh/authorized_keys2

	on remote sshd machine:
	ssh-agent sh -c 'ssh-add < /dev/null && bash'
	# start ssh-agent, add the default identity keys

ssh + rsync
	rsync -avz -e ssh --delete /file/ user@remote:/path/
	rsync -vrlptg --delete /SRC /DEST --exclude-from=/EXCLUDE
	--exclude 'source'
	--exclude-from '/home/path/exclude.txt'

	rsync --compress --sparse --progress -e ssh source.file user@ipaddr:/path
	rsync --archive -v -z -r --inplace --progress -e ssh source target

ssh to run cmd on remote srv
	ssh remote_server "cmd 1; cmd 2"

ssh + socks
	install tsocks -> edit /etc/tsocks.conf
	tsocks ssh root@10.10.49.193

ssh encrypted channel port forwarding
	ssh -T -L 5901:myvncserver:5900 -C -N username@mysshserver
	# -T: not to allocate a tty for shell. this optional.
	# -L 5901:myvncserver:5900 forward port 5901 on the local machine to port 5900 on myvncserver
	# -C: tells ssh to employ compression. optional
	# -N: tells ssh not to execute a shell or cmd. since the purpose is to connect via VNC w/o cmd.

	ssh -L 8888:ssh_host:80 -L 110:ssh_host:110 25:ssh_host:25 user@computer -N

ssh proxy
	sudo ssh -N -v -D 8081 user@domain.net
	ssh -C2qTnN -D 8080 usr@domain.net

	ssh -D *:9999 158.85.164.5

	for i in 50070 8080 8088 2222 2223; do ssh -N -f -L 192.168.200.2:$i:127.0.0.1:$i localhost;

openssh / ssh install
	./configure --prefix=PATH --with-ssl-dir=PATH
	cd $ssh	-> mv etc etc.orig -> copy old etc whih stores old keys
	# sshd restart with reading config file
	$sshd/sbin/sshd -t -f $sshd/etc/sshd_config

ssh	execute the remote cmd. Copy the following in eXecutable script.
	ssh -n user@host.com ./run.sh <<END_SCRIPT
	quote passwd
	END_SCRIPT
	exit 0

	error_code > channle 6: open failed: connect failed: Connection timed out
	Add the following into sshd_config
	AllowTcpForwarding yes
	GatewayPorts yes

ssh vpn | ssh tunnel vpn
	# set "PermitTunnel yes" on both server and client side

	# client:
	ssh -i /home/mabo/.ssh/heat_key -f -o Tunnel=ethernet -T -N -w 2000:2000 root@158.85.124.50
	ip link set tap2000 up
	ip addr add 10.100.100.10/24 dev tap2000
	ip route del default
	ip route add default via 10.100.100.1 dev tap2000

	# server:
	iptables -t nat -A POSTROUTING -s 10.100.100.0/24 -j SNAT --to-source 158.85.124.50
	ip link set tap2000 up
	ip addr add 10.100.100.1/24 dev tap2000

ssh vpn	# set "PermitTunnel yes" on both server and client side
	on server
	openvpn --mktun --dev tap9
	ip addr add 10.0.0.1/24 dev tap9
	ip route add 10.0.0.0/24 dev tap9
	ip link set tap9 up

	on client
	openvpn --mktun --dev tap9
	ip addr add 10.0.0.2/24 dev tap9
	ip route add 10.0.0.0/24 dev tap9
	ip link set tap9 up

	ssh -o Tunnel=ethernet -w 9:9 usr@server_ip

ssh tunnel
	ssh -i .ss_lock -N -f -R  192.168.200.2:2222:127.0.0.1:22 jump_host_ip
	ssh -L <local port>:<remote computer>:<remote port> <user>@<remote ip>

ssh tunnel @ ly	# 192.168.200.2 (hub@idevops) is an internal IP of a VM within qingcloud
	bryan@idevops 	ssh -N -f -L 192.168.200.2:20081:127.0.0.1:20081 localhost
	root@lynd1	ssh -N -f -R 192.168.200.2:20081:192.168.1.101:22 bryan@idevops

ssh tunnel script
=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
#!/bin/bash
# Get the  PID of the ssh process run by the SSHTunnel user
rm -f /tmp/pid.SSHTunnel > /dev/null

#ps -U mytunneluser | grep -v grep | grep ssh >/tmp/pid.SSHTunnel
ps -ef | grep 20081 | grep ssh | awk '{print $2}' > /tmp/pid.SSHTunnel

# If the file is zero sized, then SSH is not running
if [ -s /tmp/pid.SSHTunnel ]
	then
		echo "I'm alive. Hahaha"
fi

if [ ! -s /tmp/pid.SSHTunnel ]
	then
  		echo "SSH Tunnel not running - restarting"
		ssh -N -f -R 192.168.200.2:20081:192.168.1.101:22 bryan@121.201.13.44
fi

rm -f /tmp/pid.SSHTunnel >/dev/null
=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

ssh tunnel crontab
# m     h       dom     mon     dow     command
*/1     *       *       *       *       /root/createtunnel.sh


Linux Password Lost/ Control / passwd / password lost
	boot machine as linux single in lilo. then do passwd change.
	For Grub hit 'e' at the Grub screen and then add 'single' to the kernel line and boot.

iptables/ IPTables load modules for passive ftp / iptables faq
	/sbin/modprobe ip_conntrack_ftp
	/sbin/modprobe ip_nat_ftp

	# Flushing
	iptables -F
	iptables -F INPUT
	iptables -F OUTPUT
	iptables -F FORWARD
	iptables -F -t mangle

	# Deleting
	iptables -X

	# monitor the traffic
	iptables [ -t <table>] [-v -n] -L [chain]
	iptables -L -nv
	iptables -L -nv -t nat
	iptables -L -nv -t mangle
	iptables -L INPUT -nv
	iptables -L FORWARD -nv

	# block / blocking by mac address
	iptables -I INPUT -p all -m mac --mac-source aa:bb:cc:dd:ee:ff -j DROP

	# syn flood protection
	echo "1" > /proc/sys/net/ipv4/tcp_syncookies
	iptables -A INPUT -m limit --limit 10/minute --limit-burst 20 -j ACCEPT

	# Syn Cookies
	echo "1" > /proc/sys/net/ipv4/tcp_syncookies
	echo "1" > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
	echo "1" > /proc/sys/net/ipv4/icmp_ignore_bogus_error_responses
	echo "1" > /proc/sys/net/ipv4/conf/all/accept_redirects
	echo "1" > /proc/sys/net/ipv4/conf/all/log_martians

	# Reduce DDOS by reducing timeouts
	echo 30 > /proc/sys/net/ipv4/tcp_fin_timeout
	echo 1800 > /proc/sys/net/ipv4/tcp_keepalive_time
	echo 1 > /proc/sys/net/ipv4/tcp_window_scaling
	echo 0 > /proc/sys/net/ipv4/tcp_sack
	echo 1280 > /proc/sys/net/ipv4/tcp_max_syn_backlog

	# Anti- Spoofing
	for a in /proc/sys/net/ipv4/conf/*/rp_filter;
		do
			echo 1 > $a
		done

	# DNAT
	echo "1" > /proc/sys/net/ipv4/ip_forward
	iptables -A FORWARD -i $INET_IFACE -o $LAN_IFACE -p tcp \
		--sport 1024: -d $WEB_IP --dport 80 \
		-m state --state NEW -j ACCEPT
	iptables -A FORWARD -i $LAN_IFACE -o $INET_IFACE -p tcp \
		-m state --state ESTABLISHED,RELATED -j ACCEPT
	iptables -t nat -A PREROUTING -p tcp -d 15.45.23.67 --dport 80 \
		-j DNAT --to-destination 192.168.1.1-192.168.1.10

	# SNAT working with -o | --out-interface ONLY
	echo "1" > /proc/sys/net/ipv4/ip_forward
	iptables -A FORWARD -i $LAN_IFACE -o $INET_IFACE \
		-m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
	iptables -A FORWARD -i $INET_IFACE -o $LAN_IFACE \
		-m state --state ESTABLISHED,RELATED -j ACCEPT
	iptables -t nat -A POSTROUTING -p tcp -s $LAN_NETWORK -o $INET_IFACE \
		-j SNAT --to-source 194.236.50.155-194.236.50.160:1024-32000


	#iplimit. When more than 4 connect from single IP, do REJECT
	iptables -A INPUT -p tcp --syn --dport http \
		 -m iplimit --iplimit-above 4 -j REJECT


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
	# transparent proxy
# Outgoing LAN webclient traffic is redirected to Squid.
iptables -t nat -A PREROUTING -i $LAN_IFACE -p tcp \
         -s $LAN_NETWORK --sport 1024:65535 --dport 80 \
         -j REDIRECT --to-port 3128

# INPUT rule to accept the packet. To ACCEPT the REDIRECT
iptables -A INPUT -i $LAN_IFACE -p tcp \
         -s $LAN_NETWORK --sport 1024:65535 -d $LAN_IP --dport 3128 \
         -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

# Squid establishes connections with the remote web server as client.
iptables -A OUTPUT -o $INET_IFACE -p tcp \
         -s $INET_IP --sport 1024:65535 --dport 80 \
         -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

iptables -A INPUT -i $INET_IFACE -p tcp \
         --sport 80 -d $INET_IP --dport 1024:65535 \
         -m state --state ESTABLISHED,RELATED -j ACCEPT

# Responds as a server to LAN clients.
iptables -A OUTPUT -o $LAN_IFACE -p tcp \
         -s $LAN_IP --sport 80 --dport 1024:65535 \
         -m state --state ESTABLISHED,RELATED -j ACCEPT
vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv


# iptables for ftp/ ftp
	In general, in ftp scenario, this is the diagram of packet flow.

	the control connection
	ftp client:tcp:1024+ -> ftp server:tcp:21

	the data channel
	in port mode, ftp server:tcp:20 -> ftp client:tcp:1024+

	in passive mode, ftp client:tcp:1024+ -> ftp server:tcp:1024+

	# Control connection
	iptables -A INPUT -i $INTERNET_INTERFACE -p tcp \
	    --sport 1024:65535 -d $INTERNET_IP --dport 21 \
	    -m state --state NEW -j ACCEPT

	# Port Mode data channel
	iptables -A OUTPUT -o $INTERNET_INTERFACE -p tcp \
    	    -s $INTERNET_IP --sport 20 \
    	    --dport 1024:65535 -m state --state NEW -j ACCEPT

	# Passive Mode data channel
	iptables -A INPUT -i $INTERNET_INTERFACE -p tcp \
    	    --sport 1024:65535 -d $INTERNET_IP --dport 1024:65535 \
    	    -m state --state NEW -j ACCEPT

iptables/ denial-of-service attacks
	tcp syn flooding -> echo 1 > /proc/sys/net/ipv4/tcp_syncookies
	ping flooding
	ping of death. large ping packet
	udp flooding
	fragmentation bombs
	buffer overflows
	icmp redirect bombs

iptables/ firewall optimization
	# begin with rules that block traffic on high ports -> nfs or x
	# use the state module for established and related matches
	# consider the transport protocol
	 	tcp service: bypass the spoofing rules
	 	udp: place incoming packet rules after spoofing rules
	 	tcp vs. udp: place udp rules after tcp rules
	 	icmp: place their rules late in the rule chain
	# place firewall rules for heavily used service as early as possible
	# use the multiport module to specify port lists

Linux security. suid/ suig. To diable the suid bits on selected programs.
	chmod a-s /usr/bin/chage
	chmod a-s /usr/bin/gpasswd
	chmod a-s /usr/bin/wall
	chmod a-s /usr/bin/chfn
	chmod a-s /usr/bin/chsh
	chmod a-s /usr/bin/newgrp
	chmod a-s /usr/bin/write
	chmod a-s /usr/sbin/usernetctl
	chmod a-s /usr/sbin/traceroute
	chmod a-s /bin/mount          
	chmod a-s /bin/umount
	chmod a-s /bin/ping  
	chmod a-s /sbin/netreport

	chattr +i /etc/services
	chattr +i /etc/passwd
	chattr +i /etc/shadow
	chattr +i /etc/gshadow
	chattr +i /etc/group

immune immunization/ immunize
	chmod 700 /bin/rpm
	chmod -R 700 /etc/rc.d/init.d

su - root control
	in /etc/pam.d/su, uncomment
	#auth       required     /lib/security/pam_wheel.so use_uid

	Add user xxx into wheel group
	usermod -G10 xxx  
	usermod -G wheel xxx

	wheel/ WHEEL group is BSD flavor, vs. AIX it's AUDIT group w/ gid 10.

Home machine securing
	dhcpd.conf
	iptables firewall+ nat, chkconfig --level 2345 firewall.iptables on
	chkconfig --level 2345 sendmail etc off
	sshd_config -> PermitRootLogin no + Protocol 2
	wheel/ /etc/pam.d/su
	chattr +i passwd group shadow gshadow services
	chmod 700 /bin/rpm, chmod -R 700 /etc/rc.d/init.d

	TP701C Video config: Chips&Tech, CT65545, 50-90 Hz, 640x 480

file permission		1755	-> sticky
			2000	-> sgid		chmod g+S
			4000	-> suid		chmod u+s

chcon - change security context
	# scenario: Apache doesn't display TARGET_FILE even it has right permission set
	chcon -R -t httpd_sys_content_t TARGET_FILE

###############################################################################
###############################################################################

Linux Performance / linux performance / linux perf

Linux Kernel Configuration Options (kernel configuration/ config options)
	vi /etc/sysctl.conf
	net.ipv4.ip_forward = 1  
	or sample
	sysctl -a -> list all options, such net.ipv4.ip_forward...
	sysctl -w net.ipv4.ip_forward ='1'

	#Ignore ICMP echo requests to broadcast address
	net.ipv4.icmp_destunreach_rate = 10
	net.ipv4.icmp_echoreply_rate = 10
	net.ipv4.icmp_paramprob_rate = 10
	net.ipv4.icmp_timeexceed_rate = 10

	kernel config opetions for IP/ip
	#default ttl
	net.ipv4.ip_default_ttl = 255
	#local port range for tcp and udp connections
	net.ipv4.ip_local_port_range = 1024 32000
	#no path MTU discovery
	net.ipv4.ip_no_pmtu_disc = 1
	#ip frag mem (fragmentation memory) threshholds and timeouts
	net.ipv4.ipfrag_high_thresh = 262144
	net.ipv4.ipfrag_low_thresh = 196608
	net.ipv4.ipfrag_time = 30

	kernel config options for tcp/TCP
	#detect broken connection early
	net.ipv4.tcp_keepalive_probes = 5
	net.ipv4.tcp_keepalive_time = 600
	#protection against SYN attacks
	net.ipv4.tcp_syncookies = 1
	#protect against unfinished connections
	net.ipv4.tcp_retries1 = 3
	#protection against FIN attacks
	net.ipv4.tcp_fin_timeout = 30

liux max perf / maximum performance
	add these settings to /etc/sysctl.conf
	# Decrease the time default value for tcp_fin_timeout connection
	net.ipv4.tcp_fin_timeout = 30
	# Decrease the time default value for tcp_keepalive_time connection
	net.ipv4.tcp_keepalive_time = 1800
	# Turn off tcp_window_scaling
	net.ipv4.tcp_window_scaling = 0
	# Turn off the tcp_sack
	net.ipv4.tcp_sack = 0
	# Turn off tcp_timestamps
	net.ipv4.tcp_timestamps = 0

linux tcpip tuning
	sysctl -w net/ipv4/tcp_rmem="4096 524176 524176"
	sysctl -w net/ipv4/tcp_wmem="4096 524176 524176"
	sysctl -w net/ipv4/tcp_sack=1
	sysctl -w net/ipv4/tcp_window_scaling=1

These changes are lost after a reboot. You can update the /etc/sysctl.conf
file make it permanent. The format of the sysctl.conf file is a bit different
from the sysctl command:

	net.ipv4.tcp_rmem = 4096 524176 524176
	net.ipv4.tcp_wmem = 4096 524176 524176
	net.ipv4.tcp_sack = 1
	net.ipv4.tcp_window_scaling = 1

###############################################################################
###############################################################################

Linux multimedia

Linux picture / photoshop
	$gimp

picture resize
	convert -resize 750x500 -quality 80% *.jpg
	find . -name "*.jpg" -exec convert -resize 900x -quality 100% {} /output_dir/{} \;
	mogrify -resize 800 *.jpg
	mogrify -resize 320x240! *.jpg	# resize to a fixed size

picture blur
	convert orig.jpg -blur 0x4 blurred.jpg

picture dither (reduce color)
	convert Black_Swan_bg2.jpg -blur 0x4 +dither -colors 16 Black_Swan_bg6.jpg

Linux multimedia
	CD rip		grip	# naming \t - Track number
	audio		xmms	# rm ~/.xmms if having any issue.

	# resolve the msg: modprobe: Can't locate module sound-... in
	# /var/log/messages

	kcontrol -> sound -> mixer -> Max count of tested devices per mixer ->
	change from 2 to 1

	# volume control
	gnome-volume-control

	# CD extract
	sound-jucier

	edit /etc/modules.conf and add =>
	alias sound-slot-1 off
	alias sound-service-1-0 off

linux grip / rip cd / multimedia pre- req:
	gcc-c++, libstdc++-devel, curl-devel, vte-devel, libgnomeui, ncurses-devel

linux alsa | advanced linux sound architect configuration | capture | audio
	alsamixer > f5 to expand > enable "Capture" and capture over Line > to enable record
	text mode record 		> arecord -vv -fdat foo.wav
	text mode play a recorded file 	> aplay foo.wav

	gnome-sound-properties		#

amarok kde amarok audio internet radio > increase buffer
	edit ~/.kde/share/apps/amarok/xine-config
	engine.buffers.audio_num_buffers:230 change to read 10000

	edit ~/.kde/share/config/amarokrc
	add > Streaming Buffer = 400 or 800

Windows / windows dvd copier.
	Run DVD Decrypter to find out how many layers on DVD.
	Use DVD Decrypter to copy all DVD files on hard drive.
	Use DVDFab to Copy main movie from DVD (Strip/ Split)
		or Use DVD2one to shrink the movie into one DVD
	Use Nero to burn the disk.

language setting @ windows
	regional & language > languages > details > add @ settings > add
	regional & language > advanced > set @ select a language to match lang..

###############################################################################
###############################################################################

X, xfree86/ XFree86, GUI/gui, graphic / X11 / x11
Linux X window/ icewm preference configuration
	/usr/X11R6/lib/X11/icewm/preferences	(search theme to change)

x window kill
	ctrl+ alt+ backspace

x window, start a new session on pts8
	startx -- :1

Linux Xwindow/ xwindow/ xwin
	xinit-> twm &
	xsetroot -solid lightblack
	xsetroot -solid "medium sea green"
	xsetroot -bitmap bitmapfilename

	xclock -g 60x50-0+0 -bw 0 &

	xdaliclock -root -builtin3 -cycle

Linux Graphic VI / vi
	$gvim
	~/.vimrc -> add following line at the bottom
	set guifont=-B&H-LucidaTypewriter-Medium-R-Normal-Sans-12-120-75-75-M-70-ISO8859-1
	highlight Normal guibg=grey90
	highlight Cursor guibg=Red guifg=NONE
	highlight NonText guibg=grey80
	highlight Constant gui=NONE guibg=grey95
	highlight Special gui=NONE guibg=grey95

vimrc	git clone https://github.com/altercation/vim-colors-solarized
	mv solarized.vim ~/.vim/colors/

vimrc	~/.vimrc
	syntax enable
	set background=dark
	colorscheme solarized

ImageMagick
	/usr/local/bin/magick/display -window root filename
	display -window root /usr/share/wallpapers/No-Ones-Laughing-3.jpg

	xwindow screensaver config
	xset

	************
	add below into XF86Config to activate DPMS #DPMS=display power mgmt sys
	    Section "Monitor"
	        Identifier  "My Monitor"
	        HorizSync   31.5-130    
	        VertRefresh 55-160  
	        Option      "DPMS"
	    EndSection
	************

	then run #xset +dpms ; xset 1200 1500 1800

Linux icewm/ IceWM	display power mgmt
	xset -dpms	# disable display power mgmt

Linux screenshot capture
	xset screen shot/ screenshot capture
	http://www.saragossa.net/LinuxG3/ls-skreen.shtml
	xwd > /tmp/mydump.xwd	# crosshair click to select
	xwud -in /tmp/mydump.xwd

	import -window root /tmp/screenshot.jpeg

Linux xlock/ screensaver/ screen saver causing system hangs (icewm/ IceWM)
# Command to lock display/screensaver
LockCommand="xlock -mode blank"

Linux screen screenshare screen sharing


Linux screen screenshare screen sharing
	1. set /usr/bin/screen setuid root
	2. teacher runs $screen -S SessionName #-S specifies sessionname
	3. student ssh into teacher's machine
	4. teacher Ctrl-a :multiuser on
	5. teacher Ctrl-a :acladd student_id
	   to change readonly -> Ctrl-a :aclchg student -w
	6. student $screen -x username/SessionName

SuSE X display local setting
(optional)	from logged in user's session	-> xhost +local
		from su'ed user's session	-> export DISPLAY=:0.0

xscreensaver # lock screen immediately
	xscreensaver-command -lock

Linux text mode screen configuration
	screen
	/etc/skel/.screenrc

text mode terminal screensaver
	setterm -blank [0-60]

Linux xterm/XTerm	http://www.uwsg.iu.edu/edcert/session3/x11/xterm.html
	Change xterm font size permanently-> 	vi ~/.Xresource
					eg.-> 	xterm*background: DarkBlue
						xterm*font: 9x15
	Make change effective-> 	xrdb -load .Xresource
					xrdb -merge $HOME/.Xdefaults
	Font size	Tiny	5x8
			Small	6x10
			Medium	8x13
			Large	9x15
			Huge	10x20

font
	mkdir /usr/share/fonts/local/
	cp mingliu.ttc /usr/share/fonts/local/
	ttmkfdir -d /usr/share/fonts/local/ -o /usr/share/fonts/local/fonts.scale 	# create fonts.scale
	cd /usr/share/fonts/local/ ; mkfontdir 						# create fonts.dir
	chkfontpath --add /usr/share/fonts/local/ 					# add font path
	fc-cache /usr/share/fonts/local/ 						# create font cache

	mkdir /usr/share/fonts/windows ; cd /usr/share/fonts/windows
	ttmkfdir .
	cp fonts.scale fonts.dir
	chkfontpath --add /usr/share/fonts/windows

IBM terminal 315x	export TERM=ibm3151

aterm / transparent terminal / transparent aterm / x11 / X11

	xlsfonts | less		# list all the available fonts

	Blue Terminal
	/usr/share/xfce/icons/Terminal.xpm
	/usr/local/bin/aterm -tr -rv -tint blue -sh 60 -ls -name 'Blue Terminal'

	Red Terminal
	/usr/share/xfce/icons/Terminal.xpm
	/usr/local/bin/aterm -tr -rv -tint red -sh 60 -ls -name 'Red Terminal'

	-rv	revised font color
	+sb	disable scrollbar
	-sh	shade
	-sl	-sl number, save number lines in the scrollback buffer.
	-tr 	transparent
	-tint	tint blue, the background color
	-name	terminal name

	aterm.Xdefaults
		aterm*transparent:true
		aterm*transpscrollbar:true
		aterm*shading:60

gnome-xgl | gnome | xgl
	gnome-xgl-switch --disable-xgl
	gnome-xgl-switch --enable --auto


###############################################################################
###############################################################################

Java / JAVA / java
	unzip -l *.jar	# list all contents in jar
	zip -r wcsruntime.jar com/*
	# zip all classes in com/ into wcsruntime.jar

java thread dump
	kill -3 `ps -leaf | grep Commerce | head -1 | awk '{print $2}'`
	# get a tree view of the threads
	ps -e f

download java plugin
	http://wp.netscape.com/plugins/jvm.html

java plugin in mozilla http://plugindoc.mozdev.org/faqs/java.html
	download jre 1.4.2 from http://java.sun.com/j2se/1.4.2/download.html
	install from rpm -> cd $MOZILLA_HOME/plugin/ ->
	ln -s /usr/java/j2re1.4.2_02/plugin/i386/ns610-gcc32/libjavaplugin_oji.so

	# if receiving libgcc_s.so.1 cannot open shared object file error
	# need to download libgcc_s.so.1 gcc322 rpm then install

###############################################################################
###############################################################################

Application/ application

adobe disable adobe plugin in firefox
	cd /usr/lib/acroread/Browser/intellinux/ -> mv nppdf.so nppdf.so.orig

firefox using specific profile, like java
	firefox -profile <java_profile>

firefox start with a newly created profile
	firefox -ProfileManager

firefox gives: Could not initialize the browser security component
	Help > Troubleshooting > Open Containing Folder > rm -fr cert8.db

firefox The page you are trying to view cannot be shown because the authenticity of the received data could not be verified.  Please contact the website owners to inform them of this problem.

	about:config > security.tls.insecure_fallback_hosts > left mouse click > add the complete website

Linux Wine Configuration
	Download from www.linuxwine.com-> tar -xvzf source
	cp xxx /tmp/wine
	Read README and follow the indtruction
	Change the path in /usr/local/etc/wine.conf-> add /windows/C/winnt/system;/windows/C/winnt/system32

Microsoft windows diet / windows performance/ microsoft
	cmd -> sfc.exe /purgecache
	%windows%driver/cachei386/driver.cab
	%windows%$NtUninstallQ*$
	edit %windows%infsysoc.inf -> replace all 'hide' with blank -> go to add/ remove programs

	disacle Windows Messager -> regedit -> HKEY_CURRENT_USERSoftwareMicrosoftWindowsCurrentVersionRunMSMSGS� /BACKGROUND

	fasten speed -> regedit -> HKEY_LOCAL_MACHINESYSTEMCurrentControlSetControlSession ManagerMemory ManagementPrefetchParameters -> EnablePrefetcher -> 1

Lotus Notes / lotus notes connection
	File -> Mobile -> Server Phone Number -> Advanced -> Connections
	# increase/ decrease font size. add the following into ~/lotus*/notes.init
	Dislay_font_adjustment=-1

        # disable sametime in systemtray, add
        com.ibm.collaboration.realtime.application/useSystemTray=false
        into /opt/ibm/lotus/notes/framework/rcp/plugin_customization.ini

	# refresh design to recover Sent folder missing
	Workspace > mail on Local > right click > Replication > Refresh Design > choose actual server address @ With design from Server > Yes

Wiki | wiki
http://wiki.dreamhost.com/MediaWiki
Wiki / wiki > edit LocalSettings.php, AFTER the line:
	require_once( "includes/DefaultSettings.php" );

    - disallow edits by unregistered users
	$wgGroupPermissions['*']['edit'] = false;
	$wgShowIPinHeader = false;

    - disallow account creation
	$wgGroupPermissions['*']['createaccount'] = false;

    - $wgLogo image location
    	$wiki/skins/common/image/wiki.png

    - Wiki security
    	move LocalSettings.php from config to root directoy
	chmod 640 LocalSettings.php

wiki logo	mv skins/common/images/wiki.png

=-=-
Wireless | wireless configuration
wireless configuration on thinkpad t43
	/etc/sysconfig/hwdata
	lspci -vn	# lspci | grep -i Atheros
	lshw -C network	# show hardware device
	iwlist eth1 scanning
	modprobe ipw2200 led=1

	Edit /etc/modprobe.conf	>
		alias eth1 ipw2200
		options eth1 led=1

	modprobe ipw2200 debug=0x40000
	modprobe ipw2200 hwcrypto=0

	iwlist wlan0 scan | grep Frequency | sort | uniq -c | sort -n
	iwlist wlan0 scan | grep -C3 NETWORK_ID
	iwlist wlan0 scan | grep \(Channel
	
	iwconfig eth1 essid linksys channel 6 rate auto key hex_26_chars

=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

	# Set ethX to 100/ full duplex
	ethtool -s ethX autoneg off speed 100 duplex full

=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

leap / LEAP dependent and required packages
	c4eb-config-NetworkManager hostapd wpa_supplicant

	#example of /etc/xsupplicant/xsupplicant.conf
	#for LEAP protocol
	
	network_list = all
	#the list of networks to access
	
	default_netname = default
	#the default access network
	
	first_auth_command = <BEGIN_COMMAND>dhclient %i<END_COMMAND>
	#The command before authentication, which is usually used to get some info from
	#the network
	
	logfile = /var/log/xsupplicant.log
	#log file

	myssid #here is your network id, may be listed in the network list
	{
	  type = wireless
	  ssid = <BEGIN_SSID>myssid<END_SSID>
	  allow_types = all
	  identity = <BEGIN_ID>myid@domain.org<END_ID>
	  eap-leap {
	      username = <BEGIN_UNAME>myid@domain.org<END_UNAME>
	      password = <BEGIN_PASS>mypasswd<END_PASS>
	  }#setup for leap
	}

=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

vpnc - http://www.unix-ag.uni-kl.de/~massar/vpnc/
dependency:	libgcrypt	libgcrypt-devel
		libgpg-error	libgpg-error-devel
	ftp://ftp.gnupg.org/gcrypt/libgcrypt/

Dependency: libgcrypt
Install Guide:
	make > cp vpnc /usr/local/sbin > cp vpnc-disconnect /usr/local/sbin
	mkdir /etc/vpnc -> cp vpnc-script /etc/vpnc/
	mkdir /var/run/vpnc

Example /etc/vpnc.conf:
IPSec gateway 199.246.40.10
IPSec ID labuser
IPSec secret labuser
Xauth username someone
Domain torolab

=-=-


HHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHH
HHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHH

hadoop HADOOP distributed file system

	apt-get install sun-java6-jdk
	
	sudo addgroup hadoop
	sudo adduser --ingroup hadoop hadoop
	
	ssh-keygen -t rsa -P ""
	cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorzied_keys2
	
	# disable IPv6
	HADOOP_OPTS=-Djava.net.preferIPv4Stack=true
	
	sudo mkdir /opt/hadoop-datastore
	chown -R hadoop.hadoop /opt/hadoop-datastore
	chown -R hadoop.hadoop /opt/hadoop*
	
	tar -xzvf hadoop-0.17.0.tar.gz
	sudo mv hadoop-0.17.0 /opt
	
	cd /opt
	sudo ln -s hadoop-0.17.0/ hadoop
	
	configuration | Configuration
	config/hadoop-env.sh
	export JAVA_HOME=/usr/lib/jvm/java-6-sun
	
	config/*-site.xml

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
In file conf/core-site.xml:

<!-- In: conf/core-site.xml -->
<property>
  <name>hadoop.tmp.dir</name>
  <value>/your/path/to/hadoop/tmp/dir/hadoop-${user.name}</value>
  <description>A base for other temporary directories.</description>
</property>

<property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:54310</value>
  <description>The name of the default file system.  A URI whose
  scheme and authority determine the FileSystem implementation.  The
  uri's scheme determines the config property (fs.SCHEME.impl) naming
  the FileSystem implementation class.  The uri's authority is used to
  determine the host, port, etc. for a filesystem.</description>
</property>

In file conf/mapred-site.xml:

<!-- In: conf/mapred-site.xml -->
<property>
  <name>mapred.job.tracker</name>
  <value>localhost:54311</value>
  <description>The host and port that the MapReduce job tracker runs
  at.  If "local", then jobs are run in-process as a single map
  and reduce task.
  </description>
</property>

In file conf/hdfs-site.xml:

<!-- In: conf/hdfs-site.xml -->
<property>
  <name>dfs.replication</name>
  <value>1</value>
  <description>Default block replication.
  The actual number of replications can be specified when the file is created.
  The default is used if replication is not specified in create time.
  </description>
</property>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

	hadoop@ubuntu:~$ <HADOOP_INSTALL>/hadoop/bin/hadoop namenode -format
	hadoop@ubuntu:~$ <HADOOP_INSTALL>/bin/start-all.sh
	
	port to verify: 50030, 50040, 50070

HHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHH
HHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHH

hadoop cluster apache distributed file system hdfs
	install java / jvm
	sudo useradd hadoop; sudo passwd hadoop; create /home/hadoop/.ssh
	scp $MASTER_USER_SSH_PUB hadoop@hadoop:~/.ssh/authorized_keys2
	sudo cp hadoop*.tar.gz /opt; sudo ln -s hadoop-<VERSION> hadoop
	sudo chown -R hadoop.hadoop /opt/hadoop

	define JAVA_HOME in /opt/hadoop/conf

	define $HADOOP_INST/conf/hadoop-site.xml > update localhost

$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!-- Put site-specific property overrides in this file. -->

<configuration>

<property>
  <name>hadoop.tmp.dir</name>
  <value>/your/path/to/hadoop/tmp/dir/hadoop-${user.name}</value>
  <description>A base for other temporary directories.</description>
</property>

<property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:54310</value>
  <description>The name of the default file system.  A URI whose
  scheme and authority determine the FileSystem implementation.  The
  uri's scheme determines the config property (fs.SCHEME.impl) naming
  the FileSystem implementation class.  The uri's authority is used to
  determine the host, port, etc. for a filesystem.</description>
</property>

<property>
  <name>mapred.job.tracker</name>
  <value>localhost:54311</value>
  <description>The host and port that the MapReduce job tracker runs
  at.  If "local", then jobs are run in-process as a single map
  and reduce task.
  </description>
</property>

<property>
  <name>dfs.replication</name>
  <value>1</value>
  <description>Default block replication.
  The actual number of replications can be specified when the file is created.
  The default is used if replication is not specified in create time.
  </description>
</property>

</configuration>


VVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVV
VVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVV


Linux Virtualization | linux virtualization virt

kvm + amd64
	egrep '(vmx|svm)' --color=always /proc/cpuinfo
	grep ' lm ' /proc/cpuinfo

edit /etc/libvirt/qemu.conf > uncomment user = root; group = root to unlock permission

	apt-get install spice-client


create guest OS w/ kvm
	# create a qcow2 vm based on img
	qemu-img create -f qcow2 f18.img 10g

	# install
	qemu-kvm -m 1024 -cdrom f18.iso -drive file=f18.img,cache=none,if=virtio,index=0 -boot d -net nic -net user -nographic -vnc :0

	# launch
	qemu-kvm -m 1024 -drive file=f18.img,if=virtio,index=0 -boot c -net nic -net user -nographic -vnc :0

windows image on kvm w/ virtio
	qemu-img create -b winxp.img -f qcow2 -o cluster_size=2M winxp.qcow2
	virt-manager to launch vm w/ storage format= qcow2 cache_mode= none
	IDE Disk1 > Advanced options > Storage format= qcow2

enable virtio windows	http://www-01.ibm.com/support/docview.wss?uid=swg21587905
	option 1 > start  windows vm install with virtio iso mounted as a cdrom then choose virtio before install
	option 2 >
	virt-manager > install windows as usual and download virtio-win*.iso
	create a temp disk > qemu-img create -f raw tempdisk.img 10M
	add tempdisk to 1st disk that contains windows install w/ selecting Virtio Disk in Device Type and None in Cache Mode
	launch windows disk with 2nd (tempdisk) attached > install virtio driver wxp (disk) and xp (net)
	shutdown > remove 2 disks > re- add the 1st disk with virtio in device type and None in Cacne mode

virtio spec xml
	<disk type='file' device='disk'>
	<driver name='qemu' type='qcow2' cache='none'/>
	<source file='/pool/pool/kvm.dsk/xp_small.img.qcow2'/>
	<target dev='vda' bus='virtio'/>
	<address type='drive' controller='0' bus='0' target='0' unit='0'/>
	</disk>

	install spice & virtio driver into windows guest on kvm
	http://spice-space.org/download/binaries/spice-guest-tools/

virt-clone
	virt-clone -o [IMG_NAME] -n [NEW_NAME] -f /path/[NEW_NAME].img

virt network bridge
	brctl show	# list network

	# initiate a hypervisor session . --readonly (optional)
	virsh connect <name> --readonly

	virsh create configuration_file.xml
	virsh restore [filename]
	virsh shutdown [domain-id, dom name or uuid]
	virsh reboot [dom-id, do-name or uuid]

	virsh nodeinfo
	virsh list

sublimesublimesublime
    enable vi > Preferences > Setting-Default > change `"ignored_packages": ["Vintage"]` to `"ignored_packages": []`


sublimesublimesublime


UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU
UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU

script in /etc/rc.local
	echo 30 | sudo tee /sys/class/backlight/acpi_video0/brightness
	echo 3 | sudo tee /sys/devices/platform/applesmc.768/leds/smc::kbd_backlight/brightness

	/usr/bin/synclient TapButton3=2
	/usr/bin/xrandr --output eDP1 --mode 1680x1050

	sudo cryptsetup luksOpen /dev/sda7 pool; sudo mount /dev/mapper/pool /media/jeff/pool/

burn iso to usb		sudo dd if=/path/Fedora-I.iso of=/dev/sdb bs=8M

ubuntu 	# release	cat /etc/issue
	# receive key
	sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5A9BF3BB4E5E17B5
	wget -q -O /tmp/ocdc-keyring.deb http://ocdc.hursley.ibm.com/ocdc/ocdc-archive-keyring.deb; sudo dpkg -i /tmp/ocdc-keyring.deb;

package to install
	ibm-global-print openclient-config-vpnc

	gnome-shell ubuntu-gnome-desktop	# install gnome-shell

	skype icedtea-7-plugin openjdk-7-jre

	ubuntu-desktop p7zip gimp imagemagick chromium-browser pidgin qemu-system-x86 libvirt-bin ubuntu-vm-builder bridge-utils ubuntu-restricted-extras ssh virt-manager virt-viewer openconnect dconf-tools network-manager-vpnc vim cups-pdf lvm2 icedtea-plugin terminator cryptsetup

	gstreamer0.10-plugins-ugly gstreamer0.10-ffmpeg libxine1-ffmpeg gxine mencoder libdvdread4 totem-mozilla icedax tagtool easytag id3tool lame nautilus-script-audio-convert libmad0 mpg321 gstreamer1.0-libav vlc

chinese input method / im-config / im-choose

	fcitx fcitx-table-wbpy fcitx-googlepinyin

apt-get proxy
	apt-get install [PACKAGE] -o acquire::http::proxy="http://[IP]:[8085]"


	compizconfig-settings-manager compiz-fusion-plugins-extra
	ccsm > General > General Options > Desktop Size

	# install dvd codec / player repository on ubuntu
	https://help.ubuntu.com/community/RestrictedFormats
	sudo apt-get install ubuntu-restricted-extras
	sudo /usr/share/doc/libdvdread4/install-css.sh

	# nmon
	deb http://cz.archive.ubuntu.com/ubuntu lucid main universe

adduser
	sudo adduser ubuntu		# create /home/ubuntu
	sudo adduser ubuntu sudo	# add user ubuntu into sudoer group

apt-file	# file which package a particular file belongs to
	apt-get install apt-file
	apt-file update
	apt-file search /path/target_file

java
	apt-get install sun-java6-bin		# java
	apt-get install sun-java6-jdk
	update-java-alternatives -l	# list
	update-java-alternatives -s java-6-sun

java-sun in firefox plugin
	cd /usr/lib/firefox-addons/plugins
	sudo ln -s /usr/lib/jvm/java-6-sun/jre/lib/i386/libnpjp2.so .

internal repo / ibm repo / notes repo
	deb http://ocdc.hursley.ibm.com/ocdc raring-safe IBM IBM-layer
	deb-src http://ocdc.hursley.ibm.com/ocdc raring-safe IBM IBM-layer

	sudo apt-get install ocdc-repository; sudo apt-get update
	sudo dpkg-reconfigure ocdc-repository

#Security updates
deb http://security.ubuntu.com/ubuntu lucid-security main restricted universe multiverse

pidgin
	apt-get install pidgin	# sametime replacement
	cd ~/.purple; vi accouns.xml -> change fake_client_id = 1
	# unset libnotify to disable notification

	edit ~/.purple
		<setting name='server' type='string'>messaging.ibm.com</setting>
		<setting name='port' type='int'>1533</setting>
		<setting name='client_id_val' type='int'>5293</setting>
		<setting name='force_login' type='bool'>0</setting>
		<setting name='fake_client_id' type='bool'>1</setting>

touchpad	# disable touchpad
	apt-get install gsynaptics
	gconf-editor > desktop > gnome > perpherals > touchpad > off + touchpad_enabled

	synclient touchpaddoff=1
	synclient TapButton3=2 	# to enable 3rd button on Ubuntu 13.10 mac on ubuntu macmacmacmac

	xinput list

		 Virtual core pointer                      id=2    [master pointer  (3)]
		 Virtual core XTEST pointer                id=4    [slave  pointer  (2)]
		 SynPS/2 Synaptics TouchPad                id=12   [slave  pointer  (2)]
		 Virtual core keyboard                     id=3    [master keyboard (2)]
    		 Virtual core XTEST keyboard               id=5    [slave  keyboard (3)]
    		 Power Button                              id=6    [slave  keyboard (3)]
    		 Video Bus                                 id=7    [slave  keyboard (3)]
    		 Power Button                              id=8    [slave  keyboard (3)]
    		 Sleep Button                              id=9    [slave  keyboard (3)]
    		 Laptop_Integrated_Webcam_1.3M             id=10   [slave  keyboard (3)]
    		 AT Translated Set 2 keyboard              id=11   [slave  keyboard (3)]
    		 Dell WMI hotkeys                          id=13   [slave  keyboard (3)]

	xinput set-prop 12 "Device Enabled" 0

touchpad > enable 3rd button copy & paste
	synclient TapButton3=2 	# to enable 3rd button on Ubuntu 13.10 mac on ubuntu macmacmacmac
	echo synclient TapButton3=2 > ~/touchpad_settings.sh
	chmod +x ~/touchpad_settings.sh
	gsettings set org.gnome.settings-daemon.peripherals.input-devices hotplug-command "/home/user/touchpad_settings.sh"


wicd | wireless -> apt-get install wicd | replace of network-manager
install wpa_supplicant > vi /etc/wpa_supplicant/wpa_supplicant.conf
	network={
	        ssid="{NETWORK_ID}"
	        psk="{26_HEX_SECRET}"
	        priority=5
	        }
sudo wpa_supplicant -iwlan0 -c/etc/wpa_supplicant/wpa_supplicant.conf

update-rc.d (equivalent to chkconfig @ ubuntu) -> apt-get install rcconf
	update-rc.d vsftpd defaults 	(to restore)
	dpkg-reconfigure vsftpd 	(to restore)
	update-rc.d -f vsftpd remove	(to remove )

log into text mode >	update-rc.d -f gdm remove	#textmode
log into gui mode  > 	update-rc.d -f gdm defaults

log into text mode in karmic koala ubuntu text mode
	vi /etc/default/grub > change to "GRUB_CMDLINE_LINUX_DEFAULT="quiet text"" > sudo update-grub

tasksel --list-tasks
tasksel --install dns-server

change software source	> software-properties-gtk

	# accept key
	wget -q http://apt.wicd.net/wicd.gpg -O | sudo apt-get add -
	wget -q -O /tmp/ocdc-keyring.deb http://ocdc.hursley.ibm.com/ocdc/ocdc-archive-keyring.deb; sudo dpkg -i /tmp/ocdc-keyring.deb;

	dpkg -L package
	dpkg --contents package.deb

	dpkg -S /path/file	(what package owns /path/file)

	dpkg-reconfigure ibm-java-installer 	(reinstall ibm-java)

	dpkg --clear-avail	# fix dpkg cache corrupted

totem | movie-player
	increase the buffer
	gconf-editor > apps > totem > buffer-size = 20 | network-buffer-threshold = 20

linode br0 > edit /etc/network/interfaces
	auto lo
	iface lo inet loopback

	auto br0
	iface br0 inet dhcp
        	bridge_ports eth0
        	bridge_fd 9
        	bridge_hello 2
        	bridge_maxage 12
        	bridge_stp off

linode security
	Append "set -o vi" in /etc/bash.bashrc
	useradd -c "usr" -m -s "/bin/bash" usr -G sudo
	passwd usr; cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
	vi /etc/ssh/sshd_config # set PermitRootLogin no; /etc/init.d/ssh restart

openvpn	# create tls-auth key, then copy to /etc/openvpn and update /etc/openvpn/server.conf to reflect the change
	openvpn --genkey --secret static.key

openvpn /etc/openvpn/server.conf
	port 1195
	proto tcp
	tls-auth ta.key 0 # 0 on srv & 1 on client

openvpn --auth-user-pass /etc/openvpn/yegle/up --config /etc/openvpn/yegle/fremont-1-normal.ovpn

openvpn force all traffic from the client to get directed to the VPN server > edit server.conf
	push "redirect-gateway def1 bypass-dhcp"

	echo "1" > /proc/sys/net/ipv4/ip_forward
	iptables -t nat -A POSTROUTING -j MASQUERADE

rsyslog > /etc/rsyslog.conf > to avoid rate-limiting error @ /var/log/messages
	$SystemLogRateLimitInterval 2
	$SystemLogRateLimitBurst 50

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<G
virtualbox sun VirtualBox bluescreen

	HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Intelppm
	And changing the Start value to 4

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

Philips bluray region free setting
	no disc in tray > Press Home > Scroll to settings > 13893108520

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

3g wcdma on Ubuntu 10.04 w/ China Unicom
	sudo apt-get install usb-modeswitch ; reboot
	nm-connection-editor > Mobile Broadband > ... > APN: uninet PIN: 1234

sony vaio
Configure video driver + trackball @ /etc/default/grub

	GRUB_CMDLINE_LINUX_DEFAULT="quiet splash mem=1900mb nohz=off i8042.reset i8042.nomux i8042.nopnp i8042.noloop"
	
	sudo add-apt-repository ppa:gma500/ppa && sudo apt-get update
	
	sudo apt-get remove mplayer sudo apt-get install gnome-mplayer gecko-mediaplayer
	sudo apt-get install poulsbo-driver-2d poulsbo-driver-3d poulsbo-config
	
	sudo update-grub
sony vaio

=-=-=-=-=-=-=-=-=-=-=-=-=-

Tenda router configuration
192.168.2.1 / admin:admin

I9oyOv8SuplL	rcxasa
114.255.174.144

w3-connections https://lmc2.watson.ibm.com:15001/mobile

=-=-=-=-=-=-=-=-=-=-=-=-=-

Android device adb sdk

	yum install android-tools

	sudo vi /etc/udev/rules.d/51-android.rules > add the following
	SUBSYSTEM=="usb", SYSFS{idVendor}=="0bb4", SYMLINK+="android_adb", MODE="0666"

for Samsung Galaxy 3
	SUBSYSTEM=="usb", ATTR{idVendor}=="04e8", MODE="0666", GROUP="plugdev"

	sudo chmod a+r /etc/udev/rules.d/51-android.rules

reboot then adb device

	adb devices
	adb root
	adb shell

	flash clockwork recovery (CWM)
	dd if=/sdcard/cwmXXXrecovery.img of=/dev/block/mmcblk0p6 bs=4096

android sametime ewg1.artour.ibm.com:15001


macmacmacmac
ubuntu 13.10 saucy for mac
	symptom: boot hanging at smp
	solution: disable smp in /etc/default/grub

	symptom: slow boot
	solution: libata.force=1:noncq > append to /etc/default/grub

	install bcm driver on saucy (wifi network)
	sudo apt-get update
	sudo apt-get install bcmwl-kernel-source
	sudo modprobe wl

	audio internal speaker
	apt-get install alsa-tools
	hda-verb /dev/snd/hwC1D0 0x1 set_gpio_data 1

	switch fn (function) key -> f8= f8, not "play/pause"
	echo 2 | sudo tee /sys/module/hid_apple/parameters/fnmode

script in /etc/rc.local
        echo 30 | sudo tee /sys/class/backlight/acpi_video0/brightness
        echo 3 | sudo tee /sys/devices/platform/applesmc.768/leds/smc::kbd_backlight/brightness

        /usr/bin/synclient TapButton3=2
        /usr/bin/xrandr --output eDP1 --mode 1680x1050

macmacmacmac


FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF

Fedora fedora

create livecd from ubuntu
	apt-get install syslinux isomd5sum extlinux
	extract iso and copy LiveOS/livecd-iso-to-disk
	livecd-iso-to-disk --overlay-size-mb 512 /path/iso /path/usb
	dd if=/path/livecd.iso of=/dev/sdX bs=8M

repo setting > http://rpmfusion.org/Configuration/
	su -c 'yum localinstall --nogpgcheck http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-stable.noarch.rpm http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-stable.noarch.rpm'

chromium repo
	chromium-browser --proxy-server=192.168.101.2:31280

adobe flash >
	rpm -ivh http://linuxdownload.adobe.com/adobe-release/adobe-release-x86_64-1.0-1.noarch.rpm
	rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-adobe-linux
	yum install flash-plugin nspluginwrapper alsa-plugins-pulseaudio libcurl

repo fast repo
	yum install yum-plugin-fastestmirror

skype	yum install skype*.rpm , including glibc.i686 libXv.i686 alsa-lib.i686 libXScrnSaver.i686 qt-4.8.2-2.fc17.i686 qt-x11-4.8.2-2.fc17.i686

package to install
	kvm virt-manager libvirt p7zip gimp ImageMagick lvm2 xterm dconf-editor scim scim-pinyin ibus-pinyin chromium pidgin rdesktop vim rubygem-boxgrinder-build gnome-tweak-tool wget telnet spice-client anyconnect openssh terminator icedtea-web pv xorg-x11-drv-intel gpg powertop

multimedia driver gstreamer
	lsdvd libdvdnav libdvdread ffmpeg gstreamer-ffmpeg gstreamer-plugins-bad gstreamer-plugins-bad-extras gstreamer-plugins-ugly libdvdcss flash-plugin mplayer phonon-backend-gstreamer

notes repo for x64	http://ocfedora.hursley.ibm.com/fedora/1X/x86_64/	-> yum install ibm-notes-config


nvidia driver installed, to fix "cannot open font file true"
	sudo vi /etc/default/grub
	Look for "SYSFONT=True" and replace it with "SYSFONT=latarcyrheb-sun16"
	sudo grub2-mkconfig -o /boot/grub2/grub.cfg

ms truetype font
	wget "http://blog.andreas-haerter.com/_export/code/2011/07/01/install-msttcorefonts-fedora.sh?codeblock=1" -O "/tmp/install-msttcorefonts-fedora.sh"
	chmod a+rx "/tmp/install-msttcorefonts-fedora.sh"
	su -c "/tmp/install-msttcorefonts-fedora.sh"
	(option 2)
	sudo yum install liberation-fonts-common
	sudo yum install freetype-freeworld

show date @ menu panel dconf-editor
	Goto org-> gnome-> shell and click on clock.
	Enable/Tick/Check the "show-date".

multimedia driver gstreamer
	ffmpeg convert > ffmpeg -vcodec copy -i orig.ogv outfile.avi
	ffmpeg cut     > ffmpeg -vcodec copy -ss 00:00:00 -t 00:03:54 -i orig.ogv out.ogv

	mencoder file.rmvb -oac mp3lame -lameopts preset=128 -ovc lavc -lavcopts vcodec=mpeg4:vbitrate=1200 -ofps 25 -of avi -o file.avi

check dvd
	mplayer dvd://1 -frames 0 -identify
convert dvd into wav
	mplayer dvd://1 -aid 128 -vo null -ao pcm:file='filename.wav'


check system boot
	systemd-analyze
	systemd-analyze blame

	systemctl stop    bluetooth.service
	systemctl disable bluetooth.service

	sudo package-cleanup --orphans
	sudo package-cleanup --leaves

FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF

set primary monitor / screen
	xrandr --output LVDS1 --primary
	/usr/bin/xrandr --output eDP1 --mode 1920x1200

set touchpad 3rd button on mac
	synclient TapButton3=2


HSLTHSLTHSLTHSLT
	scp -r -p o StrictHostKeyChecking=no jeffyang@bejgsa.ibm.com:/gsa/bejgsa/projects/h/hslt/build/image/HSLT_dev/$BUILDNAME /home/jeff/Downloads/scratch/hslt/build/latest/
	
	ssh -L 9.123.127.201:33090:10.10.3.38:33090 10.10.3.38
	
	# recycle Hbase/ hbase
	# In all Hbase nodes
	kill -i `ps -ef | grep java | awk '{print $2}'`
	
	#Stop Hadoop on hbase-1
	/opt/IHC-*/bin/stop-dfs.sh
	
	#Start Hadoop on hbase-1
	/opt/IHC-*/bin/start-dfs.sh
	
	#Start Hbase on hbase-1
	/opt/hbase-*/bin/start-hbase.sh
	
	#Start RestServer on hbase-4
	/iaas/iaas-rest-srv/bin/rest_server.sh start
	
	#Restart ruby on Storage-1/2
	/iaas/storage_bots/rubybots/re-run.sh

=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

boxgrinder

sudo boxgrinder-build firstbox.appl -d libvirt --delivery-config connection_uri:qemu:///system,image_delivery_uri:/home/user/boxgrinder

appliance definition
	[jeff@jytpt410 boxgrinder]$ cat firstbox.appl
	name: rhel
	summary: rhel6.1
	os:
	  name: rhel
	  version: 6
	  password: passw0rd
	hardware:
	  cpus: 1
	  memory: 512
	  partitions:
	    "/":
	      size: 5
	repos:
	  - name: "base"
	    baseurl: "file:///mnt/rhel61_iso"

vim /boot/grub/grub.conf
console=ttyS0
	kernel /boot/vmlinuz-2.6.32-220.el6.x86_64 ... console=ttyS0

fedora performance | boot performance
	systemd-analyze time			# summarized
	systemd-analyze blame			# check the details
	systemd-analyze plot > boot-graph.png	# capture and gerate a graphy report

	systemctl --failed			# check the failed
	systemctl status systemd-tmpfiles-clean.service
	systemctl disable mdmonitor.service	# service to disable jetty.service sendmail.service ip6tables.service vmware-USBArbitrator.service
	yum remove abrt*

=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

tp-link wr703n hack
	power down > power up > hit "Reset" pin when seeing indicator blinks immediately >
	telnet 192.168.1.1 to get into failsafe mode > firstboot then reboot -f to flush all config

/etc/config/network

	config interface 'loopback'
	    option ifname 'lo'
	    option proto 'static'
	    option ipaddr '127.0.0.1'
	    option netmask '255.0.0.0'
	
	config interface 'lan'
	    option ifname 'eth0'
	    option type 'bridge'
	    option proto 'static'
	    option ipaddr '192.168.1.1'
	    option netmask '255.255.255.0'

	config interface 'wan'
	    option ifname 'wlan0'
	    option proto 'dhcp'

/etc/config/wireless

	config wifi-device  radio0
	    option type     mac80211
	    option channel  11
	    option hwmode   11ng
	    option path 'platform/ar933x_wmac'
	    option htmode   HT20
	    list ht_capab   SHORT-GI-20
	    list ht_capab   SHORT-GI-40
	    list ht_capab   RX-STBC1
	    list ht_capab   DSSS_CCK-40
	    # REMOVE THIS LINE TO ENABLE WIFI:
	    #option disabled 1
	
	config wifi-iface
	    option device   radio0
	    #option network  lan
	    option network  wan
	    #option mode     ap
	    option mode     sta
	    option ssid     'THE NAME OF OUR EXISTING WIFI NETWORK'
	    #option encryption none
	    option encryption wep+shared
	    option key 'WEP PASSWORD FOR OUR EXISTING WIFI NETWORK'

=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

gitgitgitgit
	git config --list

	git config --global core.editor vim
	git config --global merge.tool vimdiff

	git branch -rd origin/gh-pages		# delete remote branch
	git push origin --delete gh-pages	# delete remote branch

	# change/ update branch, you'd touch
	.git/config + .git/HEAD + .git/refs/heads/master

archarcharcharcharcharcharcharcharcharcharcharcharcharcharcharch
archarcharcharcharcharcharcharcharcharcharcharcharcharcharcharch
archarcharcharcharcharcharcharcharcharcharcharcharcharcharcharch
archarcharcharcharcharcharcharcharcharcharcharcharcharcharcharch

refresh key
	sudo pacman-key --init
	sudo pacman-key --refresh-keys
	sudo pacman-key --populate
	sudo pacman -S archlinux-keyring
	sudo pacman -Syu	# fresh package
	sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring

package manager optimize performance
	sudo packman -Sc && sudo pacman-optimize && sudo pacman -Syu

pacman optimize > https://wiki.archlinux.org/index.php/Pacman/Tips_and_tricks#Removing_unused_packages

pacman - list unused
	pacman -Qdttq

	list installed but not from base and base-devel
	pacman -Qei | awk '/^Name/ { name=$3 } /^Groups/ { if ( $3 != "base" && $3 != "base-devel" ) { print name } }'

pacman - remove unused
	sudo pacman -Rsn $(sudo pacman -Qdtq)
	sudo pacman -Rscnd <PACKAGE_NAME>

pacman - list installed from official repo
	sudo pacman -Qen
       - list installed from unofficial repo
	sudo pacman -Qem

	figure out a file being owned by which package
	pacman -Qo /usr/lib/libappindicator3.so.1.0.0

install package from pkgbuild
	git clone [package].git
	makepkg

ubuntu fonts
	pacman -U ttf-ubuntu-font-family-0.83-1-any.pkg.tar.xz

groupadd and useradd
	groupadd users
	useradd -m -g users -G wheel -s /bin/bash tuxinator

install	https://wiki.archlinux.org/index.php/Beginners'_guide
	parted /dev/sda ->
	mklabel msdos
	mkpart primary ext4 1MiB 512MiB
	set 1 boot on
	mkpart primary linux-swap 538MiB 8624MiB
	mkpart primary ext4 9053MiB 29533MiB

	lsblk /dev/sda
	mkfs.ext4 /dev/sda1; mkfs.ext4 /dev/sda3

	mkswap /dev/sda2
	swapon /dev/sda2

	mount /dev/sda3 /mnt
	mkdir -p /mnt/boot
	mount /dev/sda1 /mnt/boot

	pacstrap -i /mnt base base-devel

	genfstab -U /mnt > /mnt/etc/fstab
	arch-chroot /mnt /bin/bash

	locale-gen
	# Create /etc/locale.conf, add/ uncomment
	LANG=en_US.UTF-8

	tzselect
	ln -s /usr/share/zoneinfo/Zone/SubZone /etc/localtime
	hwclock --systohc --utc

	mkinitcpio -p linux

	pacman -S grub os-prober
	grub-install --recheck /dev/sda
	grub-mkconfig -o /boot/grub/grub.cfg

	systemctl enable dhcpcd@enp0s25.service; systemctl start dhcpcd@enp0s25.service

	pacman -S iw wpa_supplicant dialog

	passwd
	exit

	umount -R /mnt; reboot

	groupadd <USER>
	useradd -m -g <USER> -G wheel <USER>

installed package
	gnupg then sudo mkdir /root/.gnupg; touch .gnupg/dirmngr.conf

	sudo pacman-key --init
	sudo pacman-key --refresh-keys

after- install configuration
	gnupg gnome gnome-shell gnome-extra gdm gnome-disk-utility gnome-tweak-tool gnome-control-center gnome-backgrounds

	gimp geeqie vim libreoffice-fresh nautilus vlc chromium git firefox terminator openvpn openssh wget flashplugin java-runtime-common jre7-openjdk ebtables dnsmasq

	fcitx-googlepinyin fcitx-configtool fcitx-gtk2 fcitx-gtk3 fcitx-qt4 fcitx-qt5 fcitx-im adobe-source-han-sans-cn-fonts adobe-source-han-sans-tw-fonts opendesktop-fonts ttf-fantasque-sans-git ttf-liberation ttf-hack ttf-gentium ttf-fira-mono ttf-fira-sans

configure fcitx
	cat ~/.xprofile
	export GTK_IM_MODULE=fcitx
	export QT_IM_MODULE=fcitx
	export XMODIFIERS="@im=fcitx"

virtualization virtualisation virt-manager
	pacman -Syu ebtables dnsmasq
	systemctl start virtlogd.service; systemctl start libvirtd.service
	virt-manager

printer
	systemctl status org.cups.cupsd.service

start gdm
	sudo systemctl enable gdm.services

getlantern	network autoproxy http://127.0.0.1:16823/proxy_on.pac






```