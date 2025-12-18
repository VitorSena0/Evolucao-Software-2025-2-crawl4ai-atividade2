# Tutorial Técnico: Auditoria de Governança com Phi-3.5

Este guia detalha o funcionamento do notebook desenvolvido para a análise do repositório **Crawl4AI**. O objetivo é utilizar Inteligência Artificial para identificar a Estratégia de Release e o Modelo de Branching através de evidências em arquivos de configuração e logs de versionamento.

---

## Infraestrutura e Otimização
Para viabilizar a execução do modelo **Phi-3.5-mini** em hardware gratuito (GPU Nvidia T4), aplicamos as seguintes técnicas:
* **Quantização 4-bit (NF4):** Comprime os pesos do modelo, reduzindo o uso de VRAM de ~8GB para ~2.5GB.
* **SDPA (Scaled Dot Product Attention):** Otimização do PyTorch que acelera a inferência e economiza memória ao processar textos longos.
* **Double Quantization:** Técnica que quantiza as constantes de quantização para economizar memória adicional.

---

## Execução das Células

### Célula 1: Instalação de Dependências
Prepara o ambiente com as ferramentas necessárias para carregar modelos comprimidos e gerenciar o hardware da GPU.
```python
!pip install -q -U transformers bitsandbytes accelerate
```
### Célula 2: Clonagem do Repositório
Baixa o código-fonte completo do projeto que será auditado.
```python
!git clone https://github.com/unclecode/crawl4ai.git
```
### Célula 3: Preparação do Diretório
Navega para dentro da pasta do repositório e confirma a localização atual para evitar erros de leitura de arquivos.
```python
# Entra na pasta do repositório
%cd /content/crawl4ai

# Verifica se agora você está no lugar certo (deve mostrar o caminho terminando em /crawl4ai)
!pwd
```
### Célula 4: Extração do Histórico de Merges
Gera um log visual filtrando apenas os eventos de merge e coloca em um arquivo .txt. Este arquivo é a "prova real" do fluxo de trabalho das branches.
```python
# Salva o histórico de merges das branches
!git log --graph --oneline --all --decorate --merges > /content/historico_merges.txt
```
### Célula 5: Importação de Bibliotecas
```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer, pipeline, BitsAndBytesConfig
```
### Célula 6: Leitura de arquivos
Lê os três arquivos fundamentais para a análise e armazena o conteúdo em variáveis.
```python
# lê os principais arquivos com informações relacionadas às releases e as branches
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
### Célula 7: Criação do Prompt
Define as regras teóricas no system_msg para guiar a IA e injeta os dados reais no user_msg.

### Célula 8: Carregamento do Modelo Phi-3.5
Configura a quantização 4-bit e carrega o modelo utilizando a implementação de atenção SDPA para máxima eficiência.
### Célula 9: Inferência e Relatório Final
Converte o prompt em tensores e solicita que a IA gere a análise. O veredito é salvo em um arquivo Markdown.

### 🚀 Como Executar
1. Certifique-se de que o Ambiente de Execução está configurado para T4 GPU.

2. Execute as células na ordem ou vá em Ambiente de Execução > Executar tudo.

3. O arquivo resposta.md aparecerá na pasta /content/ com a auditoria completa.