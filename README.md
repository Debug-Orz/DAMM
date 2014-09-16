# DAMM
An open source memory analysis tool built on top of Volatility. It is meant as a proving ground for interesting new techniques to be made available to the community. These techniques are an attempt to speed up the investigation process through data reduction and codifying some expert knowledge. It currently supports the following features:

## Features
* ~30 Volaility plugins combined into ~20 DAMM plugins (e.g., pslist, psxview and other elements are combined into a 'processes' plugin)
* Can run multipe plugins in one invocation
* The option to store plugin results in sqlite databases for preservation or for "cached" analysis
* A filtering/type system that allows easily filtering on attributes like pids to see all information related to some process and exact or partial matchine for strings, etc.
* The ability to show the differences between two databases of results for the same or similar machines and manipulate for the cmdline how the differencing operates
* The ability to warn on certain types of suspicious behavior
* Output for terminal, tsv or grepable

## Usage
```
python damm.py -h
usage: damm.py [-h] [-d DIR] [-p PLUGIN [PLUGIN ...]] [-f FILE] [-k KDBG]
               [--db DB] [--profile PROFILE] [--debug] [--info] [--tsv]
               [--grepable] [--filter FILTER] [--filtertype FILTERTYPE]
               [--diff BASELINE] [-u FIELD [FIELD ...]] [--warnings] [-q]

DAMM v1.0 Beta

optional arguments:
  -h, --help            show this help message and exit
  -d DIR                Path to additional plugin directory
  -p PLUGIN [PLUGIN ...]
                        Plugin(s) to run. For a list of options use --info
  -f FILE               Memory image file to run plugin on
  -k KDBG               KDBG address for the images (in hex)
  --db DB               sqlite db file, for efficient input/output
  --profile PROFILE     volatility profile for the images (e.g. WinXPSP2x86)
  --debug               Print debugging statements
  --info                Print available volatility profiles, plugins
  --tsv                 Print screen formatted output.
  --grepable            Print in grepable text format
  --filter FILTER       Filter results on name:value pair, e.g., pid:42
  --filtertype FILTERTYPE
                        Filter match type; either "exact" or "partial",
                        defaults to partial
  --diff BASELINE       Diff the imageFile|db with this db file as a baseline
  -u FIELD [FIELD ...]  Use the specified fields to determine uniqueness of
                        memobjs when diffing
  --warnings            Look for suspicious objects.
  -q                    Query the supplied db (via --db).
```

### Supported plugins
apihooks callbacks connections devicetree dlls evtlogs handles idt injections messagehooks mftentries modules mutants privileges processes services sids timers

Plugins have attributes that can have types for filtering, e.g., processes:
(use --info to see for all plugins)
	offset
	name 		: string
	pid		: pid
	ppid		: pid
	image_path_name	: string
	command_line	: string
	create_time
	exit_time
	threads
	session_id
	handles
	is_wow64
	pslist
	psscan
	thrdproc
	pspcid
	csrss
	session
	deskthrd

