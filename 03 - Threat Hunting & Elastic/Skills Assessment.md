## Hunt 1 - Lateral Tool Transfer

`(event.code : 1 or event.code : 3) and "C:\Users\Public"`
The tool _redacted_ is being used by _redacted_ to perform lateral movement.

## Hunt 2 - Registry Run Keys

filter: registry.value: exists (this eliminates the need to filter for event.code 12 or 13)
query: `"Run"` (for the name of the registry key)

## Hunt 3 - PowerShell Remoting

`powershell.file.script_block_text : "Enter-pssession dc1"`
Only 1 result.
