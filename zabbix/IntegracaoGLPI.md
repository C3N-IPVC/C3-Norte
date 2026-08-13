# Integração Zabbix – GLPI (Alertas de Problemas em Tickets)

## 1. Objetivo
Este guia descreve o processo de integração entre o **Zabbix** e o **GLPI**, permitindo que problemas (alertas) detetados pelo Zabbix num servidor sejam automaticamente criados como tickets no painel de **Assistência** do GLPI.

Esta integração é feita através de um *media type* nativo do Zabbix ("GLPI"), disponibilizado oficialmente pela Zabbix, eliminando a necessidade de scripts externos.

Quando um problema é despoletado no Zabbix:
- É criado automaticamente um ticket em GLPI, com a urgência mapeada de acordo com a severidade do alerta
- Se o problema for atualizado, é adicionado um *followup* ao ticket correspondente
- Quando o problema é resolvido em Zabbix, o ticket em GLPI é marcado como **Resolvido**

**Mapeamento de severidade (Zabbix → Urgência GLPI):**

| Severidade Zabbix | Urgência GLPI |
|---|---|
| Not classified / Information | Muito baixa |
| Warning | Baixa |
| Average | Média |
| High | Alta |
| Disaster | Muito alta |

---

## 2. Pré-requisitos
- Servidor Zabbix funcional, versão 6.0 ou superior (testado com 6.0, 6.2, 6.4, 7.0, 7.2, 7.4)
- Servidor GLPI funcional (testado com 9.5.7, 10.0.16–10.0.24 e 11.0.5–11.0.6)
- Acesso de administrador em ambas as plataformas
- Conectividade de rede entre o servidor Zabbix e o servidor GLPI (HTTP/HTTPS)

> [!NOTE]
> Existem duas formas de autenticação suportadas: **API v2 (OAuth2)**, recomendada para GLPI 11+, e **API v1 (Legacy)**, disponível para GLPI 10.x. A API v1 está marcada como depreciada pela Zabbix, pelo que a API v2 deve ser preferida sempre que a versão do GLPI o permitir.

---

## 3. Configuração no GLPI

### 3.1 Ativar a API REST
- Aceder a **Configurar > Geral > API**
- Ativar a opção **Enable Rest API**
- Se for utilizada a API v1 (Legacy), ativar também **Enable login with external token**

### 3.2 (API v2) Criar cliente OAuth
- Aceder a **Configurar > OAuth clients**
- Adicionar um novo cliente com:
  - **Scope**: `api`
  - **Grant type**: `Password`
- Guardar o **Client ID** e o **Client Secret** gerados — serão necessários na configuração do Zabbix

### 3.3 (API v1) Criar cliente de API
- Aceder a **Configurar > Geral > API**
- Clicar em **Add API client**
- Ativar o cliente e, opcionalmente, restringir o acesso por IP
- Gerar e guardar o **App Token**

### 3.4 Criar perfil de serviço
- Aceder a **Administração > Perfis**
- Criar um novo perfil com as seguintes permissões na secção **Assistência**:
  - Tickets: **Update**, **Create**, **See all tickets**
  - Followups/Tasks: **Add followup**

### 3.5 Criar utilizador de serviço
- Aceder a **Administração > Utilizadores**
- Criar um novo utilizador e associar-lhe o perfil criado no passo anterior
- Se for utilizada a API v1, gerar (ou regenerar) o **token de API** deste utilizador e guardá-lo

---

## 4. Configuração no Zabbix

### 4.1 Definir a macro global do URL
- Aceder a **Administration > Macros**
- Definir a macro `{$ZABBIX.URL}` com o URL completo do servidor Zabbix, incluindo o protocolo

Exemplos válidos:
```
http://zabbix.exemplo.pt
https://zabbix.exemplo.pt/zabbix
http://127.0.0.1:8080
```

> [!WARNING]
> O valor da macro tem de incluir sempre o protocolo (`http://` ou `https://`). Um valor como `zabbix.exemplo.pt` (sem protocolo) é inválido e a integração não funcionará corretamente.

