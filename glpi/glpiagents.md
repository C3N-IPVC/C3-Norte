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

```bash
wget https://github.com/glpi-project/glpi-agent/releases/download/1.7/glpi-agent-1.7-linux-installer.pl
