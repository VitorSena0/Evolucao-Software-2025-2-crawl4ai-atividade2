# Guia: análise de estratégias de branches e releases (notebook) — Executando no Kaggle (T4 x2)

Resumo

- O notebook clona um repositório, coleta evidências (branches, tags, CONTRIBUTING/CONTRIBUTORS), calcula métricas temporais e gera prompts controlados para análise assistida por LLM.
- O foco é produzir análises embasadas por evidências (contagens, amostras, datas) e evitar alucinações do modelo.

Pré-requisitos
- Conta no Kaggle com permissão para criar Notebooks.
- (Opcional) GPU ativada em Settings → Accelerator: GPU (T4). No caso de T4 x2 do Kaggle, o notebook força uso de 1 GPU para estabilidade.

Como usar este guia (passo-a-passo rápido)
1. Criar Notebook no Kaggle: https://www.kaggle.com/code → Create Notebook.
3. Copiar/Importar o `.ipynb` original para o Kaggle (Upload Notebook ou colar células).
4. Rodar as células na ordem: 1 → 2 → ... (o notebook já foi organizado para definir funções/variáveis antes da coleta).
5. Verificar o bloco de saída com as evidências coletadas (branches_total, tags_total, samples, métricas).
6. Revisar as análises geradas (Branching Model e Estratégia de Releases), e ajustar prompts se quiser análise diferente.

Visão geral do que o notebook faz (resumido por células)
- Célula 1 (instalação e configuração de ambiente)
  - Define helper pip_install / pip_uninstall.
  - Força uso de GPU única com oscilação para T4: `os.environ["CUDA_VISIBLE_DEVICES"] = "0"`.
  - Instala transformers/accelerate e tenta instalar bitsandbytes (quantização 4-bit). Se bitsandbytes falhar, faz fallback para fp16.
  - Carrega model/tokenizer local ou remoto (opcional) e define função `ask_codellama()` — responsável por:
    - truncar prompt ao MAX_CTX (ex.: 16384)
    - calcular espaço restante para geração
    - mover inputs para o mesmo device do modelo
    - gerar segura e eficientemente (torch.inference_mode())
  - Resultado: ambiente LLM preparado (quando aplicável) e controle de contexto ativado.

- Célula 2 (coleta de evidências do Git)
  - Clona o repositório (ex.: `unclecode/crawl4ai`) em `/kaggle/working/...` (ou reutiliza cópia existente).
  - Coleta:
    - todas as branches remotas (`git branch -r`), normaliza nomes (remove `origin/`) e conta.
    - todas as tags (`git tag --list`) e lista ordenada por data (mais recentes / mais antigas).
    - últimos commits e top autores.
    - busca recursiva por arquivos CONTRIBUTING*.md / CONTRIBUTORS*.md (case-insensitive) e extrai trecho.
  - Gera e exibe um resumo com: total de branches, total de tags, exemplos, top buckets (prefixos), amostras ordenadas e trecho do arquivo de contribuições.

- Célula 3 (prompt inicial de análise: branching + releases)
  - Monta prompts com regras estritas para a LLM:
    - “Não invente branches”
    - “Se faltar evidência, diga ‘evidência insuficiente’”
    - Fornece dados brutos e pede classificação (GitFlow / GitHub Flow / Trunk-Based) e análise de estratégia de releases.
  - Chama `ask_codellama()` e exibe respostas (a qualidade depende de o LLM ter sido carregado ou de se usar a HF Inference API).

