# Wordpress

#### User enumeration

```bash
wpscan --enumeration u --url <url>
curl -s -X GET "http://<url>/wp-json/wp/v2/users/"
```





#### RCE via edit themes

Una vez logeados en el wp-admin, nos vamos a themes->theme editor->

1. &#x20;404.php

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Y generar un 404 error modificando la url:

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

2. Modificar el header.php

<figure><img src="../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>



