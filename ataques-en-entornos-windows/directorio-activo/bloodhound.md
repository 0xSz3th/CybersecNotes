# BloodHound

BloodHound uses graph theory to reveal the hidden and often unintended relationships within an Active Directory or Azure environment. Attackers can use BloodHound to easily identify highly complex attack paths that would otherwise be impossible to quickly identify. Defenders can use BloodHound to identify and eliminate those same attack paths. Both blue and red teams can use BloodHound to easily gain a deeper understanding of privilege relationships in an Active Directory or Azure environment.



<pre class="language-bash"><code class="lang-bash"><strong># Instalación y configuración
</strong><strong>apt-get install neo4j bloodhound -y
</strong>neo4j console     # Go to the localhost:port of neo4j and configure a user
bloodhound        # connect to the neo4j database

# bloodhound-python
apt-get install bloodhound.py
bloodhound-python -c All -u '&#x3C;USER>' -p '&#x3C;PASSWORD>' -d '&#x3C;DOMAIN>' -ns '&#x3C;IP-DC>'


</code></pre>





