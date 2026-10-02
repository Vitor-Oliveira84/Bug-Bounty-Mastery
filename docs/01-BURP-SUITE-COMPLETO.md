# 🔧 BURP SUITE - Guia Completo & Detalhado

Dominar Burp Suite é essencial para bug bounty. Este guia cobre TUDO.

---

## 📥 INSTALAÇÃO & SETUP INICIAL

### 1.1 Baixar Burp Suite

**Opção 1: Community (Gratuita - Recomendada para começar)**
```
https://portswigger.net/burp/communitydownload

Funcionalidades:
✅ Proxy (interceptar requisições)
✅ Repeater (teste manual)
✅ Intruder (força bruta)
✅ Decoder (encode/decode)
✅ Comparer (comparar respostas)
✅ Sequencer (análise de aleatoriedade)
❌ Scanner automático (limitado)
❌ Colaborator (detectar interações)
```

**Opção 2: Professional (Paga - $399/ano)**
```
✅ Tudo da Community
✅ Scanner ativo completo
✅ Colaborator (excelente para SSRF/XXE)
✅ Macro e automação avançada
✅ Extensões premium
```

### 1.2 Instalação (Windows, Mac, Linux)

```bash
# Windows
1. Download burpsuite_community_windows.exe
2. Duplo clique
3. Próximo → Próximo → Instalar
4. Abrir Burp

# Linux
1. Download burpsuite_community_linux_v*.sh
2. chmod +x burpsuite_community_linux_v*.sh
3. ./burpsuite_community_linux_v*.sh
4. Seguir installer

# Mac
1. Download Burp Suite Community.dmg
2. Arrastar para Applications
3. Abrir (pode pedir permissão)
```

### 1.3 Configuração Inicial

Ao abrir Burp pela primeira vez:

```
1. Aceitar license
2. Escolher projeto: "Temporary project"
3. Configuração padrão (OK)
4. Burp vai abrir
```

---

## 🌐 PROXY - Interceptar Requisições

### 2.1 O QUE É?

O Proxy do Burp Suite intercepta TODO tráfego HTTP/HTTPS do seu navegador, permitindo:
- ✅ Ver requisições antes de enviar
- ✅ Modificar dados
- ✅ Descartar requisições
- ✅ Repetir requisições
- ✅ Analisar respostas

### 2.2 SETUP PASSO-A-PASSO

#### PASSO 1: Configurar Listener

1. Abra Burp Suite
2. Vá para **Proxy** → **Settings**
3. Clique em **Proxy listeners**
4. Clique em **Add**
5. Preencha:
   ```
   Bind to port: 8080
   Bind to address: 127.0.0.1
   ✅ Running
   ```
6. Clique **OK**

**Resultado:** Burp está escutando em http://127.0.0.1:8080

#### PASSO 2: Configurar Navegador

**CHROME:**
1. Abra Chrome
2. Configurações → Avançado → Sistema
3. Abra "Configurações de proxy"
4. Configurações de LAN → Proxy:
   ```
   Endereço: 127.0.0.1
   Porta: 8080
   ✅ Usar proxy para LAN
   ```

**FIREFOX:**
1. Abra Firefox
2. Preferências → Rede → Configurações
3. Proxy HTTP:
   ```
   Endereço: 127.0.0.1
   Porta: 8080
   ✅ Usar proxy para conexões HTTP
   ```

#### PASSO 3: Certificado SSL

Para interceptar HTTPS, Burp precisa de seu certificado:

1. Acesse http://burp/cert (com proxy ativo)
2. Baixe **cacert.der**
3. Importe no navegador:

**Chrome:**
```
Configurações → Privacidade e segurança → Segurança
Gerenciar certificados → Importar
Selecione cacert.der
Confiar em "PortSwigger" para certificados
```

**Firefox:**
```
Preferências → Privacidade → Certificados
Ver certificados → Importar
Selecione cacert.der
✅ Confiar neste certificado
```

### 2.3 USAR O PROXY

#### Interceptar Requisição

```
1. Vá para Proxy → Intercept
2. Clique em "Intercept is off" → mude para "Intercept is on"
3. Navegue no site (requisição vai pausar)
4. A requisição aparecerá em Intercept tab
5. Você pode:
   ├── [Forward] - Enviar para servidor
   ├── [Drop] - Descartar
   ├── [Responder] - Responder manualmente
   └── [Edit] - Modificar antes de enviar
```

