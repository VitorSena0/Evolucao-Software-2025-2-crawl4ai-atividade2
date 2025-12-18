# 📘 Tutorial Completo — Análise de Branches e Estratégia de Releases com LLM  
## Modelo: Qwen/Qwen2.5-7B-Instruct (Google Colab Free)

Este documento descreve **detalhadamente cada célula do notebook**, explicando **o que ela faz, por que existe e como contribui para a análise**.  
O objetivo é permitir que **qualquer pessoa replique o experimento** e compreenda as decisões técnicas adotadas.

---

## 🎯 Objetivo do Tutorial

Utilizar o modelo de linguagem **Qwen/Qwen2.5-7B-Instruct** para apoiar a identificação de:

- **Modelo de Fluxo de Trabalho (Branching Model)**
- **Estratégia de Releases (ritmo de entrega)**

A análise é baseada **exclusivamente em dados reais extraídos de um repositório Git**, garantindo rigor metodológico.

---

## 🧠 Modelo de Linguagem Utilizado

- **Nome:** Qwen/Qwen2.5-7B-Instruct  
- **Plataforma:** Hugging Face  
- **Parâmetros:** 7B  
- **Execução:** Google Colab Free (GPU T4)  
- **Otimização:** Quantização 4-bit com bitsandbytes  

---

## ⚠️ Pré-requisitos

- Conta Google
- Google Colab com **GPU ativada**
- Navegador web moderno
- (Opcional) Token do Hugging Face

---

## ▶️ Configuração Inicial no Google Colab

1. Acesse: https://colab.research.google.com  
2. Crie um novo notebook  
3. Ative a GPU em:

```
Ambiente de execução → Alterar tipo de ambiente de execução → GPU
```

4. Reinicie o ambiente

---

# 📓 CÉLULAS DO NOTEBOOK — COM EXPLICAÇÃO

---

## 🔴 CÉLULA 0 — Verificação de GPU (Obrigatória)

### 📌 Objetivo
Garantir que o notebook **não seja executado em CPU**, evitando erros posteriores.

### 🧠 Por que isso é necessário?
O modelo Qwen 7B **não funciona no Colab Free sem GPU**.  
Essa célula interrompe a execução caso a GPU não esteja ativa.

```python
import torch

print("Torch:", torch.__version__)
print("CUDA disponível:", torch.cuda.is_available())

if not torch.cuda.is_available():
    raise RuntimeError(
        "GPU NÃO ATIVA. Ative em Ambiente de execução → GPU e reinicie."
    )

print("GPU detectada:", torch.cuda.get_device_name(0))
```

---

## 🔐 CÉLULA 1 — Autenticação no Hugging Face

### 📌 Objetivo
Autenticar o usuário no Hugging Face **caso um token esteja disponível**.

### 🧠 Por que usar token?
- Evita rate limit
- Aumenta a estabilidade do download
- Não é obrigatório (modelo público)

```python
import os
from huggingface_hub import login

if "HUGGINGFACE_TOKEN" in os.environ:
    login(token=os.environ["HUGGINGFACE_TOKEN"])
    print("Hugging Face autenticado com token")
else:
    print("Token não encontrado — acesso público será utilizado")
```

---

## ⚙️ CÉLULA 2 — Setup do Ambiente

### 📌 Objetivo
Instalar apenas as dependências necessárias, **sem reinstalar o PyTorch**, evitando conflitos com CUDA.

### 🧠 Decisões importantes
- O PyTorch do Colab já vem com CUDA
- bitsandbytes é reinstalado para garantir compatibilidade

```python
import os, sys, subprocess

os.environ["CUDA_VISIBLE_DEVICES"] = "0"
os.environ["TOKENIZERS_PARALLELISM"] = "false"

def pip_install(args):
    subprocess.check_call([sys.executable, "-m", "pip"] + args.split())

def pip_uninstall(pkg):
    try:
        subprocess.check_call([sys.executable, "-m", "pip", "uninstall", "-y", pkg])
    except:
        pass

pip_install("install -q -U transformers accelerate huggingface_hub")
pip_uninstall("bitsandbytes")
pip_install("install -q -U bitsandbytes")

print("Ambiente configurado com sucesso")
```

---

## 🧠 CÉLULA 3 — Carregamento do Modelo (Qwen 7B)

### 📌 Objetivo
Carregar o modelo **Qwen2.5-7B-Instruct** de forma segura no Colab Free.

### 🧠 Estratégias usadas
- Quantização 4-bit
- Forçar uso exclusivo da GPU
- Evitar offload automático para RAM

