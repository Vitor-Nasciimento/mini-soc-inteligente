# Mini SOC Inteligente

Projeto prático de estudo em Segurança da Informação, desenvolvido para simular um fluxo básico de monitoramento de endpoints, análise de eventos e automação de alertas.

O projeto utiliza **Wazuh**, **n8n**, **Ollama** e **Telegram** para criar um fluxo de monitoramento de um endpoint Windows, desde a geração do evento até o envio de um alerta interpretado.

> Este projeto também faz parte do desenvolvimento do TCC **“Guia de Boas Práticas de Monitoramento de Endpoint Windows 10 com SIEM Wazuh e N8N”**.

---

## 🎯 Objetivo

O objetivo do projeto é estudar, na prática, conceitos relacionados a:

* Monitoramento de endpoints
* SIEM
* Coleta e análise de logs
* Regras de detecção
* Automação de processos
* Webhooks
* Integração entre ferramentas de Segurança da Informação
* Uso de Inteligência Artificial como apoio à interpretação de alertas

A proposta não é substituir a análise de um profissional de Segurança da Informação, mas demonstrar como diferentes ferramentas podem ser integradas em um fluxo de monitoramento.

---

## 🏗️ Arquitetura

O fluxo principal do projeto é:

```text
Windows 10
    │
    ▼
Wazuh Agent
    │
    ▼
Wazuh Manager
    │
    ▼
Regra customizada
    │
    ▼
n8n Webhook
    │
    ▼
Normalização dos dados
    │
    ▼
Preparação do contexto
    │
    ▼
Ollama + Qwen 2.5 1.5B
    │
    ▼
Interpretação do alerta
    │
    ▼
Geração do relatório
    │
    ▼
Telegram
```

---

## 🔧 Tecnologias utilizadas

| Tecnologia    | Função                                  |
| ------------- | --------------------------------------- |
| Windows 10    | Endpoint monitorado                     |
| Wazuh         | SIEM e monitoramento do endpoint        |
| Wazuh Agent   | Coleta de eventos do Windows            |
| Docker        | Execução dos serviços do Wazuh e n8n    |
| n8n           | Automação e integração do fluxo         |
| Ollama        | Execução local do modelo de IA          |
| Qwen 2.5 1.5B | Interpretação auxiliar dos eventos      |
| Telegram      | Recebimento das notificações            |
| PowerShell    | Geração e testes de eventos             |
| Git/GitHub    | Versionamento e documentação do projeto |

---

## 🔄 Funcionamento

### 1. Monitoramento do Windows

O Wazuh Agent é instalado no endpoint Windows e envia eventos para o Wazuh Manager.

Entre os eventos utilizados nos testes estão eventos relacionados ao gerenciamento de contas de usuário do Windows.

### 2. Regra personalizada

Foi criada uma regra personalizada no Wazuh com o ID `100003`.

A regra é utilizada para identificar o evento definido no laboratório e gerar um alerta com severidade 10.

Arquivo:

```text
fluxos/wazuh/local_rules.xml
```

### 3. Integração com o n8n

Quando a regra é acionada, o Wazuh envia o alerta em formato JSON para um webhook do n8n.

O n8n recebe os dados e realiza a primeira etapa de tratamento.

### 4. Normalização

Um nó de código JavaScript extrai informações relevantes do alerta, como:

* Severidade
* ID da regra
* Endpoint
* IP
* Evento do Windows
* Usuário envolvido
* Data e hora
* Resultado da auditoria

Isso transforma o alerta original em uma estrutura mais simples para as próximas etapas do fluxo.

### 5. Análise assistida por IA

O n8n prepara os dados e envia o contexto do alerta para um agente utilizando o modelo local **Qwen 2.5 1.5B**, executado através do Ollama.

A IA possui uma função limitada: **explicar em linguagem simples o significado do evento recebido**.

Ela não é responsável pela detecção original do evento.

### 6. Geração do relatório

Após a interpretação da IA, outro nó JavaScript monta o relatório final.

As recomendações do fluxo são definidas pelo próprio código, e não pela IA.

