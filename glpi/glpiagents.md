# Guia de Instalação e Configuração do GLPI Agent

## 1. Introdução
Este guia descreve o processo de instalação e configuração do GLPI Agent em sistemas Linux (Ubuntu) e Windows, permitindo a comunicação de inventário com o servidor GLPI.

---

# Linux (Ubuntu)

## 2. Requisitos

- Servidor GLPI em funcionamento (testado com GLPI 10.x)
- Inventário ativo:
  - Administration > Inventory > Configuration > Enable Inventory
- Máquina Ubuntu (VM ou física)
- Acesso sudo/root
- Sistema atualizado
- Conectividade de rede entre o agente e o servidor

---

## 3. Download do GLPI Agent

Faz o download da versão mais recente do GLPI Agent para Linux a partir das releases oficiais no GitHub:

>[!Warning]
> Confirma sempre se estás a utilizar a versão mais recente disponível no repositório oficial.
> https://github.com/glpi-project/glpi-agent/releases

```bash
wget https://github.com/glpi-project/glpi-agent/releases/download/1.7/glpi-agent-1.7-linux-installer.pl

```
## 4. Instalação do GLPI Agent

Tornar o instalador executável e executá-lo com permissões de superutilizador:

```bash
chmod +x glpi-agent-1.7-linux-installer.pl
sudo perl glpi-agent-1.7-linux-installer.pl --type=all
```

> [!NOTE]
> Se pretenderes instalar apenas o módulo de inventário, remove a opção `--type=all`.

Durante a instalação, poderão ser solicitadas as seguintes configurações:

- Diretório de instalação (recomendado manter o padrão)
- Arranque automático do serviço (recomendado: **Sim**)

## 5. Configuração do URL do servidor GLPI

Após a execução do comando de instalação, será solicitado que introduzas o URL do servidor GLPI (substituir pelo IP ou hostname do servidor):

```bash
http://<IP_ou_hostname>/front/inventory.php
```

> [!NOTE]
> Se o GLPI estiver instalado numa pasta personalizada (ex: `/var/www/html/glpi/public`), ajusta o caminho conforme necessário.

## 6. Verificação do serviço do GLPI Agent

Verifica se o GLPI Agent está a correr como serviço:

```bash
sudo systemctl status glpi-agent
```

- **Active (running)** → o agente está a funcionar corretamente  
- **Inactive/failed** → verificar logs com:

```bash
sudo journalctl -u glpi-agent -e
```
## 7. Forçar envio de inventário

Para testar, podes forçar manualmente o agente a enviar o inventário para o servidor GLPI:

```bash
sudo glpi-agent --debug --server http://<IP_ou_hostname>/front/inventory.php --force
```

---

## 8. Verificação no GLPI

Acede ao interface web do GLPI.

- Navega até **Assets > Computers**
  - Deverás ver a máquina Ubuntu listada