```python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig

MODEL_ID = "Qwen/Qwen2.5-7B-Instruct"

tokenizer = AutoTokenizer.from_pretrained(
    MODEL_ID,
    use_fast=True,
    trust_remote_code=True
)

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.float16,
    bnb_4bit_use_double_quant=True,
)

model = AutoModelForCausalLM.from_pretrained(
    MODEL_ID,
    quantization_config=bnb_config,
    device_map={"": 0},
    torch_dtype=torch.float16,
    low_cpu_mem_usage=True,
    trust_remote_code=True,
)

model.eval()

print("Modelo Qwen2.5-7B carregado com sucesso")
```

---

## 🧩 CÉLULA 4 — Função de Inferência Controlada

### 📌 Objetivo
Criar uma função que:
- Limite o número de tokens
- Evite estouro de memória
- Garanta respostas consistentes

```python
def ask_qwen(
    prompt,
    max_new_tokens=700,
    temperature=0.2,
    top_p=0.95,
    max_prompt_tokens=6000,
):
    messages = [
        {"role": "system", "content": "Você é um especialista em Engenharia de Software."},
        {"role": "user", "content": prompt},
    ]

    input_ids = tokenizer.apply_chat_template(
        messages,
        tokenize=True,
        truncation=True,
        max_length=max_prompt_tokens,
        return_tensors="pt"
    ).to(model.device)

    with torch.inference_mode():
        output = model.generate(
            input_ids,
            max_new_tokens=max_new_tokens,
            do_sample=True,
            temperature=temperature,
            top_p=top_p,
            pad_token_id=tokenizer.eos_token_id,
        )

    decoded = tokenizer.decode(output[0], skip_special_tokens=True)
    return decoded.split(messages[-1]["content"])[-1].strip()
```

---

## 🧪 CÉLULA 5 — Teste do Modelo

### 📌 Objetivo
Validar se o modelo está respondendo corretamente antes de prosseguir.

```python
print(ask_qwen("Explique em duas linhas o que é GitFlow."))
```

---

## 📦 CÉLULA 6 — Coleta de Evidências do Git

### 📌 Objetivo
Extrair dados reais do repositório:
- branches
- tags
- distribuição por prefixo

Esses dados são a **base factual** da análise.

```python
import subprocess, os
from collections import Counter

def git(cmd, cwd):
    return subprocess.check_output(cmd, cwd=cwd, shell=True).decode().strip()

repo_url = "https://github.com/unclecode/crawl4ai.git"
repo_dir = "/content/repo"

if not os.path.exists(repo_dir):
    subprocess.check_call(["git", "clone", repo_url, repo_dir])

branches = git("git branch -r | grep -v HEAD", repo_dir).splitlines()
branches = [b.replace("origin/", "").strip() for b in branches]

tags = git("git tag --list", repo_dir).splitlines()

bucket = Counter(b.split('/')[0] if '/' in b else 'root' for b in branches)

print("Branches:", len(branches))
print("Tags:", len(tags))
print("Distribuição:", bucket)
```

---

## 🌿 CÉLULA 7 — Análise do Branching Model

### 📌 Objetivo
Solicitar ao modelo que identifique o **modelo de fluxo de trabalho**, utilizando apenas as evidências coletadas.

```python
prompt = f"""
Analise o modelo de branching com base nos dados reais.

Branches totais: {len(branches)}
Distribuição por prefixo:
{bucket}

Classifique:
- GitFlow
- GitHub Flow
- Trunk-Based Development

Justifique com base apenas nas evidências.
"""

print(ask_qwen(prompt))
```

---

## ⏱️ CÉLULA 8 — Análise da Estratégia de Releases

### 📌 Objetivo
Identificar o ritmo de entrega do projeto a partir das tags.

```python
prompt = f"""
Analise a estratégia de releases com base nas tags abaixo.

Total de tags: {len(tags)}
Amostra de tags:
{tags[-20:]}

Classifique:
- Rapid Release
- Release Train
- LTS + Current
- Ad-hoc
"""

print(ask_qwen(prompt))
```

---

## ✅ Considerações Finais

Este tutorial evidencia que **LLMs podem apoiar análises de governança de software**, desde que:

- sejam alimentados com **dados reais**
- utilizem **prompts restritivos**
- respeitem **limitações do ambiente de execução**

O modelo **Qwen2.5-7B-Instruct** demonstrou bom desempenho na identificação de padrões estruturais em projetos open source.

