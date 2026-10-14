# Prompt — Fase 1 exploratória do Artigo 3 (MotorSound)

> Cole este texto no Claude Code aberto em `C:\Users\erick\dev\Decifrae-Artigo\Artigo3(MotorSound)`.
> Leia o `CLAUDE.md` e o `PLANO_PIPELINE_MOTOR.md` desta pasta antes de qualquer ação.

---

Estamos começando o Artigo 3 do Decifrae: diagnóstico do estado de operação de motores
elétricos pelo som, reaproveitando a pipeline de transcrição do baixo e do violão. Leia o
`CLAUDE.md` inteiro antes de começar; as regras de dados e de split dele não são negociáveis.

Esta fase é **exploratória**. Quero saber três coisas, nesta ordem: se a ideia funciona,
como ela se compara ao que a literatura e os próprios notebooks das bases já fazem, e que
artigos dá para tirar disso. Seja experimental: muitas pipelines, comparação justa, todo
resultado registrado, inclusive os ruins. Não otimize nada até termos o quadro geral.

Use sub-agentes em paralelo sempre que as tarefas forem independentes. A sessão principal
cuida do núcleo (`motorsound/`), do runner e dos commits; os sub-agentes escrevem só nos
arquivos que você atribuir e não commitam.

Ao fim de cada etapa abaixo, faça um commit e me mande um resumo curto do que saiu.

## Etapa 0 — Preparação (sessão principal)

1. `git init`, `.gitignore` (ignorar `data/`, `.venv/`, `*.wav`, `*.zip`, caches), `.venv`
   com as dependências do `CLAUDE.md`.
2. Criar a estrutura de pastas do `CLAUDE.md`. Mover `motor_acoustic.zip` para
   `referencia/` e descompactar lá; é um esqueleto anterior, só para consulta.
3. Existem dois zips do IDMT de 1,5 GB na pasta (`IDMT-ISA-ELECTRIC-ENGINE.zip` e
   `IDMT-ISA-ELECTRIC-ENGINE (1).zip`). Compare o hash dos dois. Se forem iguais, **me
   pergunte** antes de apagar um. Mova o que fica para `data/raw/` e descompacte em
   `data/idmt/`.
4. Verifique se a base A2212 já está na pasta. Se não estiver, pare e me peça: ela vem de
   https://data.mendeley.com/datasets/j4yr5fmhv4/1.
5. Inventário real das duas bases em `docs/DATASETS.md`: árvore de pastas, padrão de nomes,
   contagem por classe, por condição de ruído e por split, duração e taxa de amostragem
   reais, como identificar a gravação de origem de cada recorte do `train_cut/`. Leia o PDF
   de descrição que vem dentro do zip do IDMT. Documente o que encontrar, não o que eu disse.
6. Defina os splits em `motorsound/splits.py` seguindo o `CLAUDE.md` (validação de dentro
   do `train/` do IDMT, agrupada por gravação; grupos por arquivo no A2212) e escreva um
   teste que falha se algum grupo aparecer em dois lados.

## Etapa 1 — Literatura (sub-agentes em paralelo)

Um sub-agente por grupo abaixo. Cada um lê as fontes (texto completo quando acessível) e
escreve uma ficha por artigo em `docs/literatura/<id>.md` no formato do `CLAUDE.md`. O ponto
mais importante de cada ficha é **o protocolo de split e se há risco de vazamento** entre
segmentos da mesma gravação.

**Grupo A — o que já foi feito nestas bases**
- `grollmisch2019` — Grollmisch et al., *Sounding Industry*, EUSIPCO 2019 (autores do IDMT):
  https://new.eurasip.org/Proceedings/Eusipco/eusipco2019/Proceedings/papers/1570526697.pdf
- `strantzalis2022` — *Operational State Recognition of a DC Motor Using Edge AI*, Sensors 22(24):9658:
  https://www.mdpi.com/1424-8220/22/24/9658
- `etnext2026` — ETNeXt, Scientific Reports 2026 (usa IDMT e A2212):
  https://www.nature.com/articles/s41598-026-37590-z
- `melcnn2025` — *Acoustic-based fault diagnosis of electric motors using Mel spectrograms and CNNs*:
  https://pmc.ncbi.nlm.nih.gov/articles/PMC12835145