#### EXEMPLO PRÁTICO: Bypass de Senha

```
Interceptar requisição de login:

POST /login HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

username=admin&password=teste

MODIFICAR PARA:

username=admin' OR '1'='1 -- &password=qualquercoisa

Clicar [Forward] → Enviar com payload
```

### 2.4 MATCH & REPLACE (Automático)

Para modificar automaticamente requisições:

```
1. Proxy → Options → Match and Replace
2. Clique "Add"
3. Type: Request body
4. Match: (cookie ou dado que quer mudar)
5. Replace: (novo valor)
✅ Enabled
```

**EXEMPLO: Sempre enviar session válida**

```
Type: Request header
Match: Cookie:.*
Replace: Cookie: PHPSESSID=abc123def456
```

---

## 🔄 REPEATER - Teste Manual

### 3.1 O QUE É?

Permite enviar requisições manualmente e ver respostas, modificando dados livremente.

### 3.2 COMO USAR

#### PASSO 1: Enviar para Repeater

```
1. Intercepte uma requisição
2. Clique direito → "Send to Repeater"
3. Ou arraste para aba Repeater
```

#### PASSO 2: Modificar & Testar

```
No Repeater:
1. Mude o payload na requisição
2. Clique [Send]
3. Veja resposta à direita
4. Repita conforme necessário
```

### 3.3 EXEMPLOS DE TESTES

#### Exemplo 1: IDOR (Insecure Direct Object Reference)

```
Requisição original:
GET /api/users/123/profile HTTP/1.1

Teste 1:
GET /api/users/124/profile HTTP/1.1
[Send] → Ver resposta (é outro usuário?)

Teste 2:
GET /api/users/999/profile HTTP/1.1
[Send] → Ver resposta

Teste 3:
GET /api/users/1/profile HTTP/1.1
[Send] → Ver resposta (admin?)

✅ Se retorna dados de outros usuários = IDOR encontrado!
```

#### Exemplo 2: SQL Injection em GET

```
Original:
GET /products?id=1 HTTP/1.1

Teste 1:
GET /products?id=1' HTTP/1.1
[Send] → Erro SQL? (SQL injection provável)

Teste 2:
GET /products?id=1 AND 1=1 HTTP/1.1
[Send] → Retorna dados?

Teste 3:
GET /products?id=1 UNION SELECT NULL,NULL,NULL -- HTTP/1.1
[Send] → Retorna resultado?

✅ Se aparecer mensagens de erro SQL = SQL Injection!
```

#### Exemplo 3: XSS Refletido

```
Original:
GET /search?q=test HTTP/1.1

Teste 1:
GET /search?q=<script>alert('XSS')</script> HTTP/1.1
[Send] → Clique no link de resposta
         → Script executa? = XSS encontrado!

Teste 2 (se filtro bloqueia):
GET /search?q=<img%20src=x%20onerror=alert('XSS')> HTTP/1.1
[Send] → Teste

Teste 3 (dupla codificação):
GET /search?q=%253Cscript%253Ealert('XSS')%253C/script%253E HTTP/1.1
```

### 3.4 DICAS IMPORTANTES

**Decodificar URLs automaticamente:**
```
1. Abra Tabs → Request/Response
2. Clique em "Render" para ver HTML renderizado
3. Ou "Raw" para ver código bruto
```

**Copiar requisição como cURL:**
```
Clique direito na requisição → Copy as curl command
Cola no terminal para testar fora do Burp
```

---

## 💣 INTRUDER - Força Bruta & Fuzzing

### 4.1 O QUE É?

Automatiza testes repetitivos com payloads, perfeito para:
- Força bruta de senhas
- Fuzzing de parâmetros
- Teste de múltiplos IDs (IDOR)
- Descobrir padrões

### 4.2 SETUP COMPLETO

#### PASSO 1: Enviar para Intruder

```
1. Intercepte requisição
2. Clique direito → "Send to Intruder"
3. Vá para aba Intruder
```

#### PASSO 2: Marcar Payloads

Na aba "Positions":

```
Exemplo com parâmetro ID:
GET /api/users/§123§/profile HTTP/1.1

Os § § marcam o local do payload

Tipos de ataque:
├── Sniper (um parâmetro)
├── Battering ram (mesmo payload em vários)
├── Pitchfork (payloads diferentes por posição)
└── Cluster bomb (combinações de payloads)
```

#### PASSO 3: Preparar Payloads

Na aba "Payloads":

