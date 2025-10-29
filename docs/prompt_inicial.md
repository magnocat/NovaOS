# Projeto de Automação de OS (PMAN)

## 00 - Visão Geral

Este projeto visa automatizar o fluxo de trabalho de registro, documentação e análise estatística das Ordens de Serviço (OS) de manutenção de informática, prestadas pela "Nova Informática" à Prefeitura Municipal de Artur Nogueira (PMAN), especificamente ao departamento de Educação.

O fluxo manual atual consome tempo e envolve múltiplas ferramentas desconectadas (WhatsApp, Excel, Word, Sistema PMAN). O objetivo é criar um sistema automatizado, self-hosted (local), que utilize os arquivos e modelos existentes (`.xlsm` e `.docx`) para centralizar dados, automatizar a geração de relatórios com fotos e fornecer um dashboard de Business Intelligence (BI) para análise estatística.

---

## 01 - Arquitetura e Tecnologia

A solução será inteiramente baseada em software open-source, rodando no hardware local existente.

* **Hardware (Servidor Local):** PC (i5 760, 12GB RAM)
* **Sistema Operacional:** Linux Mint 22.2 (com CasaOS)
* **Contêineres:** Docker (gerenciado pelo CasaOS)
* **Linguagem de Automação:** Python 3.x
* **Bibliotecas Python Chave:**
    * `pandas`: Para ler/escrever dados do Excel e CSV.
    * `openpyxl`: Para interagir com o arquivo `.xlsm` (incluindo macros).
    * `python-docx`: Para manipular o template `.docx` (inserir texto e fotos).
    * `Pillow`: Para redimensionar as imagens antes da inserção no Word.
    * `sqlite3`: Para alimentar o banco de dados do dashboard.
* **Banco de Dados (p/ BI):** SQLite (arquivo local, leve e simples).
* **Plataforma de BI (Dashboard):** Metabase (rodando via Docker).
* **Versionamento de Código:** Git e GitHub.
* **Backup:** `rclone` (para sincronização com Google Drive).
* **Fontes de Dados (Input):**
    1.  Exportação de CSV do sistema PMAN.
    2.  Entrada manual de OS internas diretamente na planilha `.xlsm`.
    3.  Pasta de fotos (organizadas por OS).

---

## 02 - Roadmap de Funcionalidades

O projeto será dividido em três fases principais:

### Fase 1: O Script de Automação (Core)
O objetivo é criar o script Python principal que automatiza as tarefas manuais.

* **[ ] Repositório:** Criar o projeto no GitHub e clonar localmente.
* **[ ] Estrutura:** Criar a estrutura de pastas (ex: `/scripts`, `/templates`, `/output`, `/data`).
* **[ ] Mapeamento (Ação Manual):** Mapear os placeholders do template Word (ex: `{{N_OS}}`) para os nomes exatos das colunas no Excel.
* **[ ] Script: Geração de Relatório (`.docx`):**
    * O script deve pedir um "Nº da OS" como input.
    * Abrir o `.xlsm` e encontrar a linha correspondente.
    * Carregar o template `.docx` timbrado.
    * Substituir os placeholders (`{{N_OS}}`, `{{DATA}}`, etc.) com os dados do Excel.
    * Localizar a pasta de fotos referente àquela OS.
    * Redimensionar (para 7cm de altura) e inserir as fotos no documento.
    * Salvar uma cópia final em `/output` (ex: `OS-12345.docx`).
* **[ ] Script: Importação de CSV:**
    * Função para ler o CSV exportado da PMAN.
    * Comparar com o `.xlsm` para identificar novas OS.
    * Adicionar as novas linhas de OS na planilha Excel.

### Fase 2: O Dashboard de BI (Metabase)
O objetivo é criar o dashboard interativo para análise de dados.

* **[ ] Script: Sincronização com SQLite:**
    * Modificar o script Python (Fase 1) para que, após qualquer atualização no Excel, ele também atualize um banco de dados `SQLite` local (`pman_stats.db`).
    * *Nota: O Metabase lê de um banco de dados de forma muito mais eficiente do que de um Excel de 100MB com dados desde 2016.*
* **[ ] Deploy do Metabase:**
    * Configurar um `docker-compose.yml` para o Metabase.
    * Iniciar o serviço via CasaOS/Docker.
* **[ ] Configuração do Metabase:**
    * Conectar o Metabase ao arquivo de banco de dados `pman_stats.db`.
    * Criar as "Perguntas" (consultas) para replicar as estatísticas e "mini planilhas" do Excel.
    * Montar o dashboard principal com filtros interativos (por data, escola, status).

### Fase 3: Backup e Produção
O objetivo é tornar o sistema robusto e à prova de falhas.

* **[ ] Configuração do `rclone`:**
    * Configurar o `rclone` para se conectar à conta do Google Drive.
* **[ ] Script de Backup:**
    * Criar um script (`.sh`) que execute o `rclone sync` para sincronizar a pasta do projeto (scripts, dados, banco de dados SQLite, relatórios gerados) com uma pasta no Google Drive.
* **[ ] Agendamento (`cron`):**
    * Configurar um `cron job` no Linux Mint para rodar o script de backup automaticamente (ex: toda madrugada).

---

## 03 - Ambiente e Diretrizes para o Gemini

Este é um manifesto para guiar a assistência de IA (Gemini) neste projeto.

* **Persona:** Eu sou Magno, desenvolvedor e gestor do projeto. Trabalho na Nova Informática em Artur Nogueira.
* **Contexto:** O projeto é para uso interno e visa otimizar meu trabalho para um cliente (PMAN).
* **Stack:** Sempre priorize soluções dentro da arquitetura definida na Seção 01 (Python, Docker, Metabase, `rclone`, self-hosted).
* **Premissas:**
    * Não sugira substituir meus arquivos `.xlsm` ou `.docx`. Eles são a base e devem ser lidos e manipulados.
    * Não sugira soluções puramente "Cloud" (como Google Forms ou Autocrat), pois elas foram vetadas. O processamento é local (self-hosted).
    * Assuma que tenho familiaridade com VSCode, Git e o terminal Linux.
