# Integração Wazuh – GLPI (Alertas de Segurança em Tickets)

## 1. Objetivo
Este guia descreve **duas abordagens diferentes** para integrar o **Wazuh** com o **GLPI**, ao contrário do Zabbix, que tem um webhook nativo único. Não existe uma integração "oficial" única entre Wazuh e GLPI, pelo que aqui documentam-se as duas alternativas viáveis:

- **Parte A — Plugin GLPI (`wazuh`)**: instala-se um plugin no GLPI que se liga ao Wazuh e ao Wazuh Indexer, sincroniza agentes/ativos e permite consultar e converter alertas em tickets pela interface. Não requer scripting.
- **Parte B — Script via Wazuh Integrator**: usa o mecanismo genérico de integrações personalizadas do Wazuh (`<integration>` no `ossec.conf`) para correr um script que chama diretamente a API REST do GLPI e cria tickets automaticamente sempre que surge um alerta que cumpra os critérios definidos.

> [!NOTE]
> As duas abordagens não são mutuamente exclusivas, mas servem propósitos diferentes: a Parte A é mais rápida de implementar e dá visibilidade dos alertas por ativo, com criação de ticket manual; a Parte B é mais trabalhosa mas permite criação **automática** de tickets a partir de regras de severidade, à semelhança do que foi feito com o Zabbix.

---

# Parte A — Plugin GLPI para Wazuh