```
Payload type: Simple list
Payload options:
  1
  2
  3
  4
  5
  10
  100
  999
  admin
  root
```

#### PASSO 4: Executar

```
1. Clique "Start attack"
2. Acompanhe progresso
3. Analise "Response length", "Status code"
4. Clique em itens interessantes para ver resposta
```

### 4.3 EXEMPLO COMPLETO: IDOR com Intruder

**Objetivo:** Testar se posso acessar perfis de outros usuários

**PASSO 1: Requisição base**
```
GET /api/profile?user_id=123 HTTP/1.1
```

**PASSO 2: Marcar payload**
```
GET /api/profile?user_id=§123§ HTTP/1.1
```

**PASSO 3: Lista de payloads**
```
1
2
3
4
5
10
50
100
1000
admin
test
root
```

**PASSO 4: Executar**
```
[Start Attack]
Resultados:
  ID 1: Status 200, Length 2500 ← Admin?
  ID 2: Status 403, Length 50 (acesso negado)
  ID 3: Status 200, Length 2300 ← Outro user
  ...
```

**PASSO 5: Investigar**
```
Clique em ID 1:
  - Ver resposta completa
  - Tem informações sensíveis?
  - Pode ser IDOR de privilégio alto!
```

### 4.4 DICAS DE PAYLOADS

**Descobrir limite de usuários:**
```
1. Comece com 1, 2, 3
2. Se funciona, suba para 10, 100
3. Se retorna erro em 1000, máximo é 999
4. Teste: 1-999, admin, test, null, -1
```

**Payload para fuzzing genérico:**
```
Download: /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
Ou usar: SecLists do GitHub
```

---

## 🔐 DECODER - Análise de Dados

### 5.1 O QUE É?

Encode/decode dados em diferentes formatos.

### 5.2 FORMATOS SUPORTADOS

```
URL encoding:        %20 = espaço
Base64:              YWRtaW4= = admin
Hex:                 61646d696e = admin
HTML entities:       &lt; = <
JavaScript:          \x61\x64\x6d\x69\x6e
Unicode:             admin
```

### 5.3 EXEMPLOS

**Exemplo 1: Decodificar Base64**

```
Entrada: dXNlcl9pZD0xMjM=
Clique Decoder
Selecione "Base64" → "Decode"
Saída: user_id=123
```

**Exemplo 2: URL Encoding**

```
Entrada: admin%27%20OR%20%271%27%3D%271
Resultado decodificado: admin' OR '1'='1
```

**Exemplo 3: Cadeia de decodificação**

```
Data: YWRtaW4nIE9SICcxJz0nMQ==

1. Base64 decode → admin' OR '1'='1
2. URL encode → admin%27%20OR%20%271%27%3D%271
3. HTML encode → admin&apos; OR &apos;1&apos;=&apos;1
```

---

## 🔀 COMPARER - Comparar Respostas

### 6.1 O QUE É?

Compara duas requisições/respostas lado-a-lado.

### 6.2 COMO USAR

```
1. Envie duas requisições para Repeater
2. Clique direito na primeira → "Send to Comparer"
3. Clique direito na segunda → "Send to Comparer"
4. Aba "Comparer" mostra lado-a-lado
5. Diferenças destacadas
```

### 6.3 CASOS DE USO

**Caso 1: Testar se dois usuários retornam dados diferentes**

```
Requisição 1: GET /profile?id=1 (seu perfil)
  Resposta: nome=João, email=joao@example.com

Requisição 2: GET /profile?id=2 (outro perfil)
  Resposta: nome=Maria, email=maria@example.com

Comparer → Ver diferenças:
  ✅ Dados diferentes = Acesso funcionando
  ❌ Mesmo erro = Não consegue acessar (protegido)
```

**Caso 2: Encontrar diferenças entre versões**

```
Teste com admin=true:   Resposta A
Teste com admin=false:  Resposta B

Comparer → Ver diferenças:
  Mudou ícone? Layout? Menus? = Pode indicar acesso elevado
```

---

## 🎲 SEQUENCER - Análise de Aleatoriedade

### 7.1 O QUE É?

Analisa tokens/cookies para determinar se são realmente aleatórios.

### 7.2 COMO USAR

```
1. Intercepte requisição com token/cookie
2. Clique direito no token → "Send to Sequencer"
3. Clique "Start live capture"
4. Espera Burp coletar 100+ tokens
5. Clique "Analyze now"
```