- `estacio2023` — artigo da base A2212: https://pmc.ncbi.nlm.nih.gov/articles/PMC10539630/
- `kaggle_afroz` — notebooks do Kaggle das bases:
  https://www.kaggle.com/code/pythonafroz/electrical-motor-preventive-maintenance-with-sound e
  https://www.kaggle.com/code/pythonafroz/electrical-motor-sound-analysis
  (se o acesso exigir login ou a CLI do Kaggle, pare e me avise).
- Procure mais trabalhos que usem o IDMT-ISA-ELECTRIC-ENGINE ou o A2212 e liste os
  números reportados numa tabela única em `docs/literatura/ESTADO_DA_ARTE.md`.

**Grupo B — rastreamento de rotação sem tacômetro**
- `peeters2019` — revisão de estimação de velocidade instantânea, MSSP 129 (2019):
  https://emsg.mit.edu/dev_site/wp-content/uploads/2024/05/77_paper.pdf
- `fsst_viterbi2015` — synchrosqueezing com deslocamento de frequência + Viterbi, JMSE 137(3):
  https://scholar.xjtu.edu.cn/en/publications/a-frequency-shift-synchrosqueezing-method-for-instantaneous-speed/
- `yoo2023` — estimação considerando múltiplos harmônicos, PHM Society 2023:
  https://papers.phmsociety.org/index.php/phmap/article/view/3718
- `wheat2025` — Harmonic Feature Space para mudança de condição de operação:
  https://arxiv.org/pdf/2504.10661

**Grupo C — features clássicas e embarcado**
- `vanaraj2026` — ZCR, RMS, centróide, MFCC + Voting Classifier em BLDC:
  https://www.citedrive.com/en/discovery/ai-powered-acoustic-analysis-for-non-intrusive-fault-detection-in-bldc-motors-of-electric-vehicles/
- `glowacz2018` — *Acoustic-Based Fault Diagnosis of Commutator Motor*, Electronics 7:299,
  doi 10.3390/electronics7110299
- `tinyml_acoustic2025` — TinyML para detecção de anomalia acústica em ESP32:
  https://arxiv.org/pdf/2603.26135
- Procure 2 ou 3 trabalhos de detecção de anomalia não supervisionada em som de máquina
  (DCASE Task 2, autoencoder treinado só no estado normal) e faça fichas.

Ao final, a sessão principal consolida `docs/literatura/ESTADO_DA_ARTE.md`: tabela de
métodos × base × split × número reportado × risco de vazamento.

## Etapa 2 — Núcleo comum (sessão principal)

Copiar e adaptar, com cabeçalho de procedência, conforme `PLANO_PIPELINE_MOTOR.md` §2:
`motorsound/io.py`, `splits.py`, `cache.py` (do violão), `config.py` com `PreConfig` /
`PostConfig` (molde do violão), `metricas.py`, `stats.py` (do violão, reamostrando por
gravação), `runner.py` (molde do `runner.py` do violão) e `sequencias.py` (concatena
arquivos para medir atraso de detecção). Testes para splits, cache e métricas.

## Etapa 3 — Pipelines (sub-agentes em paralelo, um por pipeline)

Cada pipeline é um arquivo em `pipelines/` com a interface do `CLAUDE.md` e uma linha em
`docs/PIPELINES.md`. Lista inicial:

