Consider a scenario where we are monitoring the number of network connections initiated by a process within a certain time frame.
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3 | bin _time span=1h | stats count as NetworkConnections by _time, Image | streamstats time_window=24h avg(NetworkConnections) as avg stdev(NetworkConnections) as stdev by Image | eval isOutlier=if(NetworkConnections > (avg + (0.5*stdev)), 1, 0) | search isOutlier=1
```
This is grouping the results into hourly intervals and calculating the number of connections per Image. Then stramstats calculates a rolling average and the standard deviation of that number over 1 day period for each unique process image. Then eval creates a new field and assigns it to 1 for any event where the number of connections is more than 0.5 standard deviations away from the average. Finally it filters out when it doens't.

## Crafting SPL Searches Based on Analytics

Here are some examples that follow this approach.

1. Detection of abnormally long commands
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" Image=*cmd.exe | eval len=len(CommandLine) | table User, len, CommandLine | sort - len
```
We quickly find out that we got some benign activity, so let's filter it out.
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" Image=*cmd.exe ParentImage!="*msiexec.exe" ParentImage!="*explorer.exe" | eval len=len(CommandLine) | table User, len, CommandLine | sort - len
```

2. Detection of abnormal cmd.exe activity
This is somewhat close to the scenario in the beginning.
```shell-session
index="main" EventCode=1 (CommandLine="*cmd.exe*") | bucket _time span=1h | stats count as cmdCount by _time User CommandLine | eventstats avg(cmdCount) as avg stdev(cmdCount) as stdev | eval isOutlier=if(cmdCount > avg+1.5*stdev, 1, 0) | search isOutlier=1
```

3. Detection of processes loading a high number of DLLs in a specific time
```shell-session
index="main" EventCode=7 | bucket _time span=1h | stats dc(ImageLoaded) as unique_dlls_loaded by _time, Image | where unique_dlls_loaded > 3 | stats count by Image, unique_dlls_loaded
```
Again, we see some benign activity.
```shell-session
index="main" EventCode=7 NOT (Image="C:\\Windows\\System32*") NOT (Image="C:\\Program Files (x86)*") NOT (Image="C:\\Program Files*") NOT (Image="C:\\ProgramData*") NOT (Image="C:\\Users\\waldo\\AppData*")| bucket _time span=1h | stats dc(ImageLoaded) as unique_dlls_loaded by _time, Image | where unique_dlls_loaded > 3 | stats count by Image, unique_dlls_loaded | sort - unique_dlls_loaded
```
The command _dc_ calculates the distinct count of DLLs loaded.

4. Detection of transactions where the same process has been created more than once on the same computer.
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | transaction ComputerName, Image | where mvcount(ProcessGuid) > 1 | stats count by Image, ParentImage
```
The *transaction* command is used to group related events, in this case, events that share their ComputerName and Image.
The relationship between _rundll32.exe_ and _svchost.exe_ has the highest count number, so let's dive deeper into it.
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | transaction ComputerName, Image | where mvcount(ProcessGuid) > 1 | search Image="C:\\Windows\\System32\\rundll32.exe" ParentImage="C:\\Windows\\System32\\svchost.exe" | table CommandLine, ParentCommandLine
```
