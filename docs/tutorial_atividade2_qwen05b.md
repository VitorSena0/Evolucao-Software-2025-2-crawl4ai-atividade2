# Tutorial — Atividade 2 (Branches + Releases) usando Qwen 0.5B

**Projeto analisado (evidências):** `https://github.com/unclecode/crawl4ai`  
**Modelo desta parte:** `Qwen/Qwen2.5-Coder-0.5B-Instruct`  

> Objetivo: coletar evidências (branches e releases) e, com apoio do Qwen 0.5B, concluir **Branching Model** e **Estratégia de Releases** com justificativas baseadas em dados observáveis.

---

## 1) Pré-requisitos

- Google Colab com Internet habilitada.
- (Opcional) GPU ativada no Colab (não é obrigatório para o Qwen 0.5B, mas ajuda).

---

## 2) Como usar este tutorial (passo a passo)

1. Abra um notebook no Colab.
2. Cole as células abaixo **na ordem**.
3. Rode e salve as saídas (print das branches e das releases).
4. Use as conclusões prontas (seção **Resultados**) no relatório/vídeo.

---

## 3) Células do Colab (na ordem)

### Célula 1 — Instalar libs e carregar o Qwen 0.5B
```python
!pip -q install -U transformers accelerate

from transformers import AutoTokenizer, AutoModelForCausalLM, pipeline

MODEL_ID = "Qwen/Qwen2.5-Coder-0.5B-Instruct"

tok = AutoTokenizer.from_pretrained(MODEL_ID)
model = AutoModelForCausalLM.from_pretrained(MODEL_ID, device_map="auto")
gen = pipeline("text-generation", model=model, tokenizer=tok)

print("OK:", MODEL_ID)
```

### Célula 2 — Clonar o repo e coletar branches (fluxo de trabalho)
```python
import os, shutil, subprocess

REPO_URL = "https://github.com/unclecode/crawl4ai.git"
WORKDIR = "/content/repo"

def sh(cmd):
    return subprocess.check_output(cmd, shell=True, text=True, stderr=subprocess.STDOUT).strip()

if os.path.exists(WORKDIR):
    shutil.rmtree(WORKDIR)

print(sh(f"git clone --quiet {REPO_URL} {WORKDIR}"))
os.chdir(WORKDIR)

branches_head = sh("git branch -a | head -n 80")
print("=== BRANCHES (amostra) ===\n", branches_head)
```

**O que observar na saída:**  
- `main` (branch estável)  
- `develop` (branch de integração)  
- prefixos como `feature/*`, `fix/*`, `bug/*`, `docs/*` (branches por propósito)  
- branches por versão/linha (ex.: `0.3.5`, `0.4.0`, `0.4.2`)

### Célula 3 — Coletar releases via API do GitHub (datas oficiais)
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

**O que observar na saída:**  
- várias versões em 2025 (ex.: `v0.7.x`)  
- intervalos curtos entre algumas releases (ex.: `v0.7.5` e `v0.7.6` em dias seguidos)

### Célula 4 — (Opcional) pedir JSON para o Qwen (modo “anti-repetição”)
> Observação: modelos pequenos podem repetir texto quando recebem logs grandes. Por isso, esta célula usa **evidências curtas** (branches + releases) e força **JSON only**.

```python
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

---

## 4) Interpretação (como decidir a classificação)

### 4.1 Branching Model (Fluxo de Trabalho)
- **Gitflow / Gitflow-like**: quando existe `develop` além de `main` e há branches por propósito (`feature/*`, `fix/*`, `bug/*`, etc.).
- **GitHub Flow**: geralmente quando só existe `main` + branches curtas (sem `develop` persistente).

### 4.2 Estratégia de Releases
- **Rapid Releases**: muitas releases no ano e intervalos curtos entre versões.
- **Release Train**: releases em calendário fixo rígido (datas regulares).
- **LTS + Current**: indicação explícita de uma linha LTS mantida em paralelo com a linha current.

---

## 5) Resultados (conclusão desta parte)

### 5.1 Fluxo de Trabalho
**Conclusão:** **Gitflow-like** (marcado como Gitflow)  
**Justificativa (evidências):** existe `develop` além de `main` e há muitas branches por propósito (`feature/*`, `fix/*`, `bug/*`, `docs/*`), além de branches por versão/linha (ex.: `0.3.5`, `0.4.0`, `0.4.2`).

### 5.2 Estratégia de Releases
**Conclusão:** **Rapid Releases**  
**Justificativa (evidências):** várias releases publicadas em 2025 no padrão `v0.7.x`, com intervalos curtos entre algumas versões (ex.: `v0.7.5` em 2025-10-21 e `v0.7.6` em 2025-10-22).

---

## 6) Problemas comuns (e soluções rápidas)

- **O modelo repete texto (ex.: “Merge ... into next” várias vezes)**  
  Use a célula “anti-repetição” (Célula 4) e evite enviar logs grandes de `git log`.

- **Erro ao importar `transformers` após instalar pacotes**  
  Reinicie o runtime do Colab e rode novamente a Célula 1.

- **API do GitHub retorna erro / limite**  
  Tente novamente alguns minutos depois ou reduza a quantidade de consultas.

---


