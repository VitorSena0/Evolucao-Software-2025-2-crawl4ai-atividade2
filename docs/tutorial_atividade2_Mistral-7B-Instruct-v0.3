# Tutorial Técnico: Auditoria de Engenharia de Software com Mistral-7B

Este guia descreve o funcionamento do notebook desenvolvido para a análise técnica do repositório **Crawl4AI**.  
O objetivo é utilizar o modelo **Mistral-7B-Instruct-v0.3** para identificar:

- Modelo de Branching  
- Estratégia de Release  
- Diretrizes de governança  

Tudo isso por meio da **extração automatizada de evidências reais** diretamente do repositório analisado.

---

## Infraestrutura e Otimização

Para viabilizar a execução de um modelo de grande porte em hardware gratuito, foram aplicadas as seguintes técnicas de otimização no ambiente **Kaggle**:

- **GPU NVIDIA Tesla T4**  
  Acelerador de hardware utilizado para suportar a carga computacional do modelo.

- **Quantização 4-bit (BitsAndBytes)**  
  Comprime os pesos do modelo, permitindo que os 7 bilhões de parâmetros do Mistral ocupem menos VRAM, sem perda significativa de precisão.

- **Gerenciamento de Memória (Accelerate)**  
  Utilização de `device_map="auto"` para garantir que o modelo seja distribuído corretamente na memória disponível.

---

## Execução das Células

### Célula 1: Instalação de Dependências

Prepara o ambiente com as bibliotecas necessárias para carregar modelos quantizados e gerenciar a inferência via PyTorch.

    !pip install -q -U transformers accelerate bitsandbytes

---

### Célula 2: Clonagem do Repositório

Baixa silenciosamente o código-fonte do **Crawl4AI**, permitindo a mineração de metadados.

    !git clone https://github.com/unclecode/crawl4ai.git

---

### Célula 3: Extração Automatizada de Evidências

Etapa central de **curadoria técnica**.  
O script utiliza o módulo `subprocess` para extrair dados brutos que servem como **provas reais** para a análise da IA:

- **Histórico de Tags**  
  Coleta versões e datas de criação.

- **Mapeamento de Branches**  
  Identifica a existência de branches como `develop`, `feature` e `master`.

- **Workflows CI/CD**  
  Lista arquivos YAML relacionados à automação.

- **Contexto do Changelog**  
  Captura registros recentes de mudanças no projeto.

---

### Célula 4: Carregamento do Modelo Mistral-7B

Configura a quantização em **4-bit** e inicializa o modelo **Mistral-7B-Instruct-v0.3**.  
O uso de instruções específicas do Mistral proporciona uma análise mais lógica e menos prolixa do que modelos menores.

---

### Célula 5: Engenharia de Prompt

Define o papel da IA como um **Analista de Engenharia de Software Sênior**.  
O prompt:

- Injeta os dados extraídos na Célula 3  
- Solicita um veredito técnico estruturado  
- Garante a saída em português

---

### Célula 6: Inferência e Relatório Final

O modelo processa todo o contexto acumulado e gera o veredito técnico final.  
A saída passa por um tratamento para exibir apenas a análise limpa, facilitando a leitura do relatório de auditoria.

---

## Como Executar

1. **Acelerador**  
   No menu lateral do Kaggle (Settings), configure o Accelerator para **GPU T4**.

2. **Internet**  
   Certifique-se de que a opção **Internet: On** está ativada (necessária para baixar o modelo do Hugging Face).

3. **Execução**  
   Clique em **Run All** (Executar tudo).  
   O veredito aparecerá na saída da última célula.

---

## O que Esperar do Veredito

O relatório final gerado pelo notebook apresentará:

- **Branching Model**  
  Classificação fundamentada (ex.: GitFlow), baseada na estrutura de branches detectada.

- **Estratégia de Release**  
  Identificação do modelo de entrega (ex.: Release Train ou Rapid Release).

- **Auditoria de Governança**  
  Avaliação das regras de commit, práticas de teste e integrações (como Docker) observadas nos metadados do repositório.