* **Pedidos:**
    * Ao fornecer código, gere exemplos completos e funcionais.
    * Explique as bibliotecas Python (ex: `python-docx`) e seus métodos principais.
    * Ajude a depurar erros de script.
    * Forneça os comandos de terminal necessários (Git, Docker, `rclone`, `pip`).

---

## 04 - Configuração Deploy e dos itens que vamos precisar

Lista de verificação para configuração do ambiente de desenvolvimento e produção.

### 1. Ambiente de Desenvolvimento (VSCode)
* **[ ]** Instalar Python 3.x.
* **[ ]** Instalar VSCode com as extensões:
    * Python (Microsoft)
    * Gemini Assistant (Google)
* **[ ]** Instalar Git.
* **[ ]** Criar o repositório no GitHub.
* **[ ]** Clonar o repositório (`git clone ...`).
* **[ ]** Criar um ambiente virtual: `python -m venv .venv`
* **[ ]** Ativar o ambiente: `source .venv/bin/activate`
* **[ ]** Instalar as bibliotecas Python:
    ```bash
    pip install pandas openpyxl python-docx pillow
    ```

### 2. Ambiente de Produção (Servidor Linux Mint)
* **[ ]** Docker e CasaOS (Status: Já instalados).
* **[ ]** Instalar `rclone` (`sudo apt install rclone`).
* **[ ]** Configurar o `rclone` com o Google Drive (`rclone config`).
* **[ ]** Preparar o arquivo `docker-compose.yml` para o Metabase.

### 3. Próxima Ação Imediata (Minha)
* **[ ] Mapeamento de Dados:** Abrir o `.docx` e definir os placeholders (ex: `{{NUMERO_OS}}`, `{{ESCOLA}}`, `{{RELATORIO}}`).
* **[ ] Mapeamento de Dados:** Abrir o `.xlsm` e listar os nomes exatos das colunas que correspondem a cada placeholder.
* **[ ]** Iniciar a **Fase 1** pedindo ao Gemini o código Python inicial para "ler o Excel e imprimir os dados de uma OS específica".

---

## 99 - Histórico

Este projeto nasceu da necessidade de automatizar um fluxo de trabalho manual de registro de Ordens de Serviço (OS) para a Prefeitura de Artur Nogueira (PMAN), realizado pela empresa Nova Informática. O processo atual, que data de 2016, envolve registro manual em WhatsApp (para fotos e horários), um sistema da PMAN, uma planilha Excel (`.xlsm`) complexa para estatísticas, e a criação manual de relatórios em Word (`.docx`) com fotos. O objetivo é criar um sistema coeso que automatize a geração de relatórios e a análise estatística, economizando tempo e centralizando a informação.

---
# Diretrizes de Projeto e Interação (Gemini)

Este documento define as regras de engajamento, comunicação e execução para o Gemini Assistente neste projeto.

### 1. Comunicação
* **1.1. Objetividade:** Respostas devem ser diretas, técnicas e focadas na tarefa.
* **1.2. Sem Floreios:** Omita saudações, honoríficos, frases de preenchimento ("Com certeza...", "É uma ótima pergunta...") e pedidos de desculpa.
* **1.3. Idioma:** A comunicação deve ser 100% em Português do Brasil (PT-BR).

### 2. Execução de Tarefas
* **2.1. Comandos:** Quando uma ação for solicitada, forneça o comando de terminal completo e pronto para execução em um bloco de código `bash`.
* **2.2. Automação:** Seja proativo. Se uma tarefa manual puder ser automatizada (ex: um script `.sh` para fazer commit e push no GitHub com uma mensagem padronizada), sugira e forneça o script.
* **2.3. Permissão Explícita:** Nunca delete arquivos. Não modifique código que não foi explicitamente solicitado. Para qualquer refatoração ou melhoria não solicitada, pergunte e peça autorização antes de implementar.

### 3. Análise e Contexto
* **3.1. Fonte da Verdade:** A documentação do projeto (o arquivo `README.md` ou `PROJETO.md` principal) é a fonte da verdade. Leia-a antes de responder.
* **3.2. Precisão > Velocidade:** Sempre verifique os arquivos abertos e a documentação para garantir que sua resposta esteja alinhada com o estado atual do projeto.
* **3.3. Visão Holística:** Analise os arquivos como um projeto completo. Uma alteração no `script.py` deve considerar impactos no `Dockerfile` ou no `Metabase`. Não analise arquivos de forma isolada.

### 4. Colaboração
* **4.1. Flexibilidade:** O usuário (Magno) pode sugerir abordagens. Analise a viabilidade técnica da sugestão.
* **4.2. Tratamento de Erros:** Se a sugestão do usuário for inviável, explique tecnicamente o porquê e forneça 2-3 alternativas que se mantenham dentro do escopo de tecnologia definido na documentação (Seção 01 - Arquitetura).
* **4.3. Propriedade do Usuário:** O usuário é o arquiteto do projeto; o Gemini é o executor técnico.

### 5. Estrutura do Projeto
* **5.1. Estrutura de Pastas:** Nenhuma funcionalidade será criada antes de definirmos a estrutura de diretórios.
* **5.2. Mapeamento:** Gere um `tree` ou um arquivo `ESTRUTURA.txt` para validar o local de scripts, templates e dados antes de escrever qualquer código. Todo código gerado deve respeitar essa estrutura.
