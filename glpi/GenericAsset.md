# GLPI 11 – Guia para Criação de Assets Genéricos

---

## Passo 1 – Criar a Definição do Asset “Website”

**Nota:** Para criar um *generic asset*, a versão mínima necessária do GLPI é a **11**.

> **Nota para imagem:** Screenshot do caminho  
> `Setup → Asset Definitions → + Add`

1. Aceder a **Setup → Asset Definitions → + Add**
2. Preencher os seguintes campos:
   - **Label:** Website  
   - **System name:** website  
   - **Active:** Yes  
   - *(Opcionalmente, seleccionar um ícone)*

> **Nota para imagem:** Screenshot do formulário de criação da Asset Definition preenchido

3. Clicar em **Add**

---

## Passo 2 – Configurar os Campos (*Fields*)

Após criar a definição, abrir a mesma e aceder ao separador **Fields**.  
Manter apenas os campos exigidos pela **CNCS**.

> **Nota para imagem:** Screenshot do separador **Fields** da Asset Definition

| # | Field Label                         | Type              | Notes |
|---|------------------------------------|-------------------|-------|
| 1 | Nome                               | Text              | Nome representativo |
| 2 | Serviço Suportado                  | Dropdown / Text   | e.g. “Website Institucional”, “Portal Académico”, “VPN Portal” |
| 3 | Nome do Software / Equipamento     | Text              | e.g. “WordPress”, “Moodle” |
| 4 | Modelo / Versão                    | Text              | e.g. “6.4.1” |
| 5 | Endereço IP (se aplicável)         | String            | Public IP |
| 6 | FQDN (se aplicável)                | URL               | e.g. “https://www.estg.ipvc.pt” |
| 7 | Fabricante                         | Text / Dropdown   | e.g. “Open Source”, “Empresa X” |

Remover quaisquer outros campos desnecessários, tais como:
- **Status**
- **Technician in charge**
- **Inventory**

> **Nota para imagem:** Screenshot evidenciando campos removidos/desactivados

---

## Passo 3 – Organizar a Ordem dos Campos

Organizar os campos pela seguinte ordem, de acordo com o formato **CNCS QNR**:

1. Serviço Suportado  
2. Nome do Software / Equipamento  
3. Modelo / Versão  
4. Endereço IP (se aplicável)  
5. FQDN (se aplicável)  
6. Fabricante  

> **Nota para imagem:** Screenshot da ordenação dos campos (drag & drop)

No final, clicar em **Save**.

---

## Passo 4 – Adicionar Assets do Tipo Website

> **Nota para imagem:** Screenshot do menu  
> `Assets → Website → + Add`

1. Aceder a **Assets → Website → + Add**

**Nota:**  
Caso o utilizador não consiga visualizar o Asset criado, poderá ser necessário **terminar sessão e voltar a autenticar-se**.

2. Preencher os campos seguindo a mesma estrutura. Exemplo:

| Serviço Suportado        | Nome do Software | Modelo / Versão | Endereço IP        | FQDN                  | Fabricante |
|--------------------------|------------------|------------------|--------------------|-----------------------|------------|
| Website Institucional    | WordPress        | 6.4.1            | 193.136.xxx.xxx    | www.estg.ipvc.pt      | Open Source |

> **Nota para imagem:** Screenshot de um Asset Website preenchido

Cada *Website* exposto à Internet deve ser registado como **um Asset independente**.

---

## Passo 5 – Exportar o Ficheiro de Assets para a CNCS

> **Nota para imagem:** Screenshot da opção **Export → CSV**

1. Aceder a **Assets → Website**
2. Clicar em **Export → CSV**
3. Verificar que o ficheiro CSV contém **exactamente as seis colunas obrigatórias**:

