# Oráculo
Este projeto almeja desenvolver uma plataforma de chatbot para agilizar e simplificar respostas de perguntas sobre o estado das tarefas de uma equipe de desenvolvimento.

Para este fim, planejamos integrar ferramentas como repositório do Github, JIRA... e utilizando uma IA para fazer queries SQL e retornar a resposta certa.


## Sobre o projeto
Um chatbot que utiliza a IA para pesquisar em um banco com  dados, de ferramentas integradas (Github, JIRA, ...), afim de agilizar uma resposta à uma pergunta do tipo: O que **membro da equipe de desenvolvimento** está trabalhando agora?

Criado para simplificar o processo de gerenciamento de uma equipe.

## Pré-requisitos

Antes de começar, certifique-se de que você tem os seguintes requisitos instalados:
- [Docker](https://www.docker.com/)
  - Utilizado para subir o banco de dados Postgres, na versão 15.
- [Python 3.10](https://www.python.org/)
  - Utilizar a lib Airbyte para buscar dados do Github

## Instalação
Siga estas etapas para configurar o projeto localmente:

1. Gere um token pessoal no [Github](https://github.com) e insira com a chave **GITHUB_TOKEN** no arquivo .env

    - Pode ser gerado [**aqui**](https://github.com/settings/tokens)


2. Utilize os comandos a seguir para iniciar e parar contêiner com o banco de dados

    Para iniciar o contêiner:
    ```bash
      docker compose up -d
    ```

    Para parar o contêiner:
    ```bash
      docker compose down -d
    ```

3. Inicie um ambiente virtual e ative-o

    Iniciando um ambiente virtual:
    ```bash
      python -m venv .venv
    ```

    Ativando o ambiente virtual, no Linux e MacOS:
    ```bash
      source .venv/bin/activate
    ```

    No Windows Powershell:
    ```bash
      .venv\Scripts\Activate.ps1
    ```

    Desativando o ambiente virtual:
    ```bash
      deactivate
    ```

4. Instale os requerimentos do projeto com o comando:
    ```bash
      pip install -r py_requirements.txt
    ```

5. Após sucesso na instalação dos requerimentos, rode o arquivo python principal:
    ```bash
      python main.py
    ```

    Ou inicie a aplicação FastAPI com:
    ```bash
    uvicorn src.app:app --reload
    ```

## Uso da API

Uma vez que a aplicação esteja em execução, você pode enviar uma requisição POST para o endpoint `/ask` com um corpo JSON contendo sua pergunta. Por exemplo:

```json
{
  "question": "Me liste os produtos e suas quantidades em estoque"
}
```

A aplicação retornará o resultado da consulta SQL gerada com base na sua pergunta.

## Uso do n8n e OpenWebUI

Para abrir as ferramentas, acesse:

- **n8n**: [http://localhost:5678](http://localhost:5678)  
- **OpenWebUI**: [http://localhost:3000](http://localhost:3000)

### n8n

Insira no workflow a API do AI Agent.  
Para obter a API Key, acesse: [https://ai.google.dev/gemini-api/docs/api-key?hl=pt-br](https://ai.google.dev/gemini-api/docs/api-key?hl=pt-br)

### OpenWebUI

Adicione a pipeline como uma nova função e altere o endpoint, utilizando o webhook do n8n.  
**Atenção**: Substitua `'localhost'` por `'host.docker.internal'` para garantir a comunicação correta entre os containers.

## Arquitetura e Modularização

O projeto é dividido em módulos bem definidos que seguem uma arquitetura desacoplada, com responsabilidades específicas. Abaixo, explicamos de forma clara o papel de cada componente:

### Componentes Principais

- **🔁 Airbyte (ETL)**  
  Responsável por extrair dados de fontes externas como o GitHub. Ele coleta essas informações e envia para o banco de dados.

- **⚙️ Backend (FastAPI)**  
  API desenvolvida em FastAPI, responsável por receber as perguntas, processá-las com ajuda da IA (Vanna.AI), gerar a consulta SQL e retornar a resposta ao usuário.  
  Local: `src/fastapi/`

- **🧠 Vanna.AI (LLM)**  
  Modelo de linguagem usado para interpretar perguntas em linguagem natural e gerar a SQL correspondente.  
  Local: `src/vanna/`

- **🌐 OpenWebUI (Interface)**  
  Interface Web usada para interagir com o usuário final. Permite enviar perguntas e visualizar respostas.  
  Local: `src/open-web-ui/`

- **🔗 n8n (Automação)**  
  Plataforma de automação que conecta o OpenWebUI ao backend via Webhook. Gerencia a comunicação entre as partes.  
  Local: `src/n8n/`

---

### Visão Geral do Fluxo de Dados

```
Usuário (interface OpenWebUI)
         ↓
      Webhook
         ↓
       n8n (automação)
         ↓
  FastAPI (backend/API)
         ↓
     Vanna.AI (LLM)
         ↓
    SQL → Banco de dados
         ↓
   ↪ Resposta exibida ao usuário
```

---

### Resumo da Arquitetura por Papel

| Componente     | Papel              | Descrição                                                                 |
|----------------|--------------------|---------------------------------------------------------------------------|
| OpenWebUI      | Interface           | Frontend para o usuário interagir com o sistema                          |
| FastAPI        | Backend/API         | Processa as perguntas e coordena as respostas                            |
| Airbyte        | ETL                 | Coleta dados externos e injeta no banco de dados                         |
| Vanna.AI       | LLM / IA            | Converte perguntas em SQL com base na linguagem natural                  |
| n8n            | Orquestrador        | Encaminha dados entre frontend, backend e IA usando Webhooks             |
| Postgres/Chroma| Banco de Dados      | Armazena dados coletados e usados pela IA                                |


## Estrutura de diretórios

```
Oraculo/
├── .github/                 
├── src/                    
│   ├── etl/
│   │     └── airbyte.py             
│   ├── fastapi/
│   │    ├── api/
│   │    │    └── routes.py
│   │    ├── database/
│   │    │    └── vanna_client.py
│   │    ├── models/
│   │    │    └── query.py
│   │    ├── app.py
│   │    ├── chroma.sqlite3
│   │    └── config.py
│   ├── n8n/
│   │     └── My_workflow.json      
│   ├── open-web-ui/
│   │     └── pipe-conexaoWebhook.py          
│   └── vanna/
│         └── vanna-ai.py                      
├── .gitignore
├── chroma.sqlite3             
├── CODE_OF_CONDUCT.md      
├── CONTRIBUTING.md                             
├── docker-compose.yml      
├── example.env             
├── main.py
├── py_requirements.txt                 
└── README.md     
```