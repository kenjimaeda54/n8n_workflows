# n8n Workflows

[![n8n](https://img.shields.io/badge/n8n-2.26.9-red?logo=n8n&logoColor=white)](https://n8n.io)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Coleção de workflows prontos para uso no [n8n](https://n8n.io) — automações inteligentes com agentes de IA, integração com Telegram, Google Sheets, e-mail, MCP, RAG e muito mais.

---

## Índice

- [Visão Geral](#visão-geral)
- [Workflows](#workflows)
  - [Controle Financeiro](#controle-financeiro)
  - [Telegram + AI Assistants](#telegram--ai-assistants)
  - [RAG & Knowledge Base](#rag--knowledge-base)
  - [Email Automation](#email-automation)
  - [MCP Integrations](#mcp-integrations)
  - [Google Sheets](#google-sheets)
  - [RH & Recrutamento](#rh--recrutamento)
  - [Utilitários](#utilitários)
- [Como Usar](#como-usar)
- [Pré-requisitos](#pré-requisitos)
- [Exportação](#exportação)
- [Contribuição](#contribuição)
- [Licença](#licença)

---

## Visão Geral

Este repositório reúne **32 workflows** criados para o n8n, cobrindo desde automações simples (webhooks, e-mail) até sistemas complexos com **agentes de IA**, **RAG (Retrieval-Augmented Generation)** e **MCP (Model Context Protocol)**.

### Principais Recursos

| Recurso | Descrição |
|---------|-----------|
| **Agentes com IA** | Workflows que utilizam OpenAI para processar linguagem natural |
| **RAG** | Recuperação de conhecimento em base vetorial com问答 |
| **Telegram Bot** | Assistente completo com comandos, busca na web e sub-workflows |
| **MCP** | Integração com Model Context Protocol (local e Claude Desktop) |
| **Google Sheets** | Leitura e escrita de dados em planilhas |
| **Controle Financeiro** | Gestão de cartão de crédito via Telegram + Google Sheets (texto e áudio) |
| **RH & Recrutamento** | Análise automática de currículos com IA, scoring e integração via API |
| **Seguros (RAG local)** | Ingestão e consulta de documentos de seguros em base vetorial local (ChromaDB + Ollama) |
| **Email** | Envio/recebimento de e-mails com agente inteligente |

---

## Workflows

### Controle Financeiro

| Workflow | Descrição | Nós |
|----------|-----------|:---:|
| `Finances Credit  WorkFlow.json` | Assistente de controle financeiro via Telegram — registra, edita e consulta gastos do cartão de crédito usando IA. Suporta mensagens de texto e áudio (transcrição automática). | 12 |
| `Schedule finances.json` | Notificação automática a cada 3 horas — lê os dados do ciclo financeiro ativo no Google Sheets e envia um resumo formatado com barras de progresso via Telegram. | 5 |

#### Estrutura da Google Sheets

A planilha de controle financeiro (`controle_financeiro`) deve seguir a estrutura abaixo:

**Abas por ciclo financeiro** (nome em inglês):
- `September/October`, `October/November`, `November/December`, `December/January`, etc.

**Colunas de cada aba de ciclo:**

| GASTO | CATEGORIA | DATA | ID | LUGAR |
|:-----:|:---------:|:----:|:--:|:-----:|
| 250.00 | Alimentação | 2025-09-15 | 001 | iFood |
| 1200.00 | Transporte | 2025-09-18 | 002 | Uber |

> **Regra:** O campo `GASTO` deve conter apenas valores numéricos (sem `R$`, sem ponto como separador de milhar). Usar vírgula como separador decimal (ex: `223,00`). Isso facilita a consulta e soma pela IA.

**Aba `Resume` (resumo geral):**

| MES | LIMITE | GASTO ATUAL | VALOR RESTANTE |
|:---:|:------:|:-----------:|:--------------:|
| September/October | 5000.00 | =SUM('September/October'!A:A) | =B2-C2 |
| October/November | 5000.00 | =SUM('October/November'!A:A) | =B3-C3 |

> **Fórmulas obrigatórias na planilha:**
> - `GASTO ATUAL`: `=SUM('NomeDaAba'!A:A)` — soma automática de todos os gastos do ciclo
> - `VALOR RESTANTE`: `=LIMITE - GASTO ATUAL` — cálculo automático do saldo

#### Funcionalidades do Assistente Financeiro

| Funcionalidade | Exemplo de comando |
|----------------|-------------------|
| Registrar gasto | "Registra 85 reais no iFood categoria alimentação" |
| Consultar gastos | "Quanto gastei esse mês?" |
| Editar gasto | "Altera o valor do gasto 001 para 90 reais" |
| Deletar gasto | "Apaga o gasto 002" |
| Resumo por categoria | "Mostra o resumo por categoria" |
| Via áudio | Enviar mensagem de voz com a instrução |

#### Fluxo dos Workflows

```
┌─────────────────────────────────────────────────────────┐
│  Finances Credit WorkFlow (sub-workflow)                │
│                                                         │
│  Telegram Trigger → Switch (texto/áudio)                │
│       ├─ Texto → Edit Fields → AI Agent → Resposta      │
│       └─ Áudio → Get File → Extract → Transcrição →     │
│                    Edit Fields → AI Agent → Resposta     │
│                                                         │
│  AI Agent Tools:                                        │
│  • tool_append_update_google_sheet (CRUD)               │
│  • tool_delete_google_sheet                             │
│  • tool_read_google_sheet (transações)                  │
│  • tool_read_resume_google_sheet (resumo)               │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Schedule finances (notificação automática)             │
│                                                         │
│  Schedule (3h) → get cycle → get finances by month →    │
│  get resume → transform data → send Telegram message    │
└─────────────────────────────────────────────────────────┘
```

### Telegram + AI Assistants

| Workflow | Descrição | Nós |
|----------|-----------|:---:|
| `Telegram-Assistant-WorkFlow.json` | Assistente completo do Telegram com respostas inteligentes | 21 |
| `Telegram-Multi-Task.json` | Orquestrador multi-tarefa via Telegram | 16 |
| `Calendar-Sub-WorkFlow-Telegram-Multi-Task.json` | Sub-workflow de calendário para o multi-tarefa | 11 |
| `Gmail-Tool-Sub-WorkFlow-Telegram-Multi-Task.json` | Acesso ao Gmail via sub-workflow do Telegram | 11 |
| `Google-Sheets-Customer-Sub-Workflow-Telegram-Multi-Task.json` | Consulta de clientes em planilhas via Telegram | 8 |
| `Telegram-Search-Web.json` | Busca na web diretamente pelo Telegram | 6 |
| `Telegram-Sub-WorkFlow-Search-Internet.json` | Sub-workflow de pesquisa na internet | 6 |
| `Sub-Workflow-X-Posts-Twitter.json` | Sub-workflow que gera posts para X (Twitter): recebe o tema, um agente de IA (OpenAI) escreve o post e tenta salvar/enviar, com nova tentativa automática em caso de erro | 6 |

### RAG & Knowledge Base

| Workflow | Descrição | Nós |
|----------|-----------|:---:|
| `Rag-with-chatbot.json` | Chatbot com RAG para respostas contextuais | 9 |
| `RAG-App-with-ChatBoot-Q&A---Gold-Smith.json` | Aplicação RAG Q&A — Gold Smith | 9 |
| `RAG-with-database-vector.json` | RAG com banco de dados vetorial | 6 |
| `RAG-App-save-Q&A---Gold-Smith.json` | Salvamento de Q&A em base vetorial — Gold Smith | 6 |
| `Ingestion Insurances File Sub Workflow.json` | Ingestão de documentos de seguros em base vetorial (ChromaDB): a cada 10 horas lê os arquivos da pasta local, extrai o texto, divide em chunks e gera embeddings locais com Ollama. Sub-workflow de ingestão. | 14 |
| `Retrieve Insurance Local.json` | Chatbot de consulta aos documentos de seguros: agente de IA (Ollama) com memória e ferramenta de busca na base vetorial ChromaDB (`retrieve_documents`), acessado via chat. | 9 |

### Email Automation

| Workflow | Descrição | Nós |
|----------|-----------|:---:|
| `Pinnecode-send-email.json` | Envio de e-mail com Pinnecode + RAG | 9 |
| `Agent-email.json` | Agente inteligente para processar e-mails | 5 |
| `Receive-email-by-pinnecode.json` | Recebimento de e-mails via Pinnecode | 4 |
| `Send-To-email.json` | Envio simples de e-mail | 3 |
| `Email-to-pinnecode.json` | Encaminhamento de e-mail para Pinnecode | 6 |
| `Supornship-reply.json` | Resposta automática de suporte | 12 |

### MCP Integrations

| Workflow | Descrição | Nós |
|----------|-----------|:---:|
| `MCP-local.json` | Servidor MCP local para ferramentas personalizadas | 5 |
| `MCP-with-claude-desktop.json` | Integração MCP com Claude Desktop | 3 |

### Google Sheets

| Workflow | Descrição | Nós |
|----------|-----------|:---:|
| `Felling-Google-Sheets.json` | Registro de sentimentos em planilha Google | 5 |
| `Finances Credit  WorkFlow.json` | Consulta/edição de dados financeiros em planilha | 12 |
| `Schedule finances.json` | Leitura de planilha para notificação automática | 5 |

### RH & Recrutamento

| Workflow | Descrição | Nós |
|----------|-----------|:---:|
| `analyze_resume.json` | Triagem automática de currículos: recebe uma candidatura via webhook (com autenticação JWT), baixa o PDF do currículo, extrai o texto e um agente de IA avalia a compatibilidade com as palavras-chave da vaga gerando um score (0–1). O resultado é gravado no Airtable, enviado por e-mail (Gmail) e retornado à API via callback HTTP. | 14 |

### Utilitários

| Workflow | Descrição | Nós |
|----------|-----------|:---:|
| `Whatsapp-Test-Agent.json` | Agente de teste para WhatsApp | 8 |
| `Write-and-Read-Files.json` | Leitura e escrita de arquivos | 4 |
| `Simple-webhook.json` | Webhook simples de exemplo | 3 |
| `first_automation.json` | Primeira automação (exemplo inicial) | 2 |
| `hotel_second_automation.json` | Automação para reservas de hotel | 0 |
| `My-workflow.json` | Exemplo de automação agendada: dispara em intervalos regulares e lê/escreve arquivos no disco | 2 |

---

## Como Usar

### Instalação Global (recomendado)

Para rodar o n8n localmente e ter acesso aos 32 workflows desta coleção, instale o n8n globalmente:

```bash
npm install -g n8n
```

> **Importante:** o n8n **não lê os arquivos `.json` desta pasta automaticamente**. Para que os workflows apareçam no painel, você precisa importá-los para o banco de dados do n8n (passo 2 abaixo) — e o comando de importação deve ser rodado **na pasta do projeto** (a raiz do repositório), porque o `--input=./workflows` é um caminho relativo.

### 1. Iniciar o n8n

```bash
n8n start
```

O painel abre em `http://localhost:5678`.

> **Nota:** se você iniciar o n8n pela primeira vez sem importar, verá apenas alguns (ou nenhum) workflow — o painel mostra apenas o que já está no banco de dados dele (`~/.n8n/database.sqlite`). Isso é resolvido pelo próximo passo.

### 2. Importar todos os workflows

**Na pasta do projeto** (a raiz do repositório), importe a pasta `workflows/` para o banco de dados do n8n:

```bash
n8n import:workflow --separate --input=./workflows
```

- `--separate` importa cada arquivo `.json` como um workflow distinto
- Após a importação, inicie o `n8n start` novamente — todos os workflows aparecem no painel

### 3. Exportar do banco para a pasta

Se você criou workflows na interface do n8n e quer salvá-los nesta pasta (para versionar no git), o caminho inverso também funciona:

```bash
n8n export:workflow --backup --output=./workflows
```

- `--backup` gera um arquivo `.json` por workflow (formato ideal para repositório)
- Sem `--backup`, ele gera um único `.json` com tudo dentro — não use para esta pasta

### 4. Importar um Workflow manualmente

1. Acesse o painel do n8n
2. Vá em **Workflows** → **Add Workflow** → **Import from File**
3. Selecione o arquivo `.json` desejado da pasta `workflows/`
4. Configure as credenciais necessárias (API keys, tokens, etc.)

### 5. Credenciais Necessárias

| Workflow | Credenciais |
|----------|-------------|
| Telegram | Bot Token do Telegram |
| Email (IMAP/SMTP) | Credenciais de e-mail |
| Google Sheets | OAuth2 Google |
| OpenAI | API Key da OpenAI |
| Pinnecode | API Key do Pinnecode |
| WhatsApp | Configuração do canal WhatsApp |
| Controle Financeiro | Telegram Bot Token + Google Sheets OAuth2 + OpenAI API Key + OpenRouter Bearer Auth (para transcrição de áudio) |
| Análise de Currículos | JWT Auth (webhook) + OpenRouter API + Gmail OAuth2 + Airtable Token + HTTP Bearer Auth (callback da API) |

### 6. Variáveis de Ambiente (n8n)

```env
N8N_ENCRYPTION_KEY=sua_chave_de_criptografia
WEBHOOK_URL=https://seu-dominio.com/
```

---

## Pré-requisitos

- [n8n](https://n8n.io) (versão 2.26.9+)
- Docker (para deploy com docker-compose)
- Node.js 18+ (para instalação local)

### Deploy com Docker

```yaml
version: '3.8'
services:
  n8n:
    image: docker.io/n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    volumes:
      - ./n8n_data:/home/node/.n8n
    environment:
      - N8N_ENCRYPTION_KEY=sua_chave
      - WEBHOOK_URL=https://seu-dominio.com/
```

---

## Exportação

Para exportar todos os workflows do seu servidor n8n:

```bash
# Via CLI do n8n (dentro do container Docker)
sudo docker exec <container_name> n8n export:workflow --backup --output=/tmp/workflows

# Copiar para o host
sudo docker cp <container_name>:/tmp/workflows ./workflows_exportados
```

---

## Contribuição

1. Faça um fork do repositório
2. Crie uma branch: `git checkout -b feat/nova-automacao`
3. Adicione seu workflow na pasta `workflows/`
4. Faça commit: `git commit -m "feat: novo workflow de ..."`
5. Envie: `git push origin feat/nova-automacao`
6. Abra um Pull Request

---

## Licença

Distribuído sob a licença MIT. Veja [LICENSE](LICENSE) para mais informações.

---

<p align="center">
  Feito com ❤️ usando <a href="https://n8n.io">n8n</a>
</p>