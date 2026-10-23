# CLAUDE.md — Artigo 3 (MotorSound): diagnóstico acústico de motores

Contrato de trabalho desta pasta. Leia antes de qualquer alteração.

## Contexto

Este é o terceiro projeto da pasta `Decifrae-Artigo`. A ideia é levar a pipeline de
transcrição musical do Decifrae (rastreamento de pitch com pYIN + Viterbi no baixo,
decisão por cosseno + Viterbi no violão) para o diagnóstico do estado de operação de
motores elétricos pelo som. A meta de longo prazo é embarcar o resultado num
microcontrolador (ESP32) como um monitor preventivo de baixo custo.

**A fase atual é exploratória.** O objetivo não é escrever o artigo: é descobrir se a
ideia funciona, se tem relevância acadêmica e que resultados e temas de artigo aparecem.
Isso muda as prioridades em relação ao Artigo 2:

- vale mais testar muitas pipelines rápido do que polir uma;
- todo resultado negativo é registrado, não descartado;
- nenhuma pipeline é "a nossa" até os números dizerem;
- comparação justa (mesmo split, mesmas métricas) vale mais que acurácia alta.

Alvo de publicação considerado: IEEE ICIT 2027, artigo regular de 6 páginas, submissão
até **26/10/2026 (prazo final, sem extensão)**. A fase exploratória termina com um relatório
de decisão em `artigo-ICIT/DECISAO.md`. Se os dados não sustentarem um artigo até lá, isso
é um resultado válido e deve ser dito com clareza.

## Estrutura

```
Artigo3(MotorSound)/
├── CLAUDE.md                ← este arquivo
├── PROMPT_FASE1.md          ← tarefa da fase exploratória
├── PLANO_PIPELINE_MOTOR.md  ← mapeamento baixo/violão → motor (ler antes de codar)
├── referencia/              ← esqueleto anterior (motor_acoustic.zip); só referência
├── data/                    ← NUNCA commitado (ver .gitignore)
│   ├── raw/                 zips originais
│   ├── idmt/                IDMT-ISA-ELECTRIC-ENGINE descompactado
│   └── a2212/               BLDC A2212 (Estacio et al. 2023)
├── docs/
│   ├── DATASETS.md          estrutura real dos arquivos, splits, exclusões
│   ├── literatura/          uma ficha por artigo (ver "Literatura")
│   └── PIPELINES.md         uma linha por pipeline: origem, estado, resultado
├── motorsound/              núcleo importável (io, splits, cache, métricas, stats)
├── pipelines/               uma pipeline por arquivo, interface comum
├── ruidos/                  bancada de ruídos (Lee & Shen + noise.py do violão)
├── experimentos/            scripts que geram CSVs
├── out/experiments/         CSVs, nunca sobrescritos
├── out/figuras/
├── artigo-ICIT/             METODOLOGIA.md, RESULTADOS.md, IDEIAS.md, DECISAO.md
└── artigo-2027/             auditoria das bases para o artigo completo de 2027 (tem CLAUDE.md próprio)
```

## Código reaproveitado dos outros artigos

Os repositórios `../Artigo1(cross-talk)/Decifra-Baixo` e `../Artigo2(Acordes)/Decifra-Violão`
são **somente leitura** a partir daqui. Não editar, não importar por caminho relativo.
O que for reaproveitado é **copiado** para esta pasta com um cabeçalho de procedência:

```python
# Origem: Artigo2(Acordes)/Decifra-Violão/viterbi.py::decode (commit <hash>)
# Mudanças: <o que mudou e por quê>
```

O mapeamento completo está em `PLANO_PIPELINE_MOTOR.md` §2. Resumo do que vem de onde:

| Peça | Origem |
|---|---|
| Viterbi de pitch com estado de silêncio e custo de oitava | `Decifra-Baixo/fase2_v1v2v3/v3_viterbi.py::_viterbi_pitch` |
| Passa-banda SOS, spectral gate, validação harmônica | `Decifra-Baixo/fase2_v1v2v3/v2_polifonico.py` |
| Distúrbios de Lee & Shen (2011) | `Decifra-Baixo/fase1_modular/modulo_ruidos.py` |
| Viterbi de estados em log, `change_penalty` fixo, `force_n` | `Decifra-Violão/viterbi.py::decode` |
| Classificador por cosseno | `Decifra-Violão/classifier.py` |
| Separação extração cara / decisão barata, `Pre`/`PostConfig` | `Decifra-Violão/pipeline.py`, `config.py` |
| Cache com verificação de config | `Decifra-Violão/cache.py` |
| Ruído branco/rosa a SNR, percussivo, reverb | `Decifra-Violão/noise.py` |
| Bootstrap pareado, Wilcoxon, Holm, regra de decisão | `Decifra-Violão/stats.py` |
| Congelamento antes do teste | `Decifra-Violão/congelar.py` |

Lições já pagas nos outros projetos, que valem aqui:

- matriz de transição estocástica normal trava o decodificador quando há muitos
  estados; usar o `change_penalty` fixo do violão;
- `sosfiltfilt` olha o futuro e não existe no MCU; versões "embarcáveis" usam `sosfilt`;
- Butterworth em `(b, a)` é instável em baixa frequência; sempre SOS;
- o loop do `_viterbi_pitch` é O(T·S²); com grade fina de frequência, restringir a saltos
  vizinhos.

## Dados