Isso foi utilizado para reduzir a possibilidade de a IA gerar recomendações não previstas no projeto.

### 7. Notificação

O relatório final é enviado para um chat do Telegram através da integração do n8n.

---

## 🧠 Uso da Inteligência Artificial

A Inteligência Artificial é utilizada como mecanismo auxiliar de interpretação.

O Wazuh continua sendo a fonte dos dados e responsável pela geração do alerta.

A IA recebe os dados estruturados pelo n8n e produz uma explicação curta sobre o evento registrado.

Durante os testes, foram observadas situações em que modelos menores poderiam gerar interpretações que não estavam presentes nos dados originais.

Por esse motivo, o projeto considera a IA como **apoio à análise**, e não como fonte dos dados ou mecanismo autônomo de decisão.

A validação dos dados permanece baseada nas informações fornecidas pelo Wazuh e pelos eventos registrados no Windows.

---

## 🧪 Testes realizados

Um dos testes realizados consistiu na geração de um evento relacionado à criação/ativação de uma conta de usuário no Windows.

Exemplo utilizado durante os testes:

```powershell
net user WazuhTesteFinal2 Senha123! /add
```

O evento foi identificado pelo Wazuh, processado pelo n8n e posteriormente enviado para o Telegram.

Exemplo simplificado do resultado:

```text
Severidade Wazuh: 10
Regra: 100003
Endpoint: DESKTOP-53EOBML
Evento Windows: 4722
Usuário afetado: WazuhTesteFinal2
Usuário responsável: vitin
```

O relatório também recebeu uma interpretação produzida pela IA local.

As evidências dos testes estão disponíveis em:

```text
evidencias/
```

---

## 📂 Estrutura do projeto

```text
mini-soc-inteligente/
│
├── README.md
│
├── docs/
│
├── evidencias/
│   ├── Dia 04/
│   └── Dia 05/
│
├── fluxos/
│   ├── n8n/
│   │   └── mini-soc-workflow.json
│   │
│   └── wazuh/
│       └── local_rules.xml
│
├── imagens/
│
└── scripts/
```

---

## 📋 Fluxo do n8n

O workflow utilizado no projeto está disponível em:

```text
fluxos/n8n/mini-soc-workflow.json
```

O fluxo contém as etapas de:

```text
Webhook
   ↓
Code - Normalização
   ↓
Preparar contexto IA
   ↓
AI Agent
   ↓
Ollama Chat Model
   ↓
Code - Geração do relatório
   ↓
Telegram
```

As credenciais utilizadas no ambiente original não estão armazenadas no arquivo disponibilizado neste repositório.

---

## ⚠️ Limitações

Este projeto foi desenvolvido como laboratório de estudo e demonstração.

Algumas limitações são:

* O ambiente utiliza infraestrutura local.
* O modelo de IA utilizado possui tamanho reduzido.
* A IA pode apresentar interpretações imprecisas.
* A detecção depende das regras e eventos configurados no Wazuh.
* O fluxo não representa uma implementação completa de um SOC corporativo.
* As recomendações apresentadas pelo fluxo são específicas para os eventos trabalhados no laboratório.
* O projeto não deve ser utilizado como substituto de uma análise realizada por um profissional de Segurança da Informação.

---

## 📚 Contexto acadêmico

O projeto está relacionado ao desenvolvimento do TCC:

**Guia de Boas Práticas de Monitoramento de Endpoint Windows 10 com SIEM Wazuh e N8N**

Além do objetivo acadêmico, o laboratório também foi desenvolvido como projeto pessoal para praticar conceitos de:

* Blue Team
* SOC
* SIEM
* Monitoramento de endpoints
* Windows Event Logs
* Automação
* Docker
* Linux
* Redes
* Inteligência Artificial aplicada à Segurança da Informação

---

## 📌 Status do projeto

**Em desenvolvimento.**

O fluxo principal de monitoramento, integração Wazuh → n8n, interpretação auxiliar com IA e envio de alertas para Telegram já foi implementado e testado.

Novas melhorias e documentações poderão ser adicionadas ao longo do desenvolvimento do projeto.
