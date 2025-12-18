# 📘 Tutorial — Análise de Branches e Estratégia de Releases com LLM  
## Modelo: Qwen/Qwen2.5-7B-Instruct (Google Colab Free)

Este tutorial documenta **passo a passo** a execução de um notebook no **Google Colab** utilizando o modelo de linguagem **Qwen/Qwen2.5-7B-Instruct**, com o objetivo de:

- Identificar o **Modelo de Fluxo de Trabalho (Branching Model)**
- Analisar a **Estratégia de Releases (ritmo de entrega)**

A análise é baseada **exclusivamente em evidências reais** coletadas de um repositório Git.

---

## 🧠 Modelo Utilizado

- **Qwen/Qwen2.5-7B-Instruct**
- 7 bilhões de parâmetros
- Quantização 4-bit (bitsandbytes)
- Execução em GPU T4 (Google Colab Free)

---

## ⚠️ Pré-requisitos obrigatórios

- Google Colab com **GPU ativada**
- Navegador web
- (Opcional) Token do Hugging Face

---

## ▶️ Passo 1 — Criar notebook no Google Colab

Acesse:  
https://colab.research.google.com

Ative GPU em:

```
Ambiente de execução → Alterar tipo → GPU
```

Reinicie o runtime.

---

# 📓 CÉLULAS DO NOTEBOOK (DOCUMENTADAS)

## 🔴 CÉLULA 0 — Verificação de GPU (bloqueante)

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

## 🔐 CÉLULA 1 — Login no Hugging Face (com fallback)

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

## ⚙️ CÉLULA 2 — Setup do ambiente

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

print("Ambiente configurado")
```

---

## 🧠 CÉLULA 3 — Carregamento do modelo Qwen 2.5 (7B)

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

print("Modelo Qwen2.5-7B carregado")
```

---

## 🧩 CÉLULA 4 — Função de inferência segura

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

## 🧪 CÉLULA 5 — Teste do modelo

```python
print(ask_qwen("Explique em duas linhas o que é GitFlow."))
```

---

## 📦 CÉLULA 6 — Coleta de evidências do Git

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

Este tutorial demonstra como um **LLM pode apoiar análises de governança de software**, desde que:

- receba **dados reais**
- tenha **prompts restritivos**
- seja executado com **controle de recursos**

O modelo **Qwen2.5-7B-Instruct** mostrou-se adequado para análises estruturais de projetos open source no contexto acadêmico.
