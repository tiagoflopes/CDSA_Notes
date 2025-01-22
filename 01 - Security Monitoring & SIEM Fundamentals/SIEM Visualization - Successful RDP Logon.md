We know that service accounts start with "svc-".
This accounts often possess exceptionally high privileges.
Our visualization will be based on the Windows event log 4624 - An account was successfully logged on, and the logon type should be "RemoteInteractive".
The final table looks like this:
![[logon_rdp.png]]
The columns are:
1. The service account whose credentials generated the successful RDP logon attempt event.
2. The machine on which the logon attempt occurred.
3. The IP of the machine that initiated the logon attempt.
4. The number of times the event has occurred (based on the specified time frame or the entire data set, depending on the settings).