### Example
Supply a profile as in Volatility, a memory image and a list of plugins to run (or 'all') to get tsv output:
```
python damm.py --profile WinXPSP2x86 -f memory.dmp -p processes
(or python damm.py --profile WinXPSP2x86 -f memory.dmp -p processes dlls modules)
(or python damm.py --profile WinXPSP2x86 -f memory.dmp -p all)

processes
offset	name	pid	ppid	image_path_name	command_line	create_time	exit_time	threads	session_id	handles	is_wow64	pslist	psscan	thrdproc	pspcid	csrss	session	deskthrd	
0x1983020	smss.exe	376	4	\SystemRoot\System32\smss.exe	\SystemRoot\System32\smss.exe	2013-10-31 17:21:24 UTC+0000	None	3		19	False	True	False	True	True	False	False	False
0x18eea50	VBoxService.exe	860	692			2013-10-31 17:21:26 UTC+0000	None	8			False	True	False	True	True	True	True	True	
0x18e4710	svchost.exe	904	692			2013-10-31 17:21:26 UTC+0000	None	17			False	True	False	True	True	True	True	True	
0x18fdd38	lsass.exe	704	648			2013-10-31 17:21:25 UTC+0000	None	22			False	True	False	True	True	True	True	True	
0x18ffa30	services.exe	692	648	C:\WINDOWS\system32\services.exe	C:\WINDOWS\system32\services.exe	2013-10-31 17:21:25 UTC+0000	None	14	0	241	False	True	False	True	True	TrueTrue	True	
0x183ed10	alg.exe	1888	692			2013-10-31 17:21:37 UTC+0000	None	6			False	True	False	True	True	True	True	True	
0x18b4648	svchost.exe	1080	692	C:\WINDOWS\System32\svchost.exe	C:\WINDOWS\System32\svchost.exe -k netsvcs	2013-10-31 17:21:26 UTC+0000	None	74	0	1126	False	True	False	True	True	TrueTrue	True	
0x18afc70	svchost.exe	1124	692	C:\WINDOWS\system32\svchost.exe	C:\WINDOWS\system32\svchost.exe -k NetworkService	2013-10-31 17:21:26 UTC+0000	None	4	0	57	False	True	False	True	TrueTrue	True	True	
0x18a7a60	svchost.exe	1152	692	C:\WINDOWS\system32\svchost.exe	C:\WINDOWS\system32\svchost.exe -k LocalService	2013-10-31 17:21:26 UTC+0000	None	13	0	193	False	True	False	True	True	TrueTrue	True	
0x1892490	VBoxTray.exe	1932	1636			2013-10-31 17:21:29 UTC+0000	None	7			False	True	False	True	True	True	True	True	
0x18c1710	svchost.exe	992	692			2013-10-31 17:21:26 UTC+0000	None	9			False	True	False	True	True	True	True	True	
0x195c2e0	pythonw.exe	1256	1940			2013-10-31 23:09:24 UTC+0000	None	5			False	True	False	True	True	True	True	True	
0x1877b38	MagicDisc.exe	1960	1636			2013-10-31 17:21:29 UTC+0000	None	1			False	True	False	True	True	True	True	True	
...
```

To make these results persist in a sqlite db, just supply a filename for the db:
```
python damm.py --profile WinXPSP2x86 -f memory.dmp -p processes --db my_results.db
```

This will print results to the terminal as well as store them in 'my_results.db'

To see the results again:
```
python damm.py -p processes --db my_results.db
```
(Note that you no longer need the memory image or to specify a profile, and the listing will come out pretty close to instantly regardless of how long the original processing took.)

If you later wish to see processes and other plugins:
```
python damm.py --profile WinXPSP2x86 -p processes dlls modules --db my_results.db
```
Will:  
1. consult the db for the 'processes' output
2. run the 'dlls' and 'modules' plugins
3. display the results
4. store the new results in the db 

### Differencing
To use the differencing engine, create 2 databases from 2 distinct memory images - such as one from before and one from after a piece of malware is executed
```
python damm.py --profile WinXPSP2x86-f before.dmp -p processes --db before.db
python damm.py --profile WinXPSP2x86 -f after.dmp -p processes --db after.db
```

