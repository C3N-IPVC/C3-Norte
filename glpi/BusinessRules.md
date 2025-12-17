# Guia de Configuração de Business Rules no GLPI  
## Atribuição Automática de Tickets

---

## Enquadramento e Objetivo

As **Business Rules** no GLPI são um dos mecanismos mais importantes para garantir a correta classificação e encaminhamento automático dos tickets logo no momento da sua criação.

Estas regras permitem reduzir intervenção manual, normalizar processos e assegurar que os tickets chegam rapidamente às equipas corretas.

---

## Conceito de Business Rules no GLPI

As Business Rules funcionam segundo uma lógica condicional do tipo:

**SE (critérios forem cumpridos) → ENTÃO (executar ações)**

Estas regras são avaliadas automaticamente pelo GLPI sempre que um ticket é criado ou recebido, permitindo aplicar decisões normalizadas sem dependência do utilizador final.

Cada Business Rule é composta por:

- Nome e descrição  
- Um ou mais critérios (condições)  
- Uma ou mais ações  
- Definição da permissão de uso por Sub-Entidades (opcional)

---

## Pré-requisitos Técnicos

Antes de iniciar a configuração das Business Rules, é fundamental garantir que a estrutura base do GLPI se encontra corretamente definida.

Deve existir:

- Hierarquia de Entidades e Sub-Entidades configurada  
- Categorias de tickets organizadas por áreas técnicas  
- Grupos técnicos associados às equipas reais  
- Métodos de criação de tickets ativos (Interface Web, Email ou API)

---

## Aceder ao Módulo de Business Rules

Para iniciar a criação ou gestão de Business Rules, seguir o caminho:

**Setup → Rules → Tickets → Tickets for business Rules**

Neste menu é possível:

- Criar regras  
- Editar regras existentes  
- Ativar ou desativar regras  
- Definir a ordem de avaliação das regras

---

## Criação de uma Nova Business Rule

No ecrã de regras, clicar em **“Add”** para criar uma nova Business Rule.

### Campos a preencher

- **Nome da regra**  
  Deve ser descritivo e objetivo

- **Descrição**  
  Recomendada para documentar claramente o propósito da regra

*(Inserir imagem do ecrã de criação da regra)*

---

## Definição dos Critérios (IF)

Os critérios determinam em que condições a regra será aplicada.

Podem ser utilizados vários critérios em simultâneo, permitindo um controlo mais preciso sobre a ativação da regra.

### Critérios frequentemente utilizados

- **Subject contém determinada palavra-chave** *(Importante)*  

  > A utilização de expressões regulares (regex) no campo *Subject* permite identificar automaticamente variações do mesmo tipo de alerta, mesmo quando o texto do e-mail sofre pequenas alterações.  
  >  
  > Esta abordagem aumenta a eficácia da deteção, reduz falsos negativos e torna as regras mais robustas, sendo especialmente útil em alertas técnicos e de segurança gerados por diferentes sistemas.  
  >  
  > Para criar e testar expressões regulares, recomenda-se a utilização da plataforma **regex101**, selecionando o motor **PCRE (PHP)**.

- **Corpo do ticket contém termos técnicos**

- **Email do remetente**

- **Entidade associada**

Recomenda-se a utilização de critérios específicos para evitar classificações incorretas ou atribuições erradas.

*(Inserir imagem da definição de critérios)*

---

## Definição das Ações (THEN)

As ações são executadas quando **todos os critérios da regra são cumpridos**.

### Ações mais utilizadas em Business Rules de tickets

- **Definir categoria do ticket** *(Importante)*  
- **Atribuir grupo técnico**  
- **Definir prioridade**  
- **Definir impacto ou urgência**

Uma regra pode conter múltiplas ações, que são aplicadas em simultâneo.

*(Inserir imagem da definição das ações)*

---

## Ordem e Prioridade das Regras

As Business Rules são avaliadas sequencialmente, de cima para baixo.  
A ordem das regras influencia diretamente o resultado final.

### Boas práticas

- Colocar regras mais específicas no topo  
- Deixar regras genéricas no final  
- Evitar regras redundantes ou sobrepostas  
- Documentar claramente a finalidade de cada regra

*(Inserir imagem da lista de regras e respetiva ordem)*

---

## Validação e Testes

Após a criação de uma Business Rule, é essencial validar o seu funcionamento antes de a colocar em produção.

A validação pode ser realizada de duas formas distintas.

---

### Teste através do *Test rules engine*

O GLPI disponibiliza uma ferramenta interna de simulação que permite testar uma regra sem criar efetivamente um ticket.

Neste ecrã, o utilizador pode:

- Preencher manualmente os critérios relevantes  
  (ex.: assunto, entidade, origem, impacto)

- Simular a entrada de um ticket por:
  - Email  
  - Interface Web  
  - API

- Verificar se a Business Rule é acionada  
- Confirmar quais as ações aplicadas

*(Inserir imagem do Test rules engine)*

---

## Considerações Finais

A correta utilização das Business Rules permite:

- Automatizar a classificação de tickets  
- Reduzir erros humanos  
- Melhorar tempos de resposta  
- Aumentar a eficiência operacional das equipas técnicas

Uma abordagem bem estruturada e devidamente testada garante um sistema de ticketing mais fiável, escalável e alinhado com as boas práticas ITIL.
