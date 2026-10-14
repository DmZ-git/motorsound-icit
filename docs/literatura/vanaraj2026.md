# vanaraj2026 — features acústicas clássicas + Voting Classifier em BLDC

## 1. Referência

Chandra Vanaraj, P. S. Manoharan, Thenmozhi Ganesan, Hemalatha Jeyaprakash.
"AI-powered acoustic analysis for non-intrusive fault detection in BLDC motors of electric vehicles".
*Proceedings of the Institution of Mechanical Engineers, Part A: Journal of Power and Energy* (SAGE), 2026,
publicado online em 24/06/2026. Volume, número e páginas: não encontrados (o Crossref só traz o número de artigo).
DOI: 10.1177/09576509261462653.
Links: https://doi.org/10.1177/09576509261462653 ;
agregador: https://www.citedrive.com/en/discovery/ai-powered-acoustic-analysis-for-non-intrusive-fault-detection-in-bldc-motors-of-electric-vehicles/

Afiliações (Crossref): Thiagarajar College of Engineering (EEE), Kumaraguru College of Technology,
AAA College of Engineering and Technology (Índia).

## 2. Leitura

**Só o resumo** (CiteDrive + metadados do Crossref). O texto completo na SAGE devolveu HTTP 403
(paywall). Tudo o que está abaixo além do resumo está marcado como "não encontrado".

## 3. Pré-processamento, parâmetros e classificador

- Classes: operação normal, falha de rolamento, falha de hélice (três estados).
- Features citadas no resumo: Zero Crossing Rate (ZCR), RMS, centróide espectral, MFCCs.
- Taxa de amostragem, janela, hop, número de MFCCs, agregação por segmento (média? desvio?): não encontrados.
- Classificadores: "vários modelos supervisionados de ML"; o melhor foi um **Voting Classifier** (VC).
  Quais modelos entram no voto, voto rígido ou suave, hiperparâmetros: não encontrados.
- Base de dados: não encontrada no resumo. **Hipótese (não confirmada):** as três classes
  (normal, rolamento, hélice) coincidem exatamente com a base BLDC A2212 de Estacio et al.
  (Data in Brief 50, 2023), que é a nossa `data/a2212/`. Se for isso, é a comparação direta mais
  próxima do MotorSound. O resumo fala em "condições controladas de laboratório".

## 4. Protocolo de split e risco de vazamento

Protocolo: não encontrado; o resumo só diz "acurácia no conjunto de teste".

**Risco de vazamento: alto (presumido).** Se a base for a A2212, são 43 arquivos de ~10 s; para ter
dados suficientes para vários classificadores é quase certo que os arquivos foram fatiados em
segmentos e que o split foi aleatório por segmento (prática dominante nesse tipo de artigo). Além
disso, na A2212 o rolamento só existe no motor 4, de modo que mesmo um split por arquivo confunde
"rolamento" com "identidade do motor 4". ~98 % nessas condições não é evidência de que o método
detecta falha de rolamento. Rebaixar para "médio" só se o texto completo mostrar split por arquivo.

## 5. Números reportados

- Voting Classifier: **~98 % de acurácia** no teste (valor aproximado, como no resumo).
- Base: não declarada no resumo (provável A2212, ver acima).
- F1, matriz de confusão, desempenho dos demais modelos: não encontrados.

## 6. Custo embarcado

Não reportado no resumo. O artigo fala em monitoramento "não intrusivo" mas não há menção a MCU.

## 7. Para reproduzir / veredito

Seria preciso supor: sr, janela, hop, número de MFCCs, estatísticas de agregação, lista dos
classificadores do voto, tipo de voto, hiperparâmetros, duração dos segmentos e protocolo de split.
**Veredito: só inspiração** (reimplementação "inspirada em", não reprodução).

## 8. Relevância para o MotorSound → `p04_classicas`

É exatamente o baseline "barato" que precisamos e, se a base for a A2212, o número de 98 % é o que
um revisor vai citar contra nós. A pipeline `p04_classicas` deve:

- **extrair** (por janela de avaliação comum do runner, ex. 1 s): sobre quadros de 25–40 ms com hop de
  50 %, ZCR, RMS, centróide espectral e 13 MFCCs; agregar por janela com média e desvio
  (≈ 2 × 16 = 32 features). Opcional: rolloff, bandwidth, flatness (registrar como extensão nossa).
- **treinar**: `StandardScaler` + `VotingClassifier(voting="soft")` com SVM-RBF, Random Forest,
  kNN e regressão logística (suposição nossa, registrada como tal).
- **prever**: probabilidades por janela do voto suave.
- **custo_mcu**: features baratas (FFT de 512–1024 pontos + banco mel de ~26 filtros + DCT); o voto
  com RF/SVM é o caro. Reportar também a variante só com regressão logística ou árvore pequena.
- Avaliar duas vezes na A2212: (a) split aleatório por segmento, para mostrar que reproduz ~98 %;
  (b) validação cruzada agrupada por arquivo e "deixa um motor fora". A diferença (a)−(b) é um
  resultado de artigo por si (quantifica o vazamento da literatura).
- Métricas: acurácia, F1 macro, matriz de confusão por condição de ruído; na A2212, nota sobre o
  confundimento rolamento × motor 4 em toda tabela.