## A.1 O que faz
O plugin [`wazuh`](https://github.com/initiativa/wazuh) (initiativa srl, licença GPL-3.0) permite:
- Ligar a um ou mais servidores Wazuh e respetivos Wazuh Indexers
- Sincronizar os agentes Wazuh com os ativos (dispositivos) do GLPI
- Recolher periodicamente alertas e vulnerabilidades detetados pelo Wazuh
- Consultar esses eventos diretamente no separador do ativo correspondente em GLPI
- Criar tickets em GLPI manualmente a partir dos alertas selecionados

## A.2 Pré-requisitos
- Servidor GLPI funcional, com acesso de administrador
- Servidor(es) Wazuh funcional(is), incluindo o **Wazuh Indexer** acessível a partir do servidor GLPI
- Credenciais de acesso à API do Wazuh (utilizador/password) com permissões de leitura sobre alertas e vulnerabilidades
- Acesso root/sudo ao servidor GLPI, para colocar o plugin na pasta `plugins/`

> [!WARNING]
> O plugin encontra-se em desenvolvimento ativo (versão mais recente `0.1.5`) e não está disponível no GLPI Marketplace — a instalação é feita manualmente a partir dos releases no GitHub.

## A.3 Instalação do plugin

### A.3.1 Obter o plugin
- Aceder à página de releases: https://github.com/initiativa/wazuh/releases
- Descarregar o ficheiro `.zip` da versão mais recente (ex: `0.1.5`)

### A.3.2 Instalar no GLPI
No servidor GLPI, extrair o conteúdo do `.zip` para a pasta `plugins/` da instalação, garantindo que a pasta final se chama `wazuh`:

```bash
cd /var/www/html/glpi/plugins/
unzip wazuh-0.1.5.zip -d wazuh
chown -R www-data:www-data wazuh
```

> [!NOTE]
> Ajusta o caminho `/var/www/html/glpi/` e o utilizador `www-data` conforme a tua instalação (ver [Guia de Instalação do GLPI](../glpi/installation.md)).

### A.3.3 Ativar o plugin
- Aceder à interface web do GLPI como administrador
- Ir a **Configurar > Plugins**
- Localizar o plugin **Wazuh** na lista e clicar em **Instalar**, seguido de **Ativar**

## A.4 Configuração da ligação ao Wazuh
- Aceder a **Configurar > Plugins > Wazuh**
- Adicionar uma nova ligação, preenchendo:
  - **Endereço e porta do servidor Wazuh** (API do manager, por defeito porta `55000`)
  - **Endereço e porta do Wazuh Indexer** (por defeito porta `9200`)
  - **Utilizador** e **password** de acesso à API
  - Marcar a ligação como **Ativa**

Podem ser configuradas várias ligações, caso existam múltiplos servidores Wazuh a monitorizar.

## A.5 Sincronização de agentes
- Aceder a **Administração > Wazuh Agent's**
- Clicar em **Sync Agents** para importar a lista de agentes Wazuh registados no(s) servidor(es) configurado(s)
- Associar cada agente ao ativo GLPI correspondente:
  - Manualmente, através do campo **Devices** em cada agente, ou
  - Automaticamente, através do botão **Link Agents**, que associa por correspondência entre o nome do agente Wazuh e o nome do ativo em GLPI

> [!NOTE]
> A associação automática depende de os nomes dos agentes Wazuh corresponderem aos nomes definidos nos ativos GLPI (ex: hostname). Em caso de nomes divergentes, a associação deve ser feita manualmente.

## A.6 Recolha de alertas e vulnerabilidades
O plugin recolhe periodicamente os dados do Wazuh através de duas ações automáticas:
- **FetchAlerts** — recolhe os alertas gerados pelo Wazuh
- **FetchVulnerabilities** — recolhe as vulnerabilidades detetadas pelo módulo de deteção de vulnerabilidades do Wazuh

Por defeito, ambas correm de **hora a hora**. A frequência pode ser ajustada em **Configurar > Ações Automáticas** (Setup > Automatic Actions), localizando `FetchAlerts` e `FetchVulnerabilities`.

## A.7 Consulta de eventos e criação de tickets

### A.7.1 Consultar eventos de um ativo
- Aceder a **Ativos > Computadores** (ou **Equipamento de Rede**, conforme o tipo de ativo)
- Abrir o ativo associado ao agente Wazuh
- Consultar o(s) separador(es) **Wazuh**, onde são listados os alertas e vulnerabilidades recolhidos para aquele ativo

### A.7.2 Criar ticket a partir de alertas
- No separador de eventos Wazuh do ativo, selecionar o(s) alerta(s) pretendido(s)
- Em **Ações**, escolher **Criar ticket** (Create ticket)
- Confirmar/editar os detalhes do ticket gerado (título, descrição, urgência) antes de guardar

> [!NOTE]
> Aqui a criação de ticket **não é automática** — é uma ação manual sobre os alertas já recolhidos. Para automação total, ver a Parte B deste guia.

## A.8 Validação
- Confirmar que a ligação ao servidor Wazuh aparece como **Ativa** em **Configurar > Plugins > Wazuh**
- Confirmar, após a primeira execução de `FetchAlerts`, que aparecem alertas no separador Wazuh de um ativo associado a um agente
- Criar um ticket de teste a partir de um alerta e confirmar que este surge em **Assistência > Tickets** com os dados corretos

---

# Parte B — Script via Wazuh Integrator (criação automática de tickets)

## B.1 O que faz
O Wazuh disponibiliza um mecanismo genérico de integrações personalizadas, o **Integrator**, configurado através de um bloco `<integration>` no `ossec.conf` do manager. Sempre que um alerta cumpre os critérios definidos (nível mínimo, regra, grupo, etc.), o Wazuh invoca um script externo, passando-lhe o alerta em JSON. Esse script pode chamar diretamente a API REST do GLPI e criar o ticket — sem depender de nenhum plugin em GLPI.

Esta abordagem espelha a lógica usada na integração com o Zabbix (webhook automático), mas aqui o "webhook" é implementado através de um script à nossa responsabilidade.

> [!NOTE]
> Existe um projeto de referência da comunidade, [wazuh_to_GLPI](https://github.com/navein-kumar/wazuh_to_GLPI), com um script Python já feito para este cenário — pode ser usado como ponto de partida em vez do script de exemplo abaixo.

## B.2 Pré-requisitos
- Servidor Wazuh (manager) com acesso root/sudo
- Servidor GLPI com API REST ativa (ver secção B.3)
- Python 3 e a biblioteca `requests` disponíveis no servidor Wazuh

## B.3 Configuração no GLPI (API REST)
- Aceder a **Configurar > Geral > API**
  - Ativar **Enable Rest API**
  - Ativar **Enable login with external token**
- Em **Configurar > Geral > API**, clicar em **Add API client**, ativar e gerar o **App-Token**
- Criar um perfil em **Administração > Perfis** com permissões na secção **Assistência**: Tickets → **Create**, **Update**, **See all tickets**
- Criar um utilizador de serviço em **Administração > Utilizadores**, associar-lhe o perfil criado, e gerar o respetivo **User Token** (token de API pessoal)

> [!NOTE]
> Se já tiveres feito a integração com o Zabbix (ver guia anterior), podes reutilizar o mesmo utilizador de serviço e App-Token, desde que as permissões cubram ambos os casos — ou criar um utilizador dedicado ao Wazuh, para manter a rastreabilidade separada.

## B.4 Script de integração (Wazuh → GLPI)

### B.4.1 Localização e permissões
Os scripts de integrações personalizadas devem ser colocados em `/var/ossec/integrations/`, com o nome a começar obrigatoriamente por `custom-`:

```bash
sudo nano /var/ossec/integrations/custom-glpi
sudo chmod 750 /var/ossec/integrations/custom-glpi
sudo chown root:wazuh /var/ossec/integrations/custom-glpi
```

### B.4.2 Exemplo de script (Python)
O Wazuh invoca o script com três argumentos: caminho para o ficheiro do alerta, `api_key` (definido no `ossec.conf`) e `hook_url` (URL da API do GLPI). O exemplo abaixo lê o alerta, mapeia o nível de severidade para uma urgência GLPI, e cria o ticket via API REST v1:

```python
#!/usr/bin/env python3
import sys
import json
import requests

# Argumentos recebidos do Wazuh Integrator
alert_file = sys.argv[1]
user_token = sys.argv[2]          # vindo de <api_key> no ossec.conf
glpi_api_url = sys.argv[3]        # vindo de <hook_url> no ossec.conf, ex: https://glpi.exemplo.pt/apirest.php

# Definir aqui o App-Token gerado no passo B.3 (ou ler de variável de ambiente)
APP_TOKEN = "SUBSTITUIR_PELO_APP_TOKEN"

# Mapeamento de nível de alerta Wazuh (0-16) para urgência GLPI
def map_urgency(level):
    if level >= 13:
        return 5  # Muito alta
    elif level >= 10:
        return 4  # Alta
    elif level >= 7:
        return 3  # Média
    elif level >= 4:
        return 2  # Baixa
    return 1      # Muito baixa

def main():
    with open(alert_file) as f:
        alert = json.load(f)

    rule = alert.get("rule", {})
    level = rule.get("level", 0)
    description = rule.get("description", "Alerta Wazuh")
    agent = alert.get("agent", {}).get("name", "desconhecido")
    timestamp = alert.get("timestamp", "")

    headers_init = {
        "Content-Type": "application/json",
        "Authorization": f"user_token {user_token}",
        "App-Token": APP_TOKEN,
    }
    session = requests.get(f"{glpi_api_url}/initSession", headers=headers_init)
    session.raise_for_status()
    session_token = session.json()["session_token"]

    headers_ticket = {
        "Content-Type": "application/json",
        "Session-Token": session_token,
        "App-Token": APP_TOKEN,
    }
    payload = {
        "input": {
            "name": f"[Wazuh] {description} ({agent})",
            "content": (
                f"Alerta gerado pelo Wazuh.\n\n"
                f"Agente: {agent}\n"
                f"Nível: {level}\n"
                f"Descrição: {description}\n"
                f"Data: {timestamp}"
            ),
            "urgency": map_urgency(level),
        }
    }
    resp = requests.post(f"{glpi_api_url}/Ticket", headers=headers_ticket, json=payload)
    resp.raise_for_status()

    requests.get(f"{glpi_api_url}/killSession", headers=headers_ticket)

if __name__ == "__main__":
    main()
```

> [!WARNING]
> Este script é um ponto de partida e não inclui tratamento de erros, *logging*, nem proteção contra tickets duplicados para o mesmo alerta. Antes de usar em produção, adicionar tratamento de exceções e considerar deduplicação (ex: por `rule.id` + `agent.id` + janela temporal).

### B.4.3 Configurar o bloco de integração no Wazuh
Editar `/var/ossec/etc/ossec.conf` no manager e adicionar:

```xml
<ossec_config>
  <integration>
    <name>custom-glpi</name>
    <hook_url>https://glpi.exemplo.pt/apirest.php</hook_url>
    <api_key>TOKEN_DO_UTILIZADOR_GLPI</api_key>
    <level>7</level>
    <alert_format>json</alert_format>
  </integration>
</ossec_config>
```

- `<name>` deve corresponder exatamente ao nome do ficheiro do script (`custom-glpi`)
- `<level>` define o nível mínimo de severidade do alerta a partir do qual o ticket é criado (ajustar conforme a criticidade pretendida; níveis mais usados como referência: ≥7 relevante, ≥12 alta importância)
- `<alert_format>` tem de ser `json`, para que o script receba o alerta em formato estruturado

Podem ainda ser usados os campos opcionais `<rule_id>`, `<group>` e `<event_location>` para restringir ainda mais quais os alertas que despoletam a criação de ticket.

### B.4.4 Reiniciar o Wazuh Manager
```bash
sudo systemctl restart wazuh-manager
```

## B.5 Validação
- Provocar um alerta de teste com nível igual ou superior ao definido em `<level>` (ex: com o `wazuh-logtest` ou uma regra de teste)
- Verificar em `/var/ossec/logs/integrations.log` se o script foi executado sem erros
- Confirmar em GLPI, em **Assistência > Tickets**, que o ticket foi criado com a urgência correspondente

---

## Notas Finais
- A Parte A (plugin) é a opção recomendada para começar, por não exigir scripting e por dar visibilidade centralizada dos ativos e respetivos eventos de segurança
- A Parte B (script) é preferível quando se pretende criação **automática** de tickets a partir de regras de severidade, à semelhança do comportamento obtido com o Zabbix
- As credenciais (App-Token, User Token, password da API Wazuh) devem ser guardadas em local seguro e nunca commitadas em repositórios públicos

## Referências
- https://github.com/initiativa/wazuh
- https://github.com/initiativa/wazuh/releases
- https://github.com/navein-kumar/wazuh_to_GLPI
- https://documentation.wazuh.com/current/user-manual/manager/integration-with-external-apis.html
- https://wazuh.com/blog/how-to-integrate-external-software-using-integrator/