| id | Tipo | Ideia |
|---|---|---|
| `p00_logmel_cnn` | reprodução | log-mel 30×32 + CNN de 2 conv (Strantzalis 2022, rota STM) |
| `p01_mfcc_cnn1d` | reprodução | MFCC 13 + Conv1D (Strantzalis 2022, rota Edge Impulse) |
| `p02_grollmisch` | reprodução | baseline dos autores do IDMT (detalhes da ficha) |
| `p03_etnext` | reprodução | DWT sym4 7 níveis + histograma ternário + NCA/Chi2 + kNN/SVM |
| `p04_classicas` | inspirada | ZCR, RMS, centróide, MFCC + Voting (Vanaraj 2026) |
| `p05_melcnn` | reprodução | Mel + CNN (melcnn2025) |
| `p10_decifrae_pyin_viterbi` | decifrae | `_viterbi_pitch` do baixo para rotação + features + cosseno do violão + HMM de estados do violão |
| `p11_decifrae_harmsum` | decifrae | igual à p10, emissão só por soma harmônica (sem pYIN; versão embarcável) |
| `p12_decifrae_sem_viterbi` | decifrae | p11 com argmax por quadro (isola o Viterbi de rotação) |
| `p13_decifrae_sem_hmm` | decifrae | p11 sem o HMM de estados (isola o violão) |
| `p20_hfs` | inspirada | Harmonic Feature Space normalizado pela rotação estimada (Wheat 2025) |
| `p21_envelope` | inspirada | espectro de modulação do envelope + classificador raso |
| `p30_anomalia` | inspirada | treinado só em `good`/saudável; escore de anomalia (DCASE) |

Ajuste a lista ao que as fichas encontrarem: se um artigo não descreve o bastante para
reproduzir, rebaixe para "inspirada" e registre por quê.

## Etapa 4 — Experimentos de validação (sessão principal roda; análise pode ser paralela)

Tudo **só na validação**. Para cada pipeline e cada base:
1. desempenho por janela e por condição de ruído do IDMT;
2. **auditoria de vazamento**: a mesma pipeline com split aleatório por segmento contra o
   split por gravação; a diferença é um resultado em si;
3. **generalização entre condições**: treinar só em `pure` e testar nas outras;
4. atraso de detecção e trocas espúrias nas sequências concatenadas;
5. custo estimado para o ESP32.

## Etapa 5 — Bancada de ruídos (sub-agentes em paralelo)

Montar `ruidos/` copiando o `modulo_ruidos.py` (Lee & Shen) do baixo e o `noise.py` do
violão. Dois usos, em experimentos separados:

1. **Robustez**: aplicar SNR branco/rosa, percussivo, reverb e os distúrbios de Lee & Shen
   sobre os áudios de validação e medir a queda de cada pipeline. Curvas de acurácia × SNR.
2. **Bancada sintética com verdade conhecida**: os distúrbios de Lee & Shen vêm da qualidade
   de energia elétrica, e a classe `broken` do IDMT é tensão chaveada a cada 18 ms. Gere um
   tom de motor cuja rotação segue a tensão perturbada (sag, swell, flicker, interruption) e
   use para medir o erro do rastreador de rotação e o atraso de detecção. Isso é validação
   de comportamento, não resultado principal.

Guarde estes resultados separados: podem virar um artigo próprio depois.

## Etapa 6 — Análise e ideias de artigo (sessão principal)

Escreva, em estilo de artigo e em parágrafos, a partir dos CSVs:

- `artigo-ICIT/METODOLOGIA.md` e `artigo-ICIT/RESULTADOS.md`: o que foi feito e o que saiu.
- `artigo-ICIT/IDEIAS.md`: para cada tema abaixo (e outros que os dados sugerirem), diga
  que evidência os experimentos já dão, o que falta medir, a novidade em relação ao
  `ESTADO_DA_ARTE.md` e um veredito (forte / possível / fraco):
  1. transferência de técnicas de transcrição musical para monitoramento de máquinas;
  2. auditoria de vazamento: quanto da acurácia publicada nestas bases vem do split;
  3. generalização entre condições de ruído e entre bases (features físicas vs CNN);
  4. tacômetro acústico de baixo custo (a rotação como produto, não só como feature);
  5. compromisso atraso de detecção × falso alarme com HMM online;
  6. fronteira de Pareto custo no ESP32 × acurácia;
  7. detecção de anomalia treinada só no estado normal;
  8. distúrbios de qualidade de energia como bancada acústica (Lee & Shen);
  9. calibração automática dos hiperparâmetros (BO/PSO/GA), ligando com o BIMO.
- `artigo-ICIT/DECISAO.md`: a ideia funciona? Qual tema tem a melhor relação entre
  evidência e esforço para o ICIT até 26/10? O que eu levo para o orientador?

Não abra o conjunto de teste nesta fase. Quando a `DECISAO.md` estiver pronta, pare e me
mostre.
