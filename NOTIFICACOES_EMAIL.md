# Sistema de Notificações por E-mail

## Visão Geral

O sistema agora possui notificações automáticas por e-mail para alertar administradores sobre situações importantes relacionadas aos veículos.

## Casos de Notificação

### 1. Combustível Baixo (Checklist de Entrega)
**Quando:** O veículo é entregue com nível de combustível abaixo de 1/2 tanque (vazio ou 1/4).

**Conteúdo do e-mail:**
- Dados do veículo (placa, modelo, marca)
- Nível de combustível informado
- Data e hora da informação
- Nome do usuário que registrou

### 2. Não Conformidade no Checklist
**Quando:** Há alguma não conformidade detectada nos componentes ou riscos/danos no checklist de entrega.

**Conteúdo do e-mail:**
- Dados do veículo
- Data/hora de chegada
- KM final
- Checklist completo com tabelas de componentes e riscos
- Destaque para itens em não conformidade
- Nome do usuário que registrou

### 3. Cadastro de Avaria
**Quando:** Uma nova avaria é registrada no sistema.

**Conteúdo do e-mail:**
- Dados do veículo (placa, modelo, marca)
- Descrição da avaria
- Data e hora da constatação
- Nome do usuário que registrou
- Foto da avaria (anexada e com link)

## Destinatários

As notificações são enviadas **apenas** para administradores que atendem aos seguintes critérios:

1. Possuem role de "admin" na tabela usuarios
2. Estão registrados na **mesma empresa** do usuário que gerou o alerta
3. Estão registrados na **mesma área** do usuário que gerou o alerta
4. Têm o campo **"notificacao" = true** na tabela usuarios

## Configuração de Notificações

### Para Administradores

Os administradores podem controlar se desejam receber notificações através do painel de gerenciamento de usuários:

1. Acesse: **Dashboard Admin → Usuários**
2. Na listagem de usuários, há uma coluna "Notificações" com um toggle (switch)
3. Toggle VERDE (ativado) = recebe notificações
4. Toggle CINZA (desativado) = não recebe notificações

### Configuração do Servidor

Para que o sistema envie e-mails, é necessário configurar as variáveis de ambiente no arquivo `.env`:

```env
# Configurações de E-mail
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=seu_email@gmail.com
EMAIL_PASSWORD=sua_senha_de_app
EMAIL_FROM=seu_email@gmail.com
```

#### Usando Gmail:

1. Acesse sua conta Google
2. Vá em "Segurança"
3. Ative a "Verificação em duas etapas"
4. Gere uma "Senha de app" específica para o sistema
5. Use essa senha no `EMAIL_PASSWORD`

#### Outros provedores de e-mail:

- **Outlook/Hotmail:** `EMAIL_HOST=smtp-mail.outlook.com` e `EMAIL_PORT=587`
- **Yahoo:** `EMAIL_HOST=smtp.mail.yahoo.com` e `EMAIL_PORT=587`
- **SendGrid, Mailgun, etc.:** Consulte a documentação do provedor

## Estrutura Técnica

### Funções Criadas

**`get_admin_recipients(client, empresa, area)`**
- Busca admins com notificação ativada da mesma empresa e área

**`send_email_notification(recipients, subject, body_html, attachments)`**
- Envia e-mail com suporte a HTML e anexos

**`notify_low_fuel(client, user, nivel_combustivel, data_hora, veiculo)`**
- Notifica sobre combustível baixo

**`notify_non_conformity(client, user, checklist_data, veiculo)`**
- Notifica sobre não conformidades

**`notify_avaria(client, user, avaria_data, veiculo, foto_url)`**
- Notifica sobre avarias cadastradas

### Pontos de Integração

1. **Finalização de relatório** (`finalizar_relatorio`):
   - Verifica combustível após salvar relatório de entrega
   - Verifica não conformidades em componentes e riscos

2. **Cadastro de avaria** (`registrar_avaria`):
   - Envia notificação imediatamente após cada avaria ser registrada
   - Baixa e anexa a foto (se houver)

### Nova Rota API

**`POST /admin/usuarios/<user_id>/notificacao`**
- Atualiza o campo `notificacao` do usuário
- Requer autenticação de admin
- Body: `{"notificacao": true/false}`

## Tabela de Usuários

O campo `notificacao` foi adicionado à tabela `usuarios`:

```sql
ALTER TABLE usuarios ADD COLUMN notificacao BOOLEAN DEFAULT false;
```

**Nota:** Execute este comando SQL no Supabase se a coluna ainda não existir.

## Dependências

As bibliotecas necessárias já fazem parte do Python padrão:
- `smtplib` - envio de e-mails
- `email.mime` - construção de mensagens
- `requests` - download de imagens para anexo

## Solução de Problemas

### E-mails não estão sendo enviados

1. Verifique se as variáveis de ambiente estão configuradas corretamente
2. Verifique o console do servidor para mensagens de erro
3. Confirme que o admin tem a notificação ativada
4. Verifique se empresa e área correspondem

### Gmail bloqueando envio

1. Use "Senha de app" (não a senha normal da conta)
2. Certifique-se que a verificação em 2 etapas está ativa
3. Verifique se "Acesso a apps menos seguros" não está bloqueando (não recomendado)

### Anexos de fotos não funcionam

1. Verifique se a URL da foto é pública no Supabase Storage
2. Confirme que há conectividade com a internet
3. Verifique logs do servidor para erros de download

## Logs

O sistema registra no console:
- `[INFO] E-mail enviado com sucesso para: ...`
- `[WARN] Configurações de e-mail não definidas...`
- `[WARN] Nenhum destinatário para envio de e-mail`
- `[ERROR] Erro ao enviar e-mail: ...`
