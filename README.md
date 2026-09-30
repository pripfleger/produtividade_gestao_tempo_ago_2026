# Personal OS — Sistema Operacional Pessoal com IA

> Um fluxo integrado de ferramentas para gerenciar **tempo, comunicação e produtividade**, combinando os métodos **GTD** e **Ivy Lee** com automações no **Make** e resumos diários gerados pelo **Gemini**.

---

## Sobre o projeto

O **Personal OS** (*Personal Operating System*) é um sistema de organização pessoal que **não possui uma tela própria**: ele é formado por um **fluxo integrado de ferramentas** que já fazem parte da rotina profissional — Google Sheets, Google Agenda, Trello, Make e Gemini.

O projeto integra:

- **Métodos de produtividade** (GTD + Ivy Lee)
- **Planejamento** com calendário e quadro de tarefas
- **Comunicação** por meio de resumos diários enviados por e-mail
- **IA** para analisar as atividades e apoiar a tomada de decisões
- **Acompanhamento de hábitos** e lembretes de bem-estar

---

## Contexto e problema

Uso no dia-a-dia de vários sistemas que não possuem integração entre si, e por isso utiliza-se uma planilha Google Sheets para controle pessoal do cronograma. 

**Principais dificuldades identificadas:**

- Saber **por onde começar** o dia de trabalho
- Fazer várias tarefas ao mesmo tempo, sem concluí-las por completo
- Tarefas esquecidas no meio do percurso
- **Procrastinação** gerada pela indecisão
- Dificuldade de **concentração**
- Informações **espalhadas** em planilhas e sistemas sem integração

**Como era antes:** sem método específico — as tarefas eram executadas conforme lembradas, apoiadas por uma agenda física e pelo Google Agenda (com notificações antecipadas).

**Preferências que guiaram o projeto:**

- Registrar tudo **no mesmo lugar**
- Visualizar o **mês completo** ("o todo"), o que dá sensação de controle
- **Evitar muitas ferramentas** desconectadas — daí a importância da integração

---

## Objetivos

- Aumentar a produtividade
- Reduzir retrabalho e esquecimentos
- Melhorar a comunicação profissional
- Diminuir a procrastinação por indecisão
- Construir uma rotina mais **equilibrada e sustentável**

---

## Métodos de produtividade

Os dois métodos foram usados em conjunto e adaptados para uma parte das atividades do dia-a-dia.

### GTD — *Getting Things Done*

Método de 5 etapas:

1. **Capturar:** anotar todas as tarefas, ideias, compromissos, projetos e lembretes
2. **Esclarecer:** analisar se cada item exige ação — se levar até **2 minutos**, fazer imediatamente
3. **Organizar:** separar em listas (itens com data/hora, próximas ações, projetos grandes, dependências de terceiros, ideias futuras)
4. **Revisar:** revisar periodicamente as listas, atualizando e adicionando itens
5. **Executar**

### Ivy Lee

- Definir, no fim do dia, **no máximo 6 tarefas** para o dia seguinte, em **ordem de importância**
- Só passar para a próxima tarefa quando a anterior estiver concluída
- O que não for concluído vai para a lista do dia seguinte

**Por que funciona:**

| Benefício | Explicação |
|---|---|
| Elimina a fadiga de decisão | O dia já começa com a primeira tarefa definida |
| Reduz a procrastinação | Não há indecisão sobre por onde começar |
| Gera foco profundo | Impede a dispersão em multitarefas de baixa prioridade |
| Garante realismo | O limite de 6 itens força escolhas essenciais |

---

## Visão geral da arquitetura

O sistema é composto por **dois fluxos de automação** no Make:

```
FLUXO 1 — Agendamento automático
Google Sheets ──(Apps Script)──▶ Webhook ──▶ Google Sheets (Get Range Values)
      ──▶ Iterator ──▶ Google Calendar (Create an Event) ──▶ Trello (Create a Card)

FLUXO 2 — Resumo diário com IA
Trello (Get a Board) ──▶ Text Aggregator ──▶ Google Gemini AI (Generate a response)
      ──▶ Gmail (Send an email)
```

---

## Ferramentas utilizadas

| Ferramenta | Papel no sistema |
|---|---|
| **Google Sheets** | Base de informações dos clientes/alunos e gatilho da automação |
| **Google Apps Script** | Envia a linha alterada da planilha para o Webhook do Make |
| **Make** | Plataforma de automação que integra as ferramentas |
| **Google Agenda** | Registro dos agendamentos com data e hora |
| **Trello** | Quadro GTD + Ivy Lee para organização das tarefas |
| **Google Gemini** | IA que analisa o quadro e gera o resumo de produtividade |
| **Gmail** | Envio do resumo diário por e-mail |

