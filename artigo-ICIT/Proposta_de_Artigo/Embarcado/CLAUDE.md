# CLAUDE.md — `embarcados/` (porte para ESP32 e artigo ICIT 2027)

Esta pasta fica em `Artigo3(MotorSound)/embarcados/`. Contém o porte em C da pipeline
`p11_decifrae_harmsum` (feito por um colaborador), o firmware ESP-IDF, os testes de equivalência
Python × C e os artefatos do artigo do IEEE ICIT 2027.

## Contexto em uma frase

O Decifrae trata o motor como um "instrumento de uma corda só": acha a rotação pela soma dos
harmônicos (como o baixo acha a nota), decodifica com Viterbi, extrai 13 features, classifica
por cosseno ao protótipo (como o violão) e suaviza no tempo com um HMM de estados. O artigo
mostra se isso **cabe e roda num ESP32**, com equivalência comprovada contra o Python e com uma
avaliação honesta do que as bases permitem concluir.

## Prazos

- Submissão ICIT 2027: **segunda 26/10/2026** (regular 6 pp ou WiP 4 pp, mesmo prazo).
- Ponto de decisão regular × WiP: **domingo 25/10, 12h**.

## Referência e o que é somente leitura

- **Referência dourada:** a implementação Python de `p11_decifrae_harmsum` em `../pipelines/`
  e `../motorsound/` (`rotacao.py`, `classificador.py`, `hmm.py`, `sequencias.py`, `_decifrae.py`).
  **Nunca edite esses arquivos.** Se precisar instrumentar (salvar etapas intermediárias), faça
  um wrapper em `embarcados/` que importa a referência sem alterá-la.
- Somente leitura: tudo fora de `embarcados/` (incluindo `../out/`, `../artigo-ICIT/`, os repos
  de Artigo1/Artigo2 e `../../Artigo(2027)`).
- Resultados antigos (T1–T9 em `../artigo-ICIT/RESULTADOS_tabelas.md`) são diagnóstico
  histórico. Podem ser citados, nunca sobrescritos.

## Bases

- Para o ICIT, usamos o que já está baixado: **IDMT-ISA-ELECTRIC-ENGINE** e **BLDC A2212**.
- Problemas conhecidos (não "consertar" com split esperto; reportar):
  - IDMT: cada classe gravada num motor diferente → classe = motor; RMS e pico sozinhos dão ~99 %.
  - A2212: rolamento = motor 4 → vazamento; o rastreador com faixa 15–60 Hz não se transfere.
- Outras bases (ToyADMOS/ToyCar, MIMII, MaFaulDa, UOEMD, IM-VACD, KAIST) só entram se **já
  estiverem baixadas** em `../../Artigo(2027)/data/` e a auditoria de lá já existir. Nada de
  download novo acima de 2 GB antes de 26/10.
- **Não abra o conjunto de teste oficial do IDMT** sem minha autorização explícita.

## Regras de protocolo

- A unidade independente é a **gravação** (ou o motor), nunca a janela. Janelas de uma
  gravação ficam todas na mesma partição.
- Normalização, protótipos, limiares e pesos: ajustados **só no treino**. Seleção: só na validação.
- Tolerâncias de equivalência definidas e commitadas **antes** de rodar a comparação final.
- Nenhum número digitado à mão em `.md`, `.tex` ou figura: tudo sai de script lendo CSV/JSON.
- Um CSV + `_meta.json` (commit, seed, config, data) por experimento, em pastas com timestamp,
  nunca sobrescritos.
- Separar sempre, inclusive nos nomes de arquivo e no texto:
  1. estimativa analítica; 2. medição no PC; 3. build do firmware (`idf.py size`);
  4. **medição no ESP32 físico**. Só o 4 pode ser chamado de "medido no ESP32".
- Energia **não** é medida (não temos sensor de corrente). Não estime energia por datasheet.
  Entra como trabalho futuro, junto com microfone I2S real e motor em bancada.
- Não substitua silenciosamente um algoritmo do Python por outro "parecido" no C. Toda
  aproximação vira uma **variante nomeada** (ex.: `p11_c_coarse2fine`), comparada com a referência.

## Hardware e ferramentas

- ESP32 físico no USB. O modelo exato (ESP32 / ESP32-S3) é lido por `esp_chip_info` e gravado
  em `results/manifests/bancada.json` antes de qualquer medição.
- ESP-IDF v5.x, ESP-DSP. CPU fixa em 240 MHz, Wi-Fi e BT desligados durante medições.
- Áudio entra no chip **pela serial** (processor-in-the-loop) ou pela flash; o tempo é medido
  com `esp_timer_get_time()` só em torno do processamento.
- Windows: caminhos com parênteses (`Artigo3(MotorSound)`) sempre entre aspas e com `pathlib`.

## Estrutura

```
embarcados/
├── CLAUDE.md
├── README.md                 como reproduzir tudo
├── reports/                  auditorias e relatórios (repo, equivalência, benchmarks, auditoria independente)
├── configs/                  experiments/, embedded/ (congelados com hash)
├── src/c/                    núcleo C portável (host e alvo)
├── src/python/               wrappers que instrumentam a referência, scripts de ouro e medição
├── firmware/                 projeto ESP-IDF
├── tests/                    unit/, equivalence/, integration/
├── results/                  predictions/, metrics/, csv/, manifests/
├── figures/pdf, figures/png
├── tables/latex
└── paper/                    METODOLOGIA.md, RESULTADOS.md (pt, estilo artigo) + LaTeX IEEEtran (en)
```

Se o colaborador já usou outra estrutura, **adapte-se à dele** e registre o mapeamento no README;
não mova arquivos dele sem necessidade.

## Commits

- Branch de trabalho: `embarcado-icit` (crie se não existir; não trabalhe na principal).
- Commite resultados validados, figuras e os dois `.md` de `paper/` assim que cada etapa passar.
- Áudio, dados e binários grandes nunca são commitados.

## Pare e pergunte antes de

- abrir o teste oficial do IDMT;
- editar qualquer arquivo fora de `embarcados/`;
- download acima de 2 GB, login ou licença restritiva;
- treinar um modelo novo que não exista na referência;
- mudar o tema ou as afirmações centrais do artigo.
