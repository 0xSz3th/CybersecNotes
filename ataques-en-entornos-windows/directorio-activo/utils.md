# Utils

<pre class="language-bash"><code class="lang-bash"><strong># Mount a shared smb folder in linux
</strong>mount -t cifs //10.10.10.182/Data /mnt/data -o username=r.thompson,password=rY4n5eva,domain=cascade.local,rw

<strong># Enable RDP with admin creds
</strong><strong>crackmapexec smb (ip-machine) -u (admin-user) -p (admin-pass) -M rdp -o action=enable
</strong><strong>
</strong><strong># Crear servidor smb con kali
</strong>impacket-smbserver &#x3C;Name of the folder> &#x3C;path> -smb2support
impacket-smbserver smbFolder $(pwd) -smb2support
<strong>
</strong><strong># Error al descargar con (New-Object System.Net.WebClient).downloadString('someurl')
</strong>#Excepción al llamar a "DownloadString" con los argumentos "1": "Anulada la solicitud: No se puede crear un canal
#seguro SSL/TLS."
[Net.ServicePointManager]::SecurityProtocol = "tls12, tls11, tls" # Poner eso en la powershell

</code></pre>