---

## Fluxo 1 — Agendamento automático

Garante que **nenhum agendamento seja esquecido**: ao finalizar o cadastro na planilha, o evento é criado na agenda e o card é criado no Trello.

### Como funciona

1. Uma alteração na planilha (ex.: cadastro de um novo cliente) marcada com o status **"OK"** na coluna *FINALIZADO* dispara o gatilho.
2. O **Apps Script** envia o número da linha alterada para o **Webhook** do Make.
3. O Make busca os dados da linha no **Google Sheets**.
4. O **Iterator** percorre os dados (com filtro para considerar **somente datas**).
5. Um evento é criado no **Google Agenda** para cada data.
6. Um card é criado na lista **Atendimentos** do **Trello** para cada data.

### Estrutura da planilha (dados fictícios)

| RA | NOME ALUNO | TELEFONE | DATA MATRÍCULA | CURSO | PROVA 1 | PROVA 2 | PROVA 3 | PROVA 4 | FINALIZADO |
|---|---|---|---|---|---|---|---|---|---|
| 1258 | Ana Carolina Silva | (49) 99123-4501 | 10/01/2026 | MÉDIO | 12/10/2026 | 09/11/2026 | | | OK |

### Script no Apps Script

Código adicionado ao projeto de Apps Script da planilha (a coluna monitorada é a **10**, *FINALIZADO*):

```javascript
function onEdit(e) {
  var columnTarget = 10; // coluna J - FINALIZADO
  var urlWebhook = "https://hook.us2.make.com/SEU_WEBHOOK_AQUI";

  var columnChanged = e.range.getColumn();
  var rowChanged = e.range.getRow();
  var valueTyped = e.range.getValue();

  if (columnChanged === columnTarget && valueTyped !== "") {
    var payload = { "row": rowChanged };
    var options = {
      "method": "post",
      "contentType": "application/json",
      "payload": JSON.stringify(payload)
    };
    UrlFetchApp.fetch(urlWebhook, options);
  }
}
```

### Gatilho (trigger)

É necessário criar um **acionador** no Apps Script:

| Campo | Valor |
|---|---|
| Função | `onEdit` |
| Origem do evento | Da planilha |
| Tipo de evento | Ao editar |

> ⚠️ Como o `onEdit` simples não consegue chamar serviços externos (como `UrlFetchApp`), o acionador **instalável** é obrigatório.

---

## Estrutura do quadro no Trello

Foi criado um quadro novo (**Personal GTD + IvyLee**) com as listas abaixo. Os itens podem ser movidos entre listas conforme o andamento, o que dá flexibilidade para reorganizar de acordo com a necessidade.

| Lista | Finalidade |
|---|---|
| **6 atividades – Ivy Lee** | Deixar preparadas, para o dia seguinte, as atividades prioritárias |
| **Novos Itens** | Caixa de entrada: novas atividades, eventos e ideias, para classificar depois |
| **Todos os dias** | Atividades recorrentes, independentes das demais |
| **Realizar de Imediato** | Tarefas rápidas (regra dos 2 minutos do GTD) |
| **Atendimentos** | Atendimentos agendados, com nº do RA, nome do cliente e data |
| **Projetos** | Ideias e atividades longas, de médio e longo prazo |
| **Financeiro** | Atividades relacionadas à gestão contábil |
| **Concluídos – mês/ano** | Tarefas finalizadas são movidas automaticamente para cá; **uma lista por mês** mantém o histórico mensal |

### Rotina diária sugerida

1. Durante o dia, registrar tudo em **Novos Itens**.
2. Classificar cada item nas listas do GTD.
3. Executar a lista **6 atividades – Ivy Lee** na ordem de prioridade.
4. Ao final do dia, definir as **6 atividades do dia seguinte**.
5. Receber o **resumo diário por e-mail**.

---

## Fluxo 2 — Resumo diário com IA

Um segundo fluxo no Make envia, todos os dias de semana às **18h30**, um resumo de produtividade gerado pela IA.

### Etapas

1. **Trello — Get a Board:** obtém as informações do quadro
2. **Tools — Text aggregator:** unifica os dados em um único texto
3. **Google Gemini AI — Generate a response:** analisa as atividades
4. **Gmail — Send an email:** envia o resumo por e-mail

### Configuração do modelo

- **Modelo:** Gemini Flash 3.8
- **Role:** `User`