| Base | Onde | Observações |
|---|---|---|
| IDMT-ISA-ELECTRIC-ENGINE (Grollmisch et al., EUSIPCO 2019) | `data/idmt/` | 3 classes (good, broken, heavyload) produzidas por **tensão e carga**, não por defeito mecânico. Pastas `train/`, `test/`, `train_cut/`, `test_cut/`. Condições de ruído: pure, talking, white noise, atmo low/medium/high, stress test. 44,1 kHz, 32 bit, mono. |
| BLDC A2212 (Estacio et al., Data in Brief 50, 2023) | `data/a2212/` | 43 wav de ~10 s a 16 kHz, 4 motores. Saudável e hélice nos motores 1 e 2; motor 3 só saudável; **rolamento só no motor 4**. |

Regras de dados, sem exceção:

1. **A unidade independente é a gravação de origem, nunca o segmento.** Recortes da mesma
   gravação não podem cair em lados diferentes de nenhum split. Isso vale para treino,
   validação, teste, validação cruzada e bootstrap.
2. IDMT: o `test/` oficial é o conjunto de teste. A validação sai **de dentro do `train/`**,
   agrupada por gravação. O teste só é lido depois de `congelar.py` rodar e o Erick aprovar.
   Um script exploratório que abre `test/` é bug.
3. A2212: validação cruzada agrupada por arquivo. Toda tabela com o rolamento declara que
   ele está confundido com a identidade do motor 4.
4. A estrutura real dos arquivos é verificada e escrita em `docs/DATASETS.md` antes de
   qualquer parser. Nada de supor nome de arquivo.
5. Áudio e zips nunca entram no git.

## Literatura

Cada pipeline implementada nasce de uma ficha em `docs/literatura/<id>.md` com:
referência completa e link; **se foi lido o texto completo ou só o resumo**; pré-processamento,
parâmetros e arquitetura exatos; protocolo de split que o artigo usou; números reportados e em
qual base; o que não está descrito e teve de ser suposto.

Uma pipeline é marcada como **reprodução** só se seguir o artigo sem suposições relevantes;
caso contrário é **reimplementação inspirada em**. Se a reprodução não bater com o número
publicado, isso é registrado com a hipótese da diferença (split, vazamento, parâmetro
omitido). Não ajustar até bater.

Não inventar referência. Se um detalhe não foi encontrado, escrever "não encontrado".

## Interface comum das pipelines

Toda pipeline em `pipelines/` expõe:

```python
NOME: str
ORIGEM: str            # id da ficha em docs/literatura/ ou "decifrae"
TIPO: str              # "reproducao" | "inspirada" | "decifrae" | "baseline"
def extrair(y, sr, pre) -> Features        # cara, cacheável
def treinar(features_treino, rotulos, post) -> Modelo
def prever(modelo, features, post) -> probabilidades por janela
def custo_mcu() -> dict | None             # RAM, flash, MFLOP/s estimados, ou None
```

O runner trata todas igual: mesmo split, mesmas janelas de avaliação, mesmas métricas.

## Métricas

Por janela: acurácia, F1 macro, matriz de confusão, **por condição de ruído**.
Por sequência (arquivos concatenados com trocas de estado): atraso mediano de detecção e
trocas espúrias por minuto. Rastreador de rotação: erro em relação à verdade sintética.
Custo: parâmetros, RAM e latência estimados (e medidos no ESP32 quando houver).
Comparações entre pipelines: `stats.py` adaptado, com reamostragem por gravação.

## Convenções dos experimentos

- Um CSV por experimento em `out/experiments/`, com timestamp, **nunca sobrescrito**.
  Uma linha por (pipeline, gravação, condição), com a configuração repetida na linha.
- Cada CSV tem um `_meta.json` com commit, seed, config e duração.
- Nenhum número em `artigo-ICIT/` é digitado à mão: sai de script lendo CSV, com a origem.
- Seeds fixas e registradas.
- Resultados parciais validados são commitados assim que ficam prontos, com as figuras,
  atualizando `artigo-ICIT/METODOLOGIA.md` e `artigo-ICIT/RESULTADOS.md` em estilo de artigo.

## Sub-agentes

- Use sub-agentes **em paralelo** sempre que as tarefas forem independentes (fichas de
  literatura, implementação de pipelines distintas, experimentos de ruído distintos).
- Cada sub-agente recebe arquivos próprios para escrever e **não commita**.
- Só a sessão principal roda o runner completo, mexe em `motorsound/` e faz commits.
- Nenhum sub-agente lê o conjunto de teste.

## Pare e pergunte

- antes de abrir o conjunto de teste;
- antes de apagar qualquer arquivo de dados (inclusive o zip duplicado);
- antes de baixar algo que não seja uma das duas bases ou pacote Python;
- se um resultado parecer bom demais (acima de 99 %): primeiro procurar vazamento;
- se uma tarefa exigir editar os repositórios dos Artigos 1 ou 2.

## Ambiente

Windows, Python em `.venv` desta pasta. Caminhos com parênteses e acento
(`Artigo2(Acordes)/Decifra-Violão`): sempre entre aspas e com `pathlib`.
Dependências mínimas: numpy, scipy, scikit-learn, librosa, soundfile, matplotlib, pandas,
pywavelets; torch só para as reproduções com CNN.

## Estilo

Português nos comentários e na documentação. Textos em `artigo-ICIT/` em parágrafos
corridos, não em listas, exceto onde lista é de fato necessária.