Then use the --diff option for the baseline db
```
python damm.py -p processes --db after.db --diff before.db

Status	offset	name	pid	ppid	image_path_name	command_line	create_time	exit_time	threads	session_id	handles	is_wow64	pslist	psscan	thrdproc	pspcid	csrss	session	deskthrd	
New	0x1875718	tdl3	1344	1256	C:\DOCUME~1\jawauser\LOCALS~1\Temp\tdl3	"C:\DOCUME~1\jawauser\LOCALS~1\Temp\tdl3"	2013-10-31 23:09:25 UTC+0000	None	1	0	37	False	True	False	True	TrueTrue	True	True	
New	0x195bbf0	pythonw.exe	1256	1940	C:\Python27\pythonw.exe	C:\Python27\pythonw.exe C:\hkurkt\analyzer.py	2013-10-31 23:09:24 UTC+0000	None	5	0	114	False	True	False	True	True	TrueTrue	True	
New	0x182cda0	wpabaln.exe	1812	648	C:\WINDOWS\system32\wpabaln.exe	C:\WINDOWS\system32\wpabaln.exe	2013-10-31 23:10:13 UTC+0000	None	1	0	58	False	True	False	True	True	True	TrueTrue	
Changed	0x1877940	pythonw.exe	1940	1636	C:\Python27\pythonw.exe	"C:\Python27\pythonw.exe"  "C:\Documents and Settings\jawauser\Start Menu\Programs\Startup\agent.pyw" 	2013-10-31 17:21:29 UTC+0000	None	1	0	88->91	False	True	False	True	True	True	True	True	
Changed	0x190c020	csrss.exe	624	376	\??\C:\WINDOWS\system32\csrss.exe	C:\WINDOWS\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,3072,512 Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServerDllInitialization,3 ServerDll=winsrv:ConServerDllInitialization,2 ProfileControl=Off MaxRequestThreads=16	2013-10-31 17:21:25 UTC+0000	None	10	0	336->340	False	True	False	True	TrueFalse	True	True	
Changed	0x18e4020	svchost.exe	904	692	C:\WINDOWS\system32\svchost.exe	C:\WINDOWS\system32\svchost -k DcomLaunch	2013-10-31 17:21:26 UTC+0000	None	19->17	0	196->195	False	True	False	TrueTrue	True	True	True
...
```
The results look similar to the 'processes' plugin output above, but there are some differences:
* Only results that are new in 'after.db' or that are in both dbs but have some attributes that changed from 'before.db' are displayed (the output here is snipped).
* Results that are only in the 'after.db' have 'New' in the first ('Status') column
* Results in that have changed between the dbs have a 'Status' of 'Changed', and, importantly, denote the changes DAMM detected with '->': in the last line of output above the number of threads and handles has changed.

### Unique ID Manipulation
In order to determine which processes exist in both memory captures above, behind the scenes certain attributes of processes are used to make a unique identifier for each. For example, by default DAMM used the pid, ppid, name, and start time as the unique identifier of a process. This makes sense as these things are unlikey to (shouldn't? can't?) change over the life of the process, as opposed to attributes like the number of threads and handles, which change constantly. This default set works fine for comparisons of objects from memory images from the same boot of the same machine, but what about comparing across memory images taken from different boots of the machine? Or even other machines? The pid, and likely ppid will certainly not be the same, but the name, image path and command line should? 

Comparing a stock XPSP2x86 memory image with our image after some malware ran:
```
python damm.py -p processes --diff stock_WinXPSP2x86_processes.db --db after_malware.db

...
New     0x1874da0       explorer.exe    1636    1596    C:\WINDOWS\Explorer.EXE C:\WINDOWS\Explorer.EXE 2013-10-31 17:21:27 UTC+0000    None    12      0       316     False   True    False   True    True    True    True    True    
New     0x1983020       smss.exe        376     4       \SystemRoot\System32\smss.exe   \SystemRoot\System32\smss.exe   2013-10-31 17:21:24 UTC+0000    None    3               19      False   True    False   True    True    False   False
New     0x182cda0       wpabaln.exe     1812    648     C:\WINDOWS\system32\wpabaln.exe C:\WINDOWS\system32\wpabaln.exe 2013-10-31 23:10:13 UTC+0000    None    1       0       58      False   True    False   True    True    True    True
New     0x1883308       spoolsv.exe     1500    692     C:\WINDOWS\system32\spoolsv.exe C:\WINDOWS\system32\spoolsv.exe 2013-10-31 17:21:27 UTC+0000    None    14      0       113     False   True    False   True    True    True    True
Changed 0x1bcc830->0x1bcc9c8    System  4       0                       None    None    60->71          209->266        False   True    True->False     True    True    False   False   False   
```
Results in every process being tagged as 'New' (except System) becuase by default we use the pid and ppid to make the unique identifier.

