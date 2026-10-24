# Prompt — Testes do porte em C da p11 e artigo ICIT 2027

> Coloque o `CLAUDE.md` dentro de `Artigo3(MotorSound)/embarcados/`, abra o Claude Code em
> `C:\Users\erick\dev\Decifrae-Artigo\Artigo3(MotorSound)\embarcados` e cole o texto abaixo.
> Baseado no master prompt do colaborador, recortado para o prazo de 26/10.

---

Leia o `CLAUDE.md` desta pasta antes de qualquer coisa. Ele tem as regras, o que é somente
leitura e os prazos.

Um colaborador portou para C a pipeline `p11_decifrae_harmsum` e deixou o código nesta pasta.
Sua missão é **verificar, completar, medir e documentar** esse porte até um artigo submetível ao
IEEE ICIT 2027, com integridade científica acima de resultado bonito.

Temos **até segunda 26/10**. Por isso, nesta rodada:

- **Não** fazemos nova seleção de bases nem downloads grandes. Usamos IDMT e A2212, que já
  estão baixadas, com os problemas conhecidos reportados honestamente. Bases novas ficam para o
  artigo de 2027 (`../../Artigo(2027)`).
- **Não** reexecutamos as 20 pipelines. A referência Python da p11 e os baselines simples
  (RMS/pico e features clássicas) bastam para a comparação.
- O foco é: **equivalência C × Python, medição no ESP32 físico e artigo.**

Não pare depois de planejar. Execute tudo o que for possível; se algo travar (falta de arquivo,
de dado, de hardware), registre o bloqueio exato em `reports/BLOQUEIOS.md` e siga com as tarefas
independentes. Não afirme que algo funcionou sem saída que prove.

## Agentes

Use **sub-agentes em paralelo sempre que as tarefas forem independentes**. Cada agente reporta
evidência: arquivos, comandos, saídas, incertezas. A sessão principal integra e commita.

| Agente | Papel | Independência |
|---|---|---|
| **A · Auditor do repositório** | Inventário de `embarcados/` e da referência Python; status do porte etapa por etapa | — |
| **B · Engenheiro Python** | Wrapper que instrumenta a p11 de referência e gera vetores de ouro; baselines | Não edita a referência |
| **C · Engenheiro C/ESP32** | Completa o porte, build no PC e no ESP-IDF, firmware de medição | — |
| **D · Verificador de equivalência** | Escreve os próprios testes Python × C, etapa por etapa | **Não usa os testes do C** como prova |
| **E · Estatística e figuras** | Métricas, tabelas LaTeX, figuras, manifesto | — |
| **F · Auditor científico** | Revisa vazamento, equivalência, afirmações do texto; dá PASS/COND/FAIL | **Não aprova** só porque os outros disseram que passou |
| **G · Otimização** | Só depois da equivalência; toda otimização é variante nomeada e comparada | Mantém a versão de referência intacta |

## Etapa 1 · Auditoria (A, em paralelo com B iniciando o wrapper)

`reports/repository_audit.md` com:

1. Árvore de `embarcados/` e o que cada arquivo faz.
2. Para cada etapa da p11 — leitura/reamostragem, filtro, STFT (`stft_mag`), grade de rotação,
   soma harmônica, Viterbi de rotação, as 13 features na ordem abaixo, `_envelope_mod`, z-score,
   cosseno ao protótipo (softmax T), HMM de estados (forward e Viterbi), `absorb_short` — o status
   no C: **completo / parcial / ausente / diferente da referência**, com a linha do Python e a
   linha do C.
3. Parâmetros extraídos **do código Python** (n_fft, hop, janela, fs, faixa 15–60 Hz, passo da
   grade em cents, nº de harmônicos, pesos da emissão, penalidades de transição, change_penalty,
   T do softmax, min_dur). Nada inferido de comentário ou do C.
4. Se o build do C e do firmware funcionam hoje, com o comando e a saída.
5. Plano priorizado para o prazo.

Ordem das 13 features (confira com `_decifrae.py`; se divergir, vale o Python e registre):
0 rotação mediana · 1 IQR da rotação · 2 fração de silêncio · 3 saliência média no caminho ·
4 razão harmônica · 5–8 energia relativa das ordens 1–4, 5–12, 13–24, 25–48 ·
9 frequência do pico de modulação do envelope · 10 proeminência do pico · 11 centróide ·
12 planura espectral.

## Etapa 2 · Vetores de ouro (B)

Wrapper em `src/python/` que roda a p11 **de referência** e salva, para cada janela, a saída de
cada etapa listada acima (incluindo o caminho de rotação e o score do Viterbi, as probabilidades
do cosseno e a sequência do HMM). Conjunto fixo:

- janelas das três classes nas duas bases, só de partições já usadas (treino/validação);
- as mesmas janelas com ruído do módulo de ruídos existente (branco 20/5/0 dB e dois distúrbios
  Lee & Shen);
- casos de borda: silêncio, clipping, janela com rotação fora da faixa.

Exporte também os artefatos do modelo (médias/desvios do z-score, protótipos, matriz do HMM)
num formato que o C lê. Formato binário simples + JSON de metadados.

## Etapa 3 · Tolerâncias, antes da comparação (D, commit obrigatório antes da etapa 4)

`configs/embedded/tolerancias.json`, justificadas em `reports/c_equivalence.md` pela precisão
(float32 no C × float64 no Python) e pelo comportamento de cada etapa:

- contínuos (espectro, saliência, features, probabilidades): erro absoluto e relativo máximo;
- discretos (índice da rotação por quadro, estado do HMM, decisão): concordância exata, com
  exceções documentadas só quando houver empate real;
- Viterbi: comparar caminho **e** score.

Atenção a diferenças de convenção que o C precisa reproduzir: normalização e janela da FFT do
SciPy/NumPy/librosa, padding/centralização, `sosfiltfilt` (fase zero, bordas), interpolação,
log com epsilon, critério de desempate do argmax.

## Etapa 4 · Porte e equivalência (C completa; D verifica)

1. C completa o que falta, sem trocar algoritmo. Compila no PC (CMake, `-O2`) com uma FFT de
   referência e no ESP-IDF com a ESP-DSP, atrás da mesma interface.
2. D roda a comparação **etapa por etapa**, no PC primeiro. Se falhar, ache a **primeira etapa
   que diverge**, corrija e rode tudo de novo.
3. Saída: `results/csv/equivalencia_<timestamp>.csv` (por etapa: erro máx., médio, RMS,
   concordância), `reports/c_equivalence.md` com PASS/FAIL por etapa e a lista de todas as
   discrepâncias.
4. Depois do PASS no PC, repetir a comparação **no ESP32**, com as janelas enviadas pela serial.

Não escreva "equivalente à p11" em lugar nenhum antes de D e F aprovarem.

## Etapa 5 · Medição no ESP32 físico (C)

Antes: gravar o modelo do chip, frequência, flash, versão do IDF e flags do compilador em
`results/manifests/bancada.json`.

Para a p11 em C (referência) e, se houver, a variante otimizada:

- latência por janela (mediana, p95, máx. sobre ≥ 200 janelas, após aquecimento), separada por
  etapa (STFT, soma harmônica, Viterbi, features, classificação/HMM);
- **fator de tempo real** = tempo de processamento ÷ duração da janela (1 s e 3 s);
- RAM: pico de heap (`heap_caps_get_minimum_free_size`), pilha (`uxTaskGetStackHighWaterMark`),
  RAM estática (`idf.py size`);
- flash (`idf.py size-components`);
- fluxo contínuo: sequência good → broken → heavyload gravada na flash, processada janela a
  janela com o filtro forward do HMM; atraso de detecção de cada troca.

Mesmas janelas no PC: Python de referência e o mesmo C compilado no PC → tabela ESP32 × PC.

Se a p11 completa não couber (RAM, tempo), documente o gargalo e teste variantes nomeadas
(ex.: grade grossa → fina, menos harmônicos), sempre comparadas com a referência.

## Etapa 6 · Resultados diagnósticos (B + E)

Não reabra a escolha de base. Com IDMT e A2212, nas partições já usadas:

- p11 de referência, p11 em C, baseline só RMS/pico e features clássicas — **nas mesmas janelas**;
- três protocolos lado a lado, para mostrar o problema da base: aleatório por janela (com
  vazamento), por gravação e motor fora (A2212);
- métricas: acurácia balanceada, precisão, sensibilidade, especificidade, F1 por classe, matriz
  de confusão; ROC-AUC usando os **scores contínuos** do cosseno, não rótulos;
- nenhum intervalo de confiança tratando janelas da mesma gravação como independentes
  (bootstrap por gravação, como em `../motorsound/stats.py` ou equivalente; se for pequeno
  demais, diga).

## Etapa 7 · Figuras, tabelas e artigo (E escreve; F audita)

Figuras (PDF + PNG, tamanho de coluna IEEE, só de experimentos que rodaram):
arquitetura do sistema; espectro com o pente harmônico e caminho de rotação (normal × anômalo);
matriz de confusão; RMS/pico × clássicas × Decifrae nos três protocolos; erro Python × C por
etapa; latência por etapa e RAM no ESP32.

Tabelas LaTeX geradas por script: bases e o que cada protocolo permite; parâmetros e features;
desempenho; equivalência; recursos no ESP32 × PC. `results/manifests/manifesto.json` liga cada
tabela/figura ao experimento e ao commit.

Artigo em `paper/`:
- `METODOLOGIA.md` e `RESULTADOS.md` em português, estilo de artigo, parágrafos;
- LaTeX `IEEEtran` (conference), em inglês, 6 páginas: Abstract, Introduction, Related Work,
  Method, Experimental Setup, Results, Discussion (incluindo "o que as bases permitem concluir"),
  Conclusion e References (`references.bib`).

O texto deve deixar claro:
- a analogia "instrumento de uma corda só" motiva a representação; não diz que motor é nota;
- classificação de estado operacional ≠ predição de falha futura;
- IDMT/A2212 são motores pequenos de laboratório, não motores industriais;
- avaliação com unidade conhecida ≠ generalização para motor novo;
- medido no ESP32 ≠ estimado ≠ medido no PC;
- energia, microfone real e motor em bancada são trabalho futuro;
- se a p11 não superar os baselines simples, a contribuição se estreita para o que os dados
  sustentam (custo, interpretabilidade, equivalência comprovada, viabilidade embarcada).

Citações: só referências verificadas (reaproveite `../docs/literatura/`). Não invente.

## Etapa 8 · Portões e decisão

F emite `reports/independent_scientific_audit.md` com problemas críticos/maiores/menores e um
veredito **PASS / CONDITIONAL PASS / FAIL** para:

- A · integridade das bases e splits (com os problemas conhecidos declarados);
- B · Python reproduzível a partir das predições salvas;
- C · C completo, compilando, testes passando, equivalência verificada por D;
- D · viabilidade embarcada com medições no chip;
- E · todo número do texto rastreável a um arquivo.

Compilar não basta para PASS.

**Domingo 25/10, 12h:** pare e me mostre um resumo de cinco linhas — status de cada portão e a
recomendação:
- **Regular (6 pp)** se C e D passarem (equivalência no chip + medições);
- **WiP (4 pp)** se a equivalência passou só no PC ou as medições estão incompletas.

## Entregáveis finais

`README.md` com como reproduzir e um resumo de: bases usadas e limites; o que foi rodado;
status do porte; resultado da equivalência; medições no ESP32; figuras e tabelas geradas;
bloqueios restantes; veredito do auditor; recomendação regular ou WiP.
