# 🚀 RETAIL-DATA-SAAS: Plataforma de BI e Dados End-to-End para Varejo

![Status do Projeto](https://img.shields.io/badge/status-Em_Desenvolvimento-orange?style=for-the-badge)

Este projeto é um **case de estudo** que demonstra a construção de uma solução de dados completa (Data SaaS) para uma PME de varejo, transformando dados rústicos em inteligência de negócio acionável.

---

## 1. O Problema de Negócio

Uma empresa de varejo de médio porte estava em rápido crescimento, mas sua infraestrutura de dados não acompanhava. A gestão era baseada em métodos rústicos e descentralizados preenchidos manualmente por uma pequena equipe consequentemente vindo com diversos erros nos dados:

* **Compra dos produtos:** Exportadas em arquivos `.csv` do sistema de gerenciamento de compras da empresa.
* **Estoque:** Controlado manualmente em planilhas `.xlsx`.
* **Histórico de promoções:** xml retirado do sistema de promoções da empresa.
* **log de vendas:** Um log das vendas feitas em `jsonl`.
* **Preço dos produtos:** uma anotação feita para precificar os produtos `txt`.

Essa fragmentação impedia a gestão (Gerentes, Diretores) de ter uma visão clara para tomar decisões, causando perda de eficiência, dificuldade no controle de estoque e incerteza sobre a lucratividade. Com o crescimento do negocio e expanção de lojas o dono da empresa me contratou para resolver seu problema de visualição de dados tanto para a equipe que está em crescimento quando para os gerentes, diretores e o proprio dono.

## 2. A Solução e o Valor Agregado

Para resolver esse problema, foi arquitetada e implementada uma **plataforma de dados End-to-End (E2E)**, entregue como um produto de Software as a Service (SaaS).

O **valor** desta solução é **centralizar, limpar e democratizar** o acesso à informação, permitindo que cada nível da empresa consuma os dados da forma que precisa:

* **Para a Equipe Operacional:** Dashboards visuais em Power BI.
* **Para a Diretoria (CEO):** Um bot de IA no Telegram para respostas rápidas na palma da mão.
* **Para Desenvolvedores (Outros Sistemas):** Uma API segura para integração.

### Arquitetura da Solução

O projeto automatiza todo o ciclo de vida dos dados em 6 etapas:

1.  **Extract:** Coleta dos 5 arquivos brutos e "sujos".
2.  **Transform:** Um pipeline de ETL robusto com **Pandas** para limpar, tratar tipos, padronizar e unificar os dados.
3.  **Load:** Carga dos dados limpos e prontos para consumo em um Data Mart centralizado (**MYSQL**).
4.  **Presentation (BI):** 3 Dashboards em **Power BI** (Comercial, Operacional, Estratégico).
5.  **Access (API):** Uma API RESTful segura construída em **Flask**, com endpoints públicos e privados.
6.  **AI (BI Conversacional):** Um Bot de **Telegram** que usa IA (OpenAI) e *Function Calling* para "conversar" com a API e responder perguntas do CEO em linguagem natural.

## 3. Mergulhe na Documentação Técnica

Este `README` é a visão geral. Para uma análise técnica profunda de *como* cada parte foi construída, acesse os "capítulos" abaixo:

[![Botão ETL](https://img.shields.io/badge/Ver_Detalhes-Pipeline_de_ETL_e_Tratamento-blue?style=for-the-badge&logo=Python)](documentation/etl_pipeline.md)

*(Em breve: Botões para a Documentação da API, dos Dashboards e do Bot de IA)*

## 4. Tecnologias Utilizadas (Tech Stack)

* **Linguagem de Programação:** Python 3.10+
* **Pipeline de ETL:** Pandas, PyArrow, Openpyxl
* **Banco de Dados (Data Mart):** SQLite
* **API (Camada de Acesso):** Flask
* **Visualização de Dados (BI):** Power BI
* **IA & Bot:** Telegram Bot API, OpenAI

## 5. Como Rodar o MVP (Instruções)

*(Esta seção será preenchida conforme o projeto avança)*

**Pré-requisitos:**
* Python 3.10+
* Power BI Desktop

**1. Clone o Repositório:**
```bash
git clone https://github.com/joaovitorvieiradev/RETAIL-DATA-SAAS.git
cd RETAIL-DATA-SAAS