Because of this, DAMM allows the user to specify which attributes of some object can be used to create a unique identifier. If we tell DAMM to just use process name, image_path_name, and command_line, we get much more reasonable results:
```
python damm.py -p processes --diff stock_WinXPSP2x86_processes.db --db after_malware.db -u name image_path_name command_line

Status  offset  name    pid     ppid    image_path_name command_line    create_time     exit_time       threads session_id      handles is_wow64        pslist  psscan  thrdproc        pspcid  csrss   session deskthrd        
New     0x1860020       wuauclt.exe     548     1080    C:\WINDOWS\system32\wuauclt.exe "C:\WINDOWS\system32\wuauclt.exe" /RunStoreAsComServer Local\[438]SUSDS109850d1d4659d4590c0302d99249922 2013-10-31 17:22:20 UTC+0000    None    7
New     0x1891da0       VBoxTray.exe    1932    1636    C:\WINDOWS\system32\VBoxTray.exe        "C:\WINDOWS\system32\VBoxTray.exe"      2013-10-31 17:21:29 UTC+0000    None    7       0       65      False   True    False   True    True
New     0x195bbf0       pythonw.exe     1256    1940    C:\Python27\pythonw.exe C:\Python27\pythonw.exe C:\hkurkt\analyzer.py   2013-10-31 23:09:24 UTC+0000    None    5       0       114     False   True    False   True    True    True
New     0x1877448       MagicDisc.exe   1960    1636    C:\Program Files\MagicDisc\MagicDisc.exe        "C:\Program Files\MagicDisc\MagicDisc.exe"      2013-10-31 17:21:29 UTC+0000    None    1       0       24      False   True    False
New     0x182cda0       wpabaln.exe     1812    648     C:\WINDOWS\system32\wpabaln.exe C:\WINDOWS\system32\wpabaln.exe 2013-10-31 23:10:13 UTC+0000    None    1       0       58      False   True    False   True    True    True    True
New     0x1877940       pythonw.exe     1940    1636    C:\Python27\pythonw.exe "C:\Python27\pythonw.exe"  "C:\Documents and Settings\jawauser\Start Menu\Programs\Startup\agent.pyw"   2013-10-31 17:21:29 UTC+0000    None    1       0
New     0x1875718       tdl3    1344    1256    C:\DOCUME~1\jawauser\LOCALS~1\Temp\tdl3 "C:\DOCUME~1\jawauser\LOCALS~1\Temp\tdl3"       2013-10-31 23:09:25 UTC+0000    None    1       0       37      False   True    False   True    True
New     0x18ee360       VBoxService.exe 860     692     C:\WINDOWS\system32\VBoxService.exe     system32\VBoxService.exe        2013-10-31 17:21:26 UTC+0000    None    8       0       106     False   True    False   True    True    True
Changed 0x1bcc830->0x1bcc9c8    System  4       0                       None    None    60->71          209->266        False   True    True->False     True    True    False   False   False   
Changed 0x18b7020->0x18b4648    svchost.exe     1076->1080      680->692        C:\WINDOWS\System32\svchost.exe C:\WINDOWS\System32\svchost.exe -k netsvcs      2011-09-26 01:33:36 UTC+0000->2013-10-31 17:21:26 UTC+0000      None    87->7
Changed 0x1994d08->0x18ffa30    services.exe    680->692        636->648        C:\WINDOWS\system32\services.exe        C:\WINDOWS\system32\services.exe        2011-09-26 01:33:35 UTC+0000->2013-10-31 17:21:25 UTC+0000      None    15->1
Changed 0x16a2cd0->0x18fd648    lsass.exe       692->704        636->648        C:\WINDOWS\system32\lsass.exe   C:\WINDOWS\system32\lsass.exe   2011-09-26 01:33:35 UTC+0000->2013-10-31 17:21:25 UTC+0000      None    24->22  0       356->
Changed 0x1aefda0->0x1983020    smss.exe        384->376        4       \SystemRoot\System32\smss.exe   \SystemRoot\System32\smss.exe   2011-09-26 01:33:32 UTC+0000->2013-10-31 17:21:24 UTC+0000      None    3               19      False
Changed 0x189a1d0->0x18a7a60    svchost.exe     1336->1152      680->692        C:\WINDOWS\system32\svchost.exe C:\WINDOWS\system32\svchost.exe -k LocalService 2011-09-26 01:33:37 UTC+0000->2013-10-31 17:21:26 UTC+0000      None    14->1
Changed 0x16c9b40->0x1914aa8    winlogon.exe    636->648        384->376        \??\C:\WINDOWS\system32\winlogon.exe    winlogon.exe    2011-09-26 01:33:35 UTC+0000->2013-10-31 17:21:25 UTC+0000      None    16->18  0       498->509
Changed 0x1ab5248->0x18c1020    svchost.exe     944->992        680->692        C:\WINDOWS\system32\svchost.exe C:\WINDOWS\system32\svchost -k rpcss    2011-09-26 01:33:36 UTC+0000->2013-10-31 17:21:26 UTC+0000      None    11->9   0
Changed 0x14b03e0->0x183e620    alg.exe 2272->1888      680->692        C:\WINDOWS\System32\alg.exe     C:\WINDOWS\System32\alg.exe     2011-09-26 01:33:55 UTC+0000->2013-10-31 17:21:37 UTC+0000      None    7->6    0       112->105
Changed 0x1af5cd0->0x1874da0    explorer.exe    1752->1636      1696->1596      C:\WINDOWS\Explorer.EXE C:\WINDOWS\Explorer.EXE 2011-09-26 01:33:45 UTC+0000->2013-10-31 17:21:27 UTC+0000      None    32->12  0       680->316        False
Changed 0x1670020->0x18e4020    svchost.exe     868->904        680->692        C:\WINDOWS\system32\svchost.exe C:\WINDOWS\system32\svchost -k DcomLaunch       2011-09-26 01:33:35 UTC+0000->2013-10-31 17:21:26 UTC+0000      None    17
Changed 0x15685e0->0x1883308    spoolsv.exe     1516->1500      680->692        C:\WINDOWS\system32\spoolsv.exe C:\WINDOWS\system32\spoolsv.exe 2011-09-26 01:33:39 UTC+0000->2013-10-31 17:21:27 UTC+0000      None    14      0       159->
Changed 0x1816ab8->0x190c020    csrss.exe       612->624        384->376        \??\C:\WINDOWS\system32\csrss.exe       C:\WINDOWS\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,3072,512 Windows=On SubSystemType=Windows S
Changed 0x19f7548->0x18afc70    svchost.exe     1200->1124      680->692        C:\WINDOWS\system32\svchost.exe C:\WINDOWS\system32\svchost.exe -k NetworkService       2011-09-26 01:33:37 UTC+0000->2013-10-31 17:21:26 UTC+0000      None
```
DAMM now identifies fewer processes as 'New' (including the malware process), allowing the investigator to focus their efforts on these processes.