- Célula 4 (evidências fortes sobre merges e merge-base)
  - Calcula métricas mais técnicas:
    - ahead/behind entre main e outras branches (número de commits exclusivos).
    - merge-base de release/* com main e develop (ajuda a entender de onde vem o release).
    - últimos merges em main e develop (padrão PR-based vs merges diretos).
    - targets (commitish) de tags v*.
  - Estas saídas ajudam a confirmar o fluxo de trabalho (ex.: se releases saem de develop ou de main).

- Células 5–7 (prompts refinados e métricas de releases)
  - Cria prompts restritivos onde a LLM só pode usar as evidências fornecidas.
  - Separa famílias de tags (v*, docker-rebuild-*, “weird/mixed”) e calcula estatísticas: média/min/max/mediana de dias entre tags.
  - Pede classificação da estratégia de releases usando estes números (ex.: LTS + Current, Release Train, Rapid, Ad-hoc, Mista).

Saída esperada
- Resumo inicial com:
  - branches_total (ex.: 147)
  - tags_total (ex.: 40)
  - present_standard (ex.: ['main','develop','staging'])
  - amostras de branches e tags (listas)
  - trecho do CONTRIBUTORS.md / CONTRIBUTING.md
- Métricas de tags:
  - stats para v*: avg_days, median_days, min/max, count
  - stats para docker-rebuild-*
  - stats para tags mistas
- Análises em texto pelo LLM com seções: Classificação, Evidências, Riscos/Observações.
- Saídas de git avançado: merge-base, merges recentes, ahead/behind.

Como executar no Kaggle (dicas específicas para T4 x2)
  - os.environ["CUDA_VISIBLE_DEVICES"] = "0"
  - device_map={"": 0} ao carregar o modelo (quando usar HF Transformers).
- Ordem recomendada:
  1. Rodar Célula 1 (instalações e carregamento do LLM). Se não for usar LLM local, pule a parte de carga do modelo e use a Inference API na célula correspondente.
  2. Rodar Célula 2 (clonagem + coleta).
  3. Rodar Células 3–7 na sequência (análises e métricas).
zz

Interpretação das métricas (o que procurar)
- Branching
  - Muitas branches `develop`, `release/*`, `hotfix/*` → indício de GitFlow.
  - Predominância de `main` e merges pequenos frequentes sem longas-lived branches → indício de Trunk-Based/GitHub Flow.
  - Relação main...develop (branch_only >> main_only) indica atividade concentrada na develop.
- Releases / Tags
  - Família `v*` com média de ~19 dias entre tags → cadência moderada e possivelmente Release Train / frequent releases.
  - Presença de `docker-rebuild-*` tags paralelas às `v*` indica builds de imagem/recompilação frequente (pode ser ponto “Current” ou pipeline separado).
  - “Weird/mixed” tags (vr*, v.3.72, etc.) indicam inconsistência na semântica de versionamento.
- Merge-base e merges
  - Se release/* tem merge-base igual com develop → releases são originadas de develop.
  - Últimos merges em main que são do tipo "Merge pull request ..." mostram uso de PRs.

Problemas comuns e como resolver
- Erro de device (Expected all tensors on same device: cuda:0 and cuda:1)
  - Solução: não chame model.to("cuda") quando usar device_map="auto"; preferir detectar device via next(model.parameters()).device e mover inputs para esse device; ou forçar `CUDA_VISIBLE_DEVICES="0"` e device_map={"": 0}.
- bitsandbytes / Triton / Torch incompatível
  - Solução: tentar `pip install -U bitsandbytes` e verificar import. Se falhar, usar fallback para fp16 (o notebook já faz isso).
- protobuf MessageFactory.GetPrototype (protobuf 6.x)
  - Solução: reinstalar protobuf compatível (>=3.20,<4) antes de importar transformers (pode gerar warnings de dependency resolver no Kaggle).
- Falha ao clonar (erro de rede / permissões)
  - Verifique Internet ON no Kaggle e URL do repositório. Para repositórios privados, use git+token ou configure SSH (não recomendado no Kaggle).
- Respostas “inventadas” pelo LLM
  - Use prompts restritivos contidos no notebook (já implementados) ou comprove com as evidências mostradas; caso necessário, restrinja ainda mais as instruções: “Use somente os itens listados em DADOS”.
