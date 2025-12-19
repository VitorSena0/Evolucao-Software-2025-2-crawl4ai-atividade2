# Identificação de Padrões de Workflow Git e Estratégias de Releases via LLMs - Atividade 2 - Crawl4AI
## Autores
- Carlos Daniel Lima de Gois
- Felipe Osni Santos Moura
- João Pedro Cardoso Arruda
- Nicolas Matheus Ferreira de Jesus
- Samuel Bastos Borges Pinho
- Vinícius Vasconi Villas Boas Micska
- Vitor Leonardo Sena de Lima
- David Silva Santana

## Sumário
- [Visão Geral](#visão-geral)
- [Artefatos incluídos](#artefatos-incluídos)
- [Modelos utilizados](#modelos-utilizados)
- [Metodologia e scripts principais](#metodologia-e-scripts-principais)
- [Tutoriais](#tutoriais)
- [Principais achados](#principais-achados)
- [Limitações e cuidados](#limitações-e-cuidados)

## Visão Geral
Este projeto realiza uma auditoria de engenharia de software no repositório open-source [Crawl4AI](https://github.com/unclecode/crawl4ai), utilizando Modelos de Linguagem de Grande Escala (LLMs) para identificar padrões arquiteturais, especificamente o modelo de fluxo de trabalho (Branching Model) e a estratégia de releases. O trabalho compara a eficácia de diferentes modelos e técnicas de prompt engineering contra uma análise manual de referência.

**Contexto**: Disciplina de Engenharia de Software II (UFS), 2025.

## Artefatos incluídos
- **Documentação**: Relatórios detalhados e tutoriais na pasta `docs/`.
- **Notebooks**: Scripts executáveis (Jupyter Notebooks) para cada modelo avaliado na pasta `notebooks/`.
- **Evidências**: Dumps de tags e logs utilizados na análise.
- **Relatório Final**: Documento consolidado na pasta `tutorial/`.

## Modelos utilizados
1. **Mistral-7B-Instruct-v0.3** (Kaggle, 7B parâmetros)
2. **Qwen2.5-Coder-0.5B-Instruct** (Colab, 0.5B parâmetros)
3. **Qwen2.5-7B-Instruct** (Colab, 7B parâmetros)
4. **Phi-3.5-mini-instruct** (Colab, 3.8B parâmetros)
5. **CodeLlama-7B-Instruct** (Kaggle, 7B parâmetros)

## Metodologia e scripts principais

### Coleta de evidências
A coleta de dados foi realizada através de scripts Python e comandos Git integrados aos notebooks. Foram extraídos:
- Lista de branches locais e remotas (`git branch -a`).
- Histórico de tags e datas de publicação (`git tag`, API do GitHub).
- Logs de commits e grafos de merge (`git log --graph`).

### Extração do esqueleto do código
Para fornecer contexto aos LLMs sem estourar a janela de contexto, foram selecionados arquivos estratégicos:
- `CHANGELOG.md`: Para entender a frequência e tipo de atualizações.
- `.github/workflows/*.yml`: Para identificar automações de CI/CD e release.
- `CONTRIBUTING.md`: Para buscar regras explícitas de contribuição.

### Pipeline de análise
1. **Setup**: Instalação de bibliotecas (`transformers`, `bitsandbytes`, `accelerate`).
2. **Ingestão**: Carregamento dos dados coletados no prompt do modelo.
3. **Inferência**: Execução do modelo com prompts estruturados (System/User roles) e guardrails para evitar alucinações.
4. **Extração**: Obtenção da resposta em formato estruturado (JSON) ou texto analítico.

### Pós-processamento e consolidação
Os resultados de cada modelo foram comparados com a análise manual (Ground Truth). Divergências foram analisadas para entender as limitações de cada modelo (ex: alucinação de LTS no CodeLlama, precisão factual no Qwen 0.5B).

## Tutoriais

### Tutorial 1: Mistral-7B-Instruct-v0.3
**Responsável**: Samuel Pinho

#### Infraestrutura utilizada
- **Plataforma**: Kaggle
- **GPU**: NVIDIA Tesla T4 (16GB VRAM)
- **Quantização**: 4-bit (via `bitsandbytes`)
- **Memória RAM**: ~13GB (Padrão Kaggle)
- **Processador**: vCPU padrão Kaggle

#### Reprodutibilidade e execução
O notebook está configurado para rodar sequencialmente. As dependências são instaladas na primeira célula.

##### Como executar
1. Acesse o notebook: `notebooks/crawl4ai-mistral-7b-instruct-v0-3.ipynb`.
2. Importe para o Kaggle.
3. Configure o acelerador para **GPU T4 x2**.
4. Ative a Internet (necessária para download de modelos).
5. Execute as células na ordem (1 a 6).
6. Verifique a saída da última célula para o veredito.

**Guia detalhado**: [docs/tutorial_atividade2_Mistral-7B-Instruct-v0.3.md](docs/tutorial_atividade2_Mistral-7B-Instruct-v0.3.md)

### Tutorial 2: Qwen2.5-Coder-0.5B-Instruct
**Responsável**: Carlos Daniel

#### Infraestrutura utilizada
- **Plataforma**: Google Colab
- **GPU**: Opcional (roda em CPU, mas GPU T4 recomendada para rapidez)
- **Memória RAM**: Baixo consumo, compatível com tier gratuito
- **Processador**: vCPU padrão Colab

#### Reprodutibilidade e execução
Focado em eficiência e precisão factual com modelo pequeno. Utiliza guardrails para reduzir alucinações.

##### Como executar
1. Abra um novo notebook no Colab ou siga o guia em `docs/tutorial_atividade2_qwen05b.md`.
2. Instale as dependências (`transformers`, `accelerate`).
3. Execute a coleta de branches e releases via Git e API GitHub.
4. Execute a célula de inferência que solicita saída em JSON.
5. Analise a conclusão na última célula.

**Guia detalhado**: [docs/tutorial_atividade2_qwen05b.md](docs/tutorial_atividade2_qwen05b.md)

### Tutorial 3: Phi-3.5-mini-instruct
**Responsável**: João Pedro

#### Infraestrutura utilizada
- **Plataforma**: Google Colab
- **GPU**: NVIDIA Tesla T4 (Obrigatório)
- **Quantização**: 4-bit (NF4) com Double Quantization
- **Otimização**: SDPA (Scaled Dot Product Attention)
- **Memória RAM**: ~2.5GB com quantização

#### Reprodutibilidade e execução
Utiliza otimização SDPA para máxima eficiência. Coleta logs de merge, CHANGELOG e workflow CI/CD.

##### Como executar
1. Abra o notebook correspondente no Colab.
2. Certifique-se de estar usando GPU T4 (Ambiente de Execução → Alterar tipo).
3. Ative a Internet.
4. Execute as células de 1 a 9 sequencialmente.
5. O resultado será salvo em `resposta.md` na pasta de arquivos.

**Guia detalhado**: [docs/tutorial_phi_3_5_mini_instruct.md](docs/tutorial_phi_3_5_mini_instruct.md)

### Tutorial 4: Llama-3.1-8B-Instruct
**Responsável**: Felipe Osni

#### Infraestrutura utilizada
- **Plataforma**: Google Colab
- **CPU**: 1 núcleo
- **Memória RAM**: 12.7GB
- **Disco**: 107.7GB

##### Como executar
1. Abra o notebook correspondente no Colab.
2. Ative a Internet.
3. Configure a variável de ambiente para inferência remota HF_TOKEN.
4. Execute as células de 1 a 6 sequencialmente.
5. O resultado será salvo em `resposta-llama-3_1-8B-Instruct.md` na pasta de arquivos.

**Guia detalhado**: [docs/tutorial_llama_3_1_8B_Instruct.md](docs/tutorial_llama_3_1_8B_Instruct.md)

### Tutorial 5: CodeLlama-7B-Instruct
**Responsável**: Vitor Leonardo

#### Infraestrutura utilizada
- **Plataforma**: Kaggle
- **GPU**: NVIDIA Tesla T4
- **Quantização**: 4-bit
- **Análise**: Estatística de tags e branches com coleta abrangente

#### Como executar
1. Importe o notebook no Kaggle.
2. Configure o acelerador para GPU T4.
3. Ative a Internet.
4. Execute as células de setup, coleta e análise estatística.
5. Observe a classificação final (neste caso, houve divergência apontando Trunk-Based).

**Guia detalhado**: [docs/codellama7B_tutorial.md](docs/codellama7B_tutorial.md)

### Tutorial 6: Qwen2.5-7B-Instruct
**Responsável**: David Silva

#### Infraestrutura utilizada
- **Plataforma**: Google Colab
- **GPU**: NVIDIA Tesla T4 (Obrigatório)
- **Quantização**: 4-bit (BitsAndBytes)
- **Memória**: ~13–16GB com quantização
- **Processador**: vCPU padrão Colab

#### Reprodutibilidade
Requer verificação de GPU ativa antes do carregamento do modelo para evitar OOM (Out of Memory).

#### Como executar
1. Abra um novo notebook no Colab.
2. Copie o conteúdo do guia [docs/tutorial_qwen2_5_7B_colab_.md](docs/tutorial_qwen2_5_7B_colab_.md).
3. Ative GPU T4 (Ambiente de Execução → Alterar tipo).
4. Verifique se a GPU está ativa na Célula 0.
5. Execute sequencialmente até a Célula 8.
6. Consulte as últimas células para workflow e releases.

#### Etapas da análise
1. **Verificação de ambiente**: Checar disponibilidade de CUDA.
2. **Autenticação Hugging Face**: Login opcional para evitar rate limits.
3. **Setup**: Instalação de bibliotecas e configuração de variáveis de ambiente.
4. **Carregamento do modelo**: Inicialização com quantização 4-bit.
5. **Teste de funcionamento**: Validação rápida do modelo antes de análise.
6. **Coleta de dados**: Extração de branches, tags e distribuição de prefixos.
7. **Análise de Workflow**: Classificação entre GitFlow, GitHub Flow, Trunk-Based Development.
8. **Análise de Estratégia de Releases**: Identificação entre Rapid Release, Release Train, LTS + Current, Ad-hoc.

## Principais achados
- **Workflow**: Consenso de que o projeto utiliza **GitFlow** (ou variação GitFlow-like), evidenciado pelas branches `main`, `develop` e prefixos `feature/`, `fix/`, `bugfix/`, `release/`.
- **Estratégia de Releases**: Predominância de **Rapid Releases**, caracterizada por lançamentos frequentes e intervalos curtos (muitas versões v0.7.x em 2025, algumas com diferença de 24h).
- **Desempenho dos Modelos**: 
  - Mistral-7B: Excelente qualidade textual, mas necessitou validação manual sobre cadência.
  - Qwen 0.5B: Precisão factual surpreendente com bons guardrails.
  - Qwen 7B: Visão equilibrada incluindo Ad-hoc como nuance.
  - Phi-3.5: Identificou padrões, mas com texto prolixo em português.
  - CodeLlama: Divergência significativa (apontou Trunk-Based e LTS Hybrid).

## Limitações e cuidados
- **Alucinações**: Modelos podem inferir padrões inexistentes (ex: LTS) se não forem restringidos por evidências concretas. Aplicar guardrails é essencial.
- **Contexto limitado**: O limite de tokens exige seleção cuidadosa do que é enviado ao modelo. Não enviar logs inteiros evita repetição.
- **Hardware requerido**: A execução dos modelos de 7B parâmetros requer GPU com pelo menos 12GB de VRAM e uso de quantização (4-bit) para rodar em ambientes gratuitos como Colab e Kaggle.
- **Validade temporal**: Mudanças futuras no repositório alvo podem alterar conclusões. Repetir a coleta antes de gerar novos relatórios.
- **Mudanças na API GitHub**: Rate limits e descontinuações podem afetar a coleta de releases via API.