### Filtering

With all plugins run on a small memory sample, we get ~14,000 memory objects: processes, dlls, modules, etc. What if we have already identified some process or string of interest? Grep can be probematic, especially when searching for pids, so DAMM includes a simple type and filtering system. To filter on objects that have a pid attribute of a certain value:
```
python damm.py -p processes dlls connections handles after_malware.db --filter pid:1344

processes
offset	name	pid	ppid	image_path_name	command_line	create_time	exit_time	threads	session_id	handles	is_wow64	pslist	psscan	thrdproc	pspcid	csrss	session	deskthrd	
0x1875718	tdl3	1344	1256	C:\DOCUME~1\jawauser\LOCALS~1\Temp\tdl3	"C:\DOCUME~1\jawauser\LOCALS~1\Temp\tdl3"	2013-10-31 23:09:25 UTC+0000	None	1	0	37	False	True	False	True	True	TrueTrue	True

dlls
proc_pid	dll_base	size_of_image	load_count	full_dll_name	
1344	0x73000000	155648	0x1	C:\WINDOWS\system32\WINSPOOL.DRV	
1344	0x400000	77824	0xffff	C:\DOCUME~1\jawauser\LOCALS~1\Temp\tdl3	
1344	0x77f10000	299008	0xffff	C:\WINDOWS\system32\GDI32.dll	
1344	0x7e410000	593920	0xffff	C:\WINDOWS\system32\user32.dll	
...

connections
offset	pid	local_ip	local_port	remote_ip	remote_port	allocated	
0x1853580	1344	192.168.56.101	1035	192.168.43.171	2042	True	

handles
offset	pid	handle_value	granted_access	object_type	name	
0xe1007ff0	1344	0x4	0xf0003	KeyedEvent	CritSecOutOfMemoryEvent	
0x81a43310	1344	0x3c	0x1f03ff	Thread	TID 384 PID 1344	
0x81902878	1344	0x6c	0x1f01ff	File	\Device\Tcp	
0x81902240	1344	0xc	0x100020	File	\Device\HarddiskVolume1\DOCUME~1\jawauser\LOCALS~1\Temp			
0x81906158	1344	0x38	0x1f0003	Semaphore	shell.{A48F1A32-A340-11D1-BC6B-00A0C90312E1}	
0x81a43310	1344	0x64	0x1f03ff	Thread	TID 384 PID 1344	
0x81851900	1344	0x68	0x1f01ff	File	\Device\Afd\Endpoint	
0x81953f78	1344	0x20	0xf01ff	Desktop	Default	
0xe106b648	1344	0x44	0xf003f	Key	MACHINE\SYSTEM\CONTROLSET001\SERVICES\WINSOCK2\PARAMETERS\PROTOCOL_CATALOG9	
0x81902950	1344	0x70	0x1f01ff	File	\Device\Tcp	
0x81902cd0	1344	0x7c	0x100001	File	\Device\KsecDD	
0xe1aa0ca0	1344	0x80	0x2001f	Key	USER\S-1-5-21-1644491937-789336058-854245398-1003\SOFTWARE\MICROSOFT\WINDOWS\CURRENTVERSION\INTERNET SETTINGS	
(many lines removed for brevity)
```
This can give a nice overview of the objects associated with a process. 

Even more powerful, diff and filtering can be used in conjunction. I have a memory sample from before a tdl3 infection and one after. Searching for thew string 'tdl' in the before db results in ~600 hits. In the after infection db, there are ~730 hits. (Note that ntdll.dll contain the string tdl.) Using diff and filtering in conjunction as below results in only ~180 hits - a significant reduction. 
```
python damm.py -p all --diff before_tdl3.db --db after_tdl3.db --filter string:tdl --filtertype partial > string_tdl_diff.txt
```

### Warnings
In an attempt to make the traige process even easier, DAMM has a prototype warning system built in to sniff out signs of danger. For the moment it just checks some process parent/child relationships, but the intent is to expand upon this considerably.
```
python damm.py -p all --db after_tdl3.db  --warnings

< actual processes output removed >

Warnings:
svchost.exe (pid: 904) parent process is services.exe as expected.
lsass.exe (pid: 704) parent process is winlogon.exe as expected.
svchost.exe (pid: 1152) parent process is services.exe as expected.
services.exe (pid: 692) parent process is winlogon.exe as expected.
svchost.exe (pid: 1080) parent process is services.exe as expected.
System pid is 4 as expected.
...
```