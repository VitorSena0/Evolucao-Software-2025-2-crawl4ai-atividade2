## Célula 1 — Instalar libs e carregar o Qwen 0.5B
```python
!pip -q install -U transformers accelerate

from transformers import AutoTokenizer, AutoModelForCausalLM, pipeline

MODEL_ID = "Qwen/Qwen2.5-Coder-0.5B-Instruct"

tok = AutoTokenizer.from_pretrained(MODEL_ID)
model = AutoModelForCausalLM.from_pretrained(MODEL_ID, device_map="auto")
gen = pipeline("text-generation", model=model, tokenizer=tok)

print("OK:", MODEL_ID)
```

---

## Célula 2 — Clonar o repo oficial e coletar branches/tags/merges (sinais)
```python
import os, shutil, subprocess

REPO_URL = "https://github.com/unclecode/crawl4ai.git"
WORKDIR = "/content/repo"

def sh(cmd):
    return subprocess.check_output(cmd, shell=True, text=True, stderr=subprocess.STDOUT).strip()

# limpar e clonar
if os.path.exists(WORKDIR):
    shutil.rmtree(WORKDIR)
print(sh(f"git clone --quiet {REPO_URL} {WORKDIR}"))
os.chdir(WORKDIR)

# branches (amostra)
branches_head = sh("git branch -a | head -n 80")

# tags v* (amostra)
tags_v = sh("git for-each-ref --sort=-creatordate --format='%(refname:short) | %(creatordate:iso8601)' refs/tags | grep -E '^v' | head -n 40")

# merges (sinais) 
merges_signal = sh("git log --oneline --decorate --all --merges -n 250 | grep -Ei 'main|develop|next|release/|hotfix' | sort -u | head -n 60")

print("=== BRANCHES (amostra) ===\n", branches_head)
print("\n=== TAGS v* (amostra) ===\n", tags_v)
print("\n=== MERGES (sinais) ===\n", merges_signal)
```

---

## Célula 3 — Pegar Releases via API do GitHub (datas oficiais)
```python
import requests

url = "https://api.github.com/repos/unclecode/crawl4ai/releases"
resp = requests.get(url, timeout=30)
data = resp.json() if resp.status_code == 200 else []

print("HTTP:", resp.status_code)
print("Qtd releases:", len(data))

releases_txt = "(Sem releases)" if not data else "\n".join(
    [f"{r.get('tag_name')} | {r.get('published_at')} | {r.get('name')}" for r in data[:12]]
)

print("\n=== RELEASES (top 12) ===")
print(releases_txt)
```

---

## Célula 4 — Montar pacote de evidências
```python
evidencias = f"""
REPO: {REPO_URL}

[BRANCHES]
{branches_head}

[TAGS v*]
{tags_v}

[RELEASES (API)]
{releases_txt}

[MERGES (sinais)]
{merges_signal}
""".strip()

print(evidencias[:3500])
```

---

## Célula 5 — Qwen 0.5B (anti-repetição) — gerar JSON final
```python
# evidências mínimas (sem merges para evitar repetição do modelo)
evidencias_min = f"""
REPO: https://github.com/unclecode/crawl4ai

BRANCHES (trecho):
{branches_head}

RELEASES (top 12):
{releases_txt}
""".strip()

prompt = f"""
RETORNE APENAS UM JSON VÁLIDO. NÃO ESCREVA MAIS NADA.

Use exatamente este formato:
{{
  "release_strategy": "Rapid Releases|Release Train|LTS + Current|insufficient_evidence",
  "branching_model": "Gitflow|GitHub Flow|insufficient_evidence",
  "evidences": ["...", "...", "...", "...", "...", "..."],
  "confidence": 0
}}

Regras:
- Não invente branches/tags/datas.
- Evidences: copie trechos curtos EXATOS das evidências acima.
- Se faltar prova, use "insufficient_evidence".
""".strip()

out = gen(
    prompt + "\n\nEVIDÊNCIAS:\n" + evidencias_min,
    max_new_tokens=180,
    do_sample=False,
    repetition_penalty=1.25
)[0]["generated_text"]

print(out.split("EVIDÊNCIAS:")[-1].strip())
```
