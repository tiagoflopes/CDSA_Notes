I need to see some logs.
## 1
find a dll hijacking
set up sysmon
filter event id 7
find wininet.dll
_redacted_ shouldn't be running that
done

## 2
find an unmannaged powershell code running
filter event id 7
find clr.dll
_redacted_ outside system32 running it
done

## 3
find process that injected to injected process lmao
got the pid of the injected process: _redacted_
find that
_redacted_ conatins as the TargetProcessId that same id
done

## 4
find an event with targetimage being the lsass.exe and source some executable outside system32
found _redacted_
done

## 5
find logon attempt after _redacted_
no events to that date
done

## 6
find weird parent-child process relationship
find cmd.exe
found _redacted_ which parent is cmd.exe
done