# Comparativo das bases

Gerado por `scripts/comparativo.py` em 23/10/2026 13:51; não editar à mão. Números da auditoria vêm de `out/indices/` e `out/experiments/` (ver `analise/tabelas_auditoria.md`); licença, login, tamanho publicado, tipo de falha, ruído e adequação vêm de `analise/atributos_bases.csv`, que cita a ficha de cada base. IDMT e A2212 são referência, transcritas de `../docs/DATASETS.md` e dos CSVs do trabalho do ICIT. Na MIMII e no ToyCar as contagens usam o subconjunto auditado (`unidade.str.contains('_id_')`; `not unidade.str.startswith('desconhecida')`). Atalho de nível = acurácia balanceada de RMS + pico mais de 0.10 acima do acaso, no split por gravação ou dentro da unidade.

## Estrutura e riscos

| base | unidades físicas | classes | classe × unidade | canal acústico | verdade de rotação | falhas elétricas/tensão | ruído real | tamanho | licença | atalho de nível | unidade reconhecível (forma, split por gravação) | rastreador |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **mimii** | 8 | 2 | não confundida | 16 kHz | não | parcial: anomalia "voltage change" nos ventiladores, sem rótulo por arquivo | sim (ruído de fábrica misturado) | 100.2 GB (subconjunto baixado: 19.95 GB) | CC BY-SA 4.0 (DUE: CC BY-NC-SA 4.0) | sim: 0.52 vs acaso 0.500 (gravação); 0.77 dentro da unidade (acaso 0.50) | 1.00 (acaso 0.125) | — |
| **toycar** | 12 | 2 | parcial (6 de 12 unidades com ≥2 classes) | 16 kHz | não | parcial: tensão da fonte CC variada como condição de operação | sim (ruído de fábrica misturado) | 1.93 GB | CC BY 4.0 (ToyADMOS2 completo: não comercial) | sim: 0.64 vs acaso 0.500 (gravação); 0.51 dentro da unidade (acaso 0.50) | 0.87 (acaso 0.083) | — |
| **mafaulda** | 11 | 10 | parcial (4 de 11 unidades com ≥2 classes) | 51.2 kHz | tacômetro (1951 arquivos) | não | não | 12.9 GB | não encontrado | sim: 0.43 vs acaso 0.100 (gravação); 0.89 dentro da unidade (acaso 0.50) | 0.40 (acaso 0.091) | 0.97 dos arquivos a ≤2 % |
| **uoemd** | 8 | 8 | confundida (classe = unidade) | 42 kHz | setpoint do inversor (64 arquivos) | sim: desequilíbrio de tensão/falta de fase (classe mista) e falha de estator | não | 3.25 GB | CC BY 4.0 | não: 0.18 vs acaso 0.125 (gravação) | 0.45 (acaso 0.125) | 0.47 dos arquivos a ≤2 % |
| **imvacd** | 8 | 8 | confundida (classe = unidade) | 44.1 kHz, 48 kHz | setpoint do inversor (256 arquivos) | sim: mesmas classes elétricas da UOEMD | não | 3.68 GB | CC BY 4.0 | não: 0.12 vs acaso 0.125 (gravação) | 0.44 (acaso 0.125) | 0.57 dos arquivos a ≤2 % |
| **kaist** | 5 | 3 | confundida (classe = unidade) | 51.2 kHz | não | não | não | 4.25 GB (áudio 47 MB) | CC BY 4.0 | sim: 0.50 vs acaso 0.333 (aleatório; sem split por gravação) | — | — |
| **idmt** | 3 | 3 | confundida (classe = motor) | 44,1 kHz | não | estados por tensão e carga (não são falhas) | sim (fala, atmo, branco, stress test; só no teste) | 1.455 GB | CC BY-SA 4.0 | sim: 0.99 vs acaso 0.333 (protocolo do Artigo3) | — | — |
| **a2212** | 4 | 3 | parcial (rolamento = motor 4) | 16 kHz (1 arquivo a 44,1 kHz) | velocidade comandada no nome | não | não | 14.5 MB | não encontrado | sim: 0.56 vs acaso 0.333 (protocolo do Artigo3) | — | — |

## Adequação a cada caminho de artigo

Caminhos: (2) quais distúrbios de alimentação são audíveis; (4) tacômetro acústico; (5) vários motores, um microfone; (6) auditoria de avaliação; (8) anomalia sem rótulo com features harmônicas; (ordens) features de ordem relativas à rotação rastreada. "Caso de estudo" em (6) quer dizer que a base não permite deixar um motor de fora, mas é exatamente o tipo de base que uma auditoria precisa examinar.

| base | (2) | (4) | (5) | (6) | (8) | ordens |
|---|---|---|---|---|---|---|
| **mimii** | parcial: mudança de tensão existe mas não é identificável por arquivo | não: sem rotação | sim: 4 ventiladores e 4 bombas no mesmo arranjo | sim: 8 unidades com normal e anomalia | sim: normal abundante e ruído real (atalho de nível dentro da máquina) | não: sem rotação |
| **toycar** | parcial: tensão controlada como condição e não como falha | não: tensão nominal sem rpm | sim: 6 carrinhos em condições compatíveis | parcial: 6 carrinhos com normal e anomalia mas 2 concentram os dados | sim: normal abundante e ruído real | não: sem rotação |
| **mafaulda** | não | sim: tacômetro 1 pulso/volta em 12–62 Hz (varreduras sem rampa contínua) | não: uma bancada | caso de estudo: uma bancada e varreduras vizinhas no split aleatório | não | sim: falhas mecânicas com rotação medida (sem falha elétrica) |
| **uoemd** | parcial: falha de tensão real mas classe = motor e classe mista | parcial: setpoint do inversor e rampas sem verdade | sim: 8 motores do mesmo modelo em 4 velocidades conhecidas | caso de estudo: classe = motor | não | parcial: falhas mecânicas e elétricas com rotação rastreável mas classe = motor |
| **imvacd** | parcial: falha de tensão real mas classe = motor | parcial: setpoint do inversor | sim: 8 motores e 3 celulares (troca de microfone) | caso de estudo: classe = motor | não | parcial: rotação rastreável mas classe = motor |
| **kaist** | não | não: uma velocidade sem tacômetro | não | caso de estudo: 1 gravação por classe | não | parcial: BPFO/BPFI visíveis em uma velocidade |
| **idmt** | parcial: tensão alterada mas classe = motor | não: sem verdade de rotação | parcial: 3 unidades de um modelo | caso de estudo: classe = motor | não | não |
| **a2212** | não | parcial: velocidade comandada no nome do arquivo | parcial: 4 motores em várias velocidades | caso de estudo: rolamento = motor 4 | não | parcial: rotação rastreável em parte dos motores |

## Fontes por base

- **mimii**: MIMII (fan e pump a 6 dB) — Purohit et al. DCASE 2019 (Zenodo 3384388); ficha `docs/bases/mimii.md`; login: não
- **toycar**: ToyCar DCASE 2022 (ToyADMOS2) — Harada et al. DCASE 2021; Dohi et al. 2022 (Zenodo 6355122/6462969/6586456); ficha `docs/bases/toycar.md`; login: não
- **mafaulda**: MaFaulDa (UFRJ) — Ribeiro et al. SBrT 2017; Marins et al. J. Franklin Inst. 2018; ficha `docs/bases/mafaulda.md`; login: não
- **uoemd**: UOEMD-VAFCVS (Ottawa) — Data in Brief 2024 (Mendeley msxs4vj48g v2); ficha `docs/bases/uoemd.md`; login: não
- **imvacd**: IM-VACD (Ottawa por celular) — Sehri MSSP 259 2026 (Mendeley yc8yhg5xjd v2); ficha `docs/bases/imvacd.md`; login: não
- **kaist**: KAIST (Jung et al.) — Data in Brief 2023 (Mendeley ztmf3m7h5x v6); ficha `docs/bases/kaist.md`; login: não
- **idmt**: IDMT-ISA-ELECTRIC-ENGINE — Grollmisch et al. EUSIPCO 2019; ficha `../docs/DATASETS.md §1`; login: não
- **a2212**: BLDC A2212 — Estacio et al. Data in Brief 50 2023; ficha `../docs/DATASETS.md §2`; login: não