### 7.3 INTERPRETAR RESULTADOS

```
Randomness: 100%
├── Excelente (seguro)

Randomness: 50-99%
├── Bom (seguro)

Randomness: 10-50%
├── Fraco (pode quebrar)

Randomness: <10%
├── Muito fraco (vulnerável!)
```

**Se Sequencer detectar padrão:**
```
✅ Pode ser possível prever tokens
✅ Pode fazer força bruta
✅ Pode falsificar sessões
```

---

## 🤖 SCANNER - Detecção Automática

### 8.1 PASSIVE SCANNING (Sempre ativo)

```
Análise de requisições/respostas sem enviar payloads:
✅ Procura headers ausentes
✅ Identifica tecnologias
✅ Detecta informações sensíveis
✅ Não deixa rastros no servidor
```

**Para ativar:**
```
Dashboard → Proxy settings
✅ Passive scanning is enabled
```

### 8.2 ACTIVE SCANNING (Só Community limitado)

```
Envia payloads de teste ao servidor:
✅ Teste SQL injection
✅ Teste XSS
✅ Teste XXE
⚠️ Deixa rastros - NUNCA em produção!
⚠️ Community é muito limitado
```

**Para usar:**
```
1. Scope → Include in scope (adicione URL)
2. Dashboard → Scan
3. Clique "New scan"
4. Escolha "Crawl and audit"
5. Aguarde resultados
```

---

## 🔧 MACROS - Automação

### 9.1 O QUE SÃO?

Sequências automáticas de requisições.

### 9.2 EXEMPLO: Manter Session Válida

```
Problema: Sessão expira a cada 5 requisições
Solução: Macro que faz login automaticamente

PASSO 1: Project options → Macros
PASSO 2: New macro
PASSO 3: Record:
  1. Faça login (username=test, password=test)
  2. Vai fazer requisição POST /login
  3. Recebe cookie válido
PASSO 4: Save macro como "re-login"
PASSO 5: No Repeater → Configure:
  Antes de cada requisição → Executar macro "re-login"
```

---

## 📦 EXTENSÕES RECOMENDADAS

### ESSENCIAIS (Gratuitas)

```
1. Active Scan++ (melhor scanner)
   - Detecta mais vulnerabilidades
   - Payloads mais efetivos

2. Autorize (teste authorization)
   - Testa IDOR/bypass automaticamente
   - Muito útil!

3. Logger++ (registro detalhado)
   - Log de todas requisições
   - Busca rápida

4. JSON Web Token (JWT)
   - Decodifica JWT
   - Tenta quebrar secrets
   - Analisa vulnerabilidades

5. SQL Map (SQL injection)
   - Integração com sqlmap
   - Detecção automática
```

### COMO INSTALAR

```
1. Burp → Extender → BApp Store
2. Procure por extensão (ex: "Active Scan++")
3. Clique "Install"
4. Aguarde instalar
5. Ativa automaticamente
```

---

## ⚙️ BOAS PRÁTICAS

### SEGURANÇA

```
1. ❌ NUNCA ative Intercept em produção
   └─ Pode modificar dados acidentalmente

2. ✅ Use Intercept só em testes autorizados
   └─ Verifique escopo do programa

3. ✅ Faça backup de projeto importante
   └─ Project → Save

4. ✅ Use VPN em rede pública
   └─ Proxy intercepta TUDO
```

### PERFORMANCE

```
1. Desabilite Passive scanning em redes lentas
   └─ Deixa tudo mais rápido

2. Use Filter em Proxy History
   └─ Histórico gigante = Burp lento

3. Limpe Proxy History frequentemente
   └─ Project → Clear history

4. Use Temporary project
   └─ Não salva lixo em disco
```

### WORKFLOW EFICIENTE

```
1. Comece com Proxy (exploração manual)
2. Confira achados com Repeater
3. Automação com Intruder/Scanner
4. Documente tudo
5. Gere relatório
```

---

## 🎯 CHECKLIST DE SETUP

```
☐ Burp Suite instalado
☐ Listener em 8080
☐ Navegador configurado com proxy
☐ Certificado SSL importado
☐ Passive scanning ativado
☐ Extensões importantes instaladas
☐ Projeto salvo
☐ Testado em site de teste (DVWA)
```

---

**Próximo:** Volte para [VULNERABILIDADES-OWASP-TOP-10](./02-OWASP-TOP-10-ULTRA-DETALHADO.md)

