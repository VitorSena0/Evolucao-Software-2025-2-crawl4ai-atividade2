# Tutorial: Llama-3.1-8B-Instruct
### 1 - Pré-requisitos
- Uma conta no Hugging Face.
    
- Um Access Token do Hugging Face com as permissões da seção Inference do seu perfil do Hugging Face: Configurações → Tokens de acesso → Criar novo token.
    
- Uma conta no Google Colabe.
    
- Configurações de ambiente utilizadas:
    
	- Acelerador de hardware: CPU ou GPUs T4
	    
	- Tipo de ambiente: Python 3
    
### 2 - Como Executar (Passo a Passo)

1. Crie o Notebook: No Google Colab, faça o upload do arquivo .ipynb ou copie o código para novas células.
    
2. Configure a variável de ambiente HF_TOKEN: utilize o token de acesso de sua conta do Hugging Face.
    
3. Execute Sequencialmente: Rode as células na ordem (1 a 6). Não pule etapas, pois as variáveis de dados (logs e arquivos) são interdependentes.
    
4. Aguarde a Inferência.
    
5. Analise a Saída: Após a conclusão, abra o ícone de pasta na lateral esquerda e localize o arquivo resposta-llama-3_1-8B-Instruct.md para ler o veredito final da auditoria.
    

### 3 - Estrutura do Notebook

- Célula 1–2 (Coleta de Dados): Clona o repositório Crawl4AI e extrai o Histórico de Merges, gerando a "impressão digital" do fluxo de trabalho das branches.
```ipynb
!git clone https://github.com/unclecode/crawl4ai.git --quiet
```

```ipynb
%cd /content/crawl4ai

!git log --graph --oneline --all --decorate --merges > /content/historico_merges.txt
```
    
- Células 3 (Ingestão de Artefatos): Carrega os arquivos CHANGELOG.md e release.yml para variáveis, servindo como base de evidências para a análise de cadência de lançamentos.  
```ipynb
changelog = ""

releases = ""

merges = ""

with open("/content/crawl4ai/CHANGELOG.md", "r") as f:

  changelog = f.read()

with open("/content/crawl4ai/.github/workflows/release.yml", "r") as f:

  releases = f.read()

with open("/content/historico_merges.txt", "r") as f:

  merges = f.read()
```

- Célula 4 (Engenharia de Prompt): Configura o System Prompt com definições de Gitflow, GitHub Flow e Trunk-Based, estruturando o raciocínio lógico da IA.
```ipynb
system_msg = f"""Você é um Especialista em Engenharia de Software com foco em Gerenciamento de Configuração. Sua tarefa é analisar o repositório "Crawl4AI" para identificar sua Estratégia de Release e seu Fluxo de Trabalho (Branching Model).

  

PRINCIPAIS ESTRATÉGIAS DE WORKFLOW PARA PROCURAR:

- Gitflow: Presença de branches persistentes 'main' E 'develop'. Merges de features vão para 'develop'. Existem branches de 'release/v*'.

- GitHub Flow: Apenas a branch 'main' é persistente. Features branches são curtas e mergeadas direto na 'main'.

- Trunk-based: Commits frequentes e diretos na 'main' (ou via PRs curtíssimos). O histórico é quase uma linha única e reta.

  

PRINCIPAIS ESTRATÉGIAS DE RELEASE PARA PROCURAR:

- Rapid Release: Alta frequência de versões (tags v* constantes) baseadas em novas funcionalidades prontas.

- Release Train: Versões em datas fixas e previsíveis.

- LTS + Current: Coexistência de versões estáveis de longo suporte com versões de ponta.

  

VOCÊ RECEBERÁ DADOS DOS SEGUINTES ARQUIVOS:

- CHANGELOG.md: Registro histórico de versões e datas.

- release.yml: Configuração de automação do GitHub Actions.

- Histórico de Merges: O log visual (graph) da árvore de commits.

  

METODOLOGIA DE ANÁLISE:

1. Examine o 'release.yml': Tente identificar o que dispara o deploy.

2. Examine o 'CHANGELOG.md': Tente identificar o intervalo médio entre versões.

3. Examine o 'Histórico de Merges': Procure pela existência da branch 'develop' e branches com prefixo 'release/v*', 'feature/*' ou 'fix/* e verificar a persistência dessas branches'.

  

CONCLUSÃO:

Apresente sua conclusão baseada na predominância das evidências. Se houver elementos de mais de um modelo (ex: um fluxo Gitflow simplificado com ritmo de Rapid Release), descreva essa governança híbrida."""

user_msg = f"""

Aqui estão os arquivos a serem analisados:

CHANGELOG.md:

{"\n".join(changelog.splitlines()[-262:])}

  

release.yml:

{releases}

  

Histórico de Merges:

{merges}

"""
```
    
- Célula 5 (Variável de ambiente): Importa bibliotecas para inicializar a variável de ambiente HF_TOKEN.
```ipynb
from google.colab import userdata

import os

os.environ['HF_TOKEN'] = userdata.get('HF_TOKEN')
```

Célula 6 (Carga do Modelo e Auditoria Final): Inicializa o Llama-3.1-8B-Instruct por meio de inferência remota provida pelo Hugging Face e realiza o cruzamento das evidências (logs + YAML) com as regras de engenharia, gerando um relatório técnico em Markdown sobre a governança do projeto.**
```ipynb
from openai import OpenAI

client = OpenAI(
    base_url="https://router.huggingface.co/v1",
    api_key=os.environ["HF_TOKEN"],
)

completion = client.chat.completions.create(
    model="meta-llama/Llama-3.1-8B-Instruct",
    messages=[
        {
            "role": "system",
            "content": system_msg
        },
        {
            "role": "user",
            "content": user_msg
        }
    ],
    max_tokens=1500,
    temperature=0.2,
    top_p=0.9
)

with open("/content/resposta-llama-3_1-8B-Instruct.md", "w") as f:
  f.write(completion.choices[0].message.content)
```
