# Relatório de Análise Profunda - Metin2CMS v2.12

Este documento detalha a análise de segurança e funcionalidade da sua CMS, identificando pontos fortes, vulnerabilidades críticas e recomendações para uma operação segura.

## 1. Arquitetura e Tecnologias
- **Linguagem:** PHP (Requisito >= 5.6.0).
- **Bancos de Dados:**
    - **MySQL (PDO):** Conexão segura com as bases do jogo (`account`, `player`, `common`, `log`).
    - **SQLite:** Armazenamento local em `include/db/site.db` para dados exclusivos do site (notícias, logs de banimento, referências).
- **Segurança Nativa:**
    - Uso consistente de *prepared statements* (PDO), mitigando SQL Injection.
    - Proteção de diretórios sensíveis via `.htaccess` (`Deny from all` em `include/`).

## 2. Vulnerabilidades Identificadas

### 2.1. Ataque de Replay no PayPal (Crítico)
**Arquivo:** `paypal.php`
- **Problema:** O script não verifica se um `txn_id` (ID de transação) já foi processado anteriormente.
- **Risco:** Um jogador mal-intencionado pode reenviar o pacote de confirmação do PayPal (IPN) múltiplas vezes para receber Moedas do Dragão (MD) infinitamente com um único pagamento.
- **Correção:** Criar uma tabela no SQLite para registrar os `txn_id` processados e validar antes de entregar as moedas.

### 2.2. Ausência de Proteção CSRF (Alta)
**Local:** Todos os formulários (Login, Registro, Administração).
- **Problema:** Não existem tokens CSRF para validar a origem das requisições.
- **Risco:** Um administrador logado pode ser induzido a clicar em um link malicioso que executa ações no painel (deletar notícias, mudar configurações ou promover outros jogadores a admin) sem seu conhecimento.
- **Correção:** Implementar tokens de sessão únicos para cada formulário.

### 2.3. Vulnerabilidades de XSS (Média)
**Local:** Exibição de motivos de banimento e conteúdo de notícias.
- **Problema:** O sistema exibe dados do banco sem a devida sanitização com `htmlspecialchars()`.
- **Risco:** Injeção de scripts maliciosos que podem roubar cookies de sessão de outros usuários ou administradores.
- **Correção:** Envolver as saídas de texto em `htmlspecialchars($data, ENT_QUOTES, 'UTF-8')`.

### 2.4. Tokens de Segurança Fracos (Baixa)
**Arquivo:** `include/functions/basic.php`
- **Problema:** A função `generateSocialID` utiliza `rand()`, que não é criptograficamente seguro. Além disso, os tokens de recuperação de senha não expiram por tempo.
- **Risco:** Previsibilidade teórica de tokens em ataques de larga escala.
- **Correção:** Utilizar `random_bytes()` ou `openssl_random_pseudo_bytes()` e adicionar uma coluna de timestamp para expiração.

## 3. Recomendações de Configuração Segura

### 3.1. Ambiente PHP
- **Versão:** Utilize PHP 7.4 ou 8.x se possível (verifique compatibilidade de sintaxe, embora a CMS cite 5.6).
- **Configurações no `php.ini`:**
    - `display_errors = Off` (Para não expor caminhos de arquivos em caso de erro).
    - `expose_php = Off` (Esconde a versão do PHP no cabeçalho HTTP).
    - `session.cookie_httponly = On` (Dificulta roubo de sessão via XSS).
    - `allow_url_fopen = Off` (Se não for utilizar funções que dependem de URLs externas).

### 3.2. Banco de Dados MySQL
- **Permissões:** O usuário do banco configurado no `config.php` deve ter permissões apenas nas tabelas necessárias. Evite usar o usuário `root` do MySQL.
- **Acesso Remoto:** Certifique-se de que o MySQL aceite conexões apenas do IP da sua hospedagem web (se forem servidores diferentes).

### 3.3. Certificado SSL (HTTPS)
- É obrigatório o uso de HTTPS (SSL) para proteger os dados de login e transações de pagamento dos seus jogadores.

## 4. Guia de Manutenção
1. **Backups:** Sempre faça backup do arquivo `include/db/site.db`. Se ele for deletado, você perderá as notícias e o registro de quem foi banido via site.
2. **Logs:** Verifique regularmente a tabela `ban_log` no SQLite para monitorar a atividade dos seus Game Masters (GMs).

---
*Análise concluída com sucesso.*