**Prompt da mensagem** (o que a IA deve fazer):

```text
Atue como um especialista em gestão do tempo e produtividade pessoal. Analise todo o {{11.text}} que veio do meu Trello e gere um Resumo Diário de Produtividade escrito de forma clara e motivacional e em HTML, contendo o que foi concluído e movimentado durante o dia. Não esqueça de adicionar quebras de linha e espaçamentos para ficar mais legível. Destaque para os principais atendimentos e pendências resolvidas. Dê uma breve mensagem de encerramento para o dia.
```

**Instruções do sistema** (como a IA deve se comportar):

```text
Você é um assistente especialista em gestão de tempo e produtividade. Sua única função é analisar as tarefas registradas no Trello e gerar um relatório resumido de produtividade em texto simples. Seja analítico, motivacional e direto no tom de voz.
```

> `{{11.text}}` é a variável do módulo *Text aggregator* no Make. O número do módulo pode mudar no seu cenário.

### Exemplo de resumo gerado

O e-mail recebido traz, por exemplo:

- **Diagnóstico geral** do quadro
- **Concluído e mantido na rotina** (hábitos de saúde e tarefas operacionais)
- **Foco de alto impacto** — as 6 prioridades do método Ivy Lee
- **Destaque de atendimentos e pendências** (acadêmico/projetos, financeiro, ação pessoal)
- **Mensagem de encerramento** motivacional

---

## Bem-estar

Para considerar a **saúde física e mental**, o sistema inclui lembretes (na lista **Todos os dias**) para:

- Fazer **pausas** durante o trabalho, para relaxamento mental
- Realizar **alongamentos** e movimentação corporal
- Manter a **ingestão de água** (ex.: 1,5 L pela manhã e 1 L à tarde)

Isso é especialmente importante para quem passa muito tempo sentado, evitando problemas de saúde a longo prazo.

---

## Como reproduzir

### Pré-requisitos

- Conta Google (Sheets, Agenda, Gmail e Apps Script)
- Conta no [Trello](https://trello.com)
- Conta no [Make](https://www.make.com)
- Chave/acesso à API do Google Gemini

### Passo a passo

1. **Planilha:** crie uma planilha no Google Sheets com os dados dos atendimentos e uma coluna de status (ex.: `FINALIZADO`).
2. **Trello:** crie o quadro com as listas descritas [acima](#-estrutura-do-quadro-no-trello).
3. **Make — Fluxo 1:** crie o cenário `Webhooks → Google Sheets → Iterator → Google Calendar → Trello` e copie a URL do Webhook.
4. **Apps Script:** cole o [código](#script-no-apps-script) na planilha, substituindo a URL do Webhook, e crie o acionador `onEdit`.
5. **Teste:** preencha "OK" na coluna de status e confira se o evento e o card foram criados.
6. **Make — Fluxo 2:** crie o cenário `Trello → Text aggregator → Google Gemini AI → Gmail`, com agendamento às 18h30.
7. **Prompts:** configure a mensagem e as instruções do sistema conforme a [seção do Gemini](#configuração-do-modelo).

---

## Resultados e benefícios

- **Nenhum agendamento esquecido**, registrado automaticamente na agenda e no Trello
- **Clareza sobre por onde começar** o dia, graças à lista Ivy Lee
- **Visão do todo** em um fluxo integrado, sem precisar acessar cada sistema separadamente
- **Histórico mensal** de tarefas concluídas
- **Resumo diário por IA** que apoia a revisão e o encerramento do dia
- **Rotina mais equilibrada**, com lembretes de pausa, movimento e hidratação

O projeto cobre uma **pequena parte das atribuições profissionais** da autora, mas evidencia a capacidade e a flexibilidade dessas ferramentas para apoiar a produtividade e a gestão do tempo.

---

## Próximos passos

- Ampliar as integrações para outras partes da rotina e outros sistemas
- Automatizar a montagem da lista **6 atividades – Ivy Lee** com apoio da IA
- Criar lembretes automáticos de pausas, alongamento e água
- Gerar relatórios semanais e mensais de produtividade a partir da lista de concluídos

---

## Aviso sobre dados

Os dados de clientes/alunos exibidos neste projeto (nomes, RAs, telefones e datas) são **fictícios**, criados apenas para exemplificar o funcionamento. **Nunca publique** URLs de Webhook, chaves de API ou endereços de e-mail reais em repositórios públicos.

---

## Autoria

**Priscylla Pfleger** 2026# produtividade_gestao_tempo_ago_2026
# produtividade_gestao_tempo_ago_2026