### 4.2 Importar o media type GLPI
- Descarregar o ficheiro `media_glpi.yaml` a partir do repositório oficial do Zabbix:
  `git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates/media/glpi`
- Aceder a **Alerts > Media types**
- Clicar em **Import** e selecionar o ficheiro `media_glpi.yaml`

### 4.3 Configurar os parâmetros do media type
Após a importação, abrir o media type **GLPI** e configurar os parâmetros conforme o tipo de API escolhida.

**API v2 (OAuth2):**

| Parâmetro | Descrição |
|---|---|
| `glpi_url` | URL do frontend do GLPI (sem sufixo de path) |
| `glpi_legacy_api` | `false` |
| `glpi_client_id` | Client ID gerado no passo 3.2 |
| `glpi_client_secret` | Client Secret gerado no passo 3.2 |
| `glpi_username` | Login do utilizador de serviço |
| `glpi_password` | Password do utilizador de serviço |

**API v1 (Legacy):**

| Parâmetro | Descrição |
|---|---|
| `glpi_url` | URL do frontend do GLPI |
| `glpi_legacy_api` | `true` |
| `glpi_app_token` | App Token gerado no passo 3.3 (opcional) |
| `glpi_user_token` | Token de API do utilizador (obrigatório) |

Parâmetros opcionais de mapeamento de urgência (podem ser ajustados conforme a criticidade pretendida):
- `severity_not_classified`, `severity_information`, `severity_warning`, `severity_average`, `severity_high`, `severity_disaster`
- `glpi_urgency_internal`, `glpi_urgency_discovery`, `glpi_urgency_autoregistration`

> [!NOTE]
> Se o Zabbix necessitar de passar por um proxy para aceder ao GLPI, adicionar o parâmetro `http_proxy` com o URL do proxy.

Depois de configurados os parâmetros, marcar a caixa **Enabled** e guardar o media type.

> [!WARNING]
> O parâmetro **Retry count** deste media type vem por defeito a `1`. Não é recomendado aumentar este valor, uma vez que pode originar **tickets duplicados** em GLPI em caso de erros de transação.

### 4.4 Criar utilizador Zabbix com o media type GLPI
- Aceder a **Users > Users**
- Criar (ou editar) um utilizador
- No separador **Media**, adicionar uma entrada com o media type **GLPI**
  - No campo **Send to**, introduzir qualquer valor (campo obrigatório na interface, mas não é utilizado pelo webhook)
- Confirmar que este utilizador tem permissões de leitura sobre os hosts que se pretende monitorizar

### 4.5 Criar a ação de notificação (Trigger Action)
Para que os problemas detetados sejam efetivamente enviados para o GLPI, é necessário criar uma ação em Zabbix:

- Aceder a **Alerts > Actions > Trigger actions**
- Criar uma nova ação (ex: *"Notificar GLPI em problemas"*)
- Definir as condições pretendidas (ex: severidade mínima do problema)
- Na secção **Operations**, adicionar uma operação do tipo **Send message**, configurando:
  - **Send to users**: o utilizador criado no passo 4.4
  - **Send only to**: media type **GLPI**
- Ativar a ação

---

## 5. Validação da Integração
- Provocar um problema de teste num host monitorizado (ex: parar temporariamente um serviço monitorizado ou simular a condição de um trigger)
- Verificar em GLPI, em **Assistência > Tickets**, se foi criado um novo ticket com:
  - Título/descrição correspondente ao problema detetado
  - Urgência mapeada de acordo com a severidade do alerta
- Resolver o problema em Zabbix e confirmar que o ticket correspondente em GLPI é atualizado e marcado como **Resolvido**

---

## 6. Notas Importantes
- A **API v1 (Legacy)** está marcada como depreciada pela Zabbix; para novas integrações deve ser utilizada a **API v2 (OAuth2)**, sempre que a versão do GLPI o permita (GLPI 11+)
- Não aumentar o **Retry count** do media type, sob risco de criação de tickets duplicados
- Manter as credenciais (Client Secret, App Token, User Token) em local seguro, idealmente num gestor de segredos

---

## 7. Referências
- https://www.zabbix.com/integrations/glpi
- https://git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates/media/glpi
