# Resultados (fase exploratória, só validação)

Escrito em 14/10/2026. Todo número deste texto vem das tabelas T1–T9 de
`RESULTADOS_tabelas.md`, geradas por `experimentos/tabelas_resultados.py` a partir dos CSVs
listados em cada tabela; nenhuma acurácia foi digitada fora delas. O conjunto de teste do IDMT
não foi lido. As figuras estão em `out/figuras/` (`snr_idmt.png`, `pareto_custo.png`).

## A validação limpa do IDMT não discrimina nada

Na validação por blocos temporais do IDMT, 16 das 18 pipelines de classificação ficam entre
0,99 e 1,00 de acurácia por janela, e uma terceira em 0,988 (T1; as de anomalia, p30 e p30b,
têm a acurácia limitada a 2/3 por desenho e são avaliadas por AUC). Isso inclui um classificador de dois números, o RMS e o pico da janela
(`b00_nivel`), que chega a 0,991 mesmo com o RMS normalizado, porque o fator de crista sozinho
separa as classes: mediana de 9,0 dB em *good*, 12,7 dB em *broken* (a tensão chaveada torna o
sinal impulsivo) e 6,3 dB em *heavyload*. O controle com rótulos permutados por grupo cai para o
acaso (0,34), então não se trata de vazamento no código, e sim de uma base em que cada classe é
um motor físico gravado numa sessão só. O split aleatório por segmento, que é o protocolo da
maior parte da literatura, não muda nada aqui porque já não há o que inflar. Consequência
prática: os 99–100 % publicados no IDMT (ESTADO_DA_ARTE §1) não informam sobre o método, e o
único número publicado com split limpo, o de Grollmisch et al. (82–100 % por condição), só é
comparável com o nosso teste oficial, que ainda não foi lido.

## O ruído separa as pipelines, e a rotação é a feature mais robusta

Com o treino limpo e ruído aplicado só à validação (T2, Figura `snr_idmt.png`), as pipelines se
separam. A 10 dB de ruído rosa, as que dependem da estrutura harmônica ou de picos tonais se
mantêm (p02, baseline dos autores do IDMT, 1,00; p10 e p11 da família Decifrae, 1,00 e 0,99; o
controle só-rotação p20b, 0,99), enquanto as de forma espectral e as CNNs pequenas caem para
0,64–0,72 (b01, p04, p00, p05). A distância cresce com o ruído: com ruído branco a 0 dB, só a
p20b, que usa apenas três números derivados da rotação rastreada (mediana, dispersão e fração de
silêncio), continua acima de 0,95 (0,97; 0,92 a −5 dB), seguida de longe pelos histogramas de
wavelet do ETNeXt (p03b 0,87, p03 0,72); todas as outras estão entre o acaso e 0,64. A soma
sobre 48 harmônicos dá ao rastreador um ganho de processamento que nenhuma feature de forma
espectral tem. A comparação pareada confirma, depois de Holm, que a p20b supera a p11 completa a
5 dB de ruído rosa (+0,25; T4) e que o termo pYIN da emissão ajuda nesse mesmo cenário (+0,10).
A ressalva é estrutural: no IDMT rotação diferente também é motor diferente, então a robustez
mostra que a rotação é medida de forma estável, não que ela diagnostica falha.

As demais perturbações contam a mesma história com nuances (T2). Impulsos (imitando as batidas
do *stress test*) derrubam as features clássicas que usam RMS e ZCR (p04 0,45, p04c 0,35) mas não
as CNNs nem a família Decifrae (0,95–1,00); degraus de ganho e reverberação afetam pouco quase
todas.

## No A2212, as CNNs e as features clássicas vencem e a família Decifrae perde

No A2212 com validação agrupada por arquivo (T1), as melhores são as features clássicas com voto
(p04 0,979), a CNN de mel (p05 0,977), a regressão logística sobre as mesmas features (p04c
0,974) e a CNN de Strantzalis (p00 0,965). A família Decifrae fica em 0,73–0,75 e o controle
só-rotação em 0,50, e as diferenças para p00 e p02 são confirmadas (−0,21 e −0,20; T4). A causa
provável é a faixa de busca do rastreador, fixada em 15–60 Hz a partir do espectro do IDMT: no
A2212, a frequência rastreada em faixas mais altas cresce com a velocidade comandada em parte dos
motores (Spearman 0,93–1,0 em M2, M4 e hélice M1; T8), numa razão de cerca de 4,4–4,9 vezes o
valor do nome do arquivo dividido por 60, mas não em M1 saudável nem em M3. Não ajustamos a faixa
olhando a validação; o resultado fica registrado como negativo com essa hipótese.

## A auditoria de vazamento mede o quanto o split infla a acurácia

No A2212, trocar o split por arquivo pelo split aleatório de segmentos aumenta a acurácia de
todas as pipelines (coluna Δ da T1): 18 pontos no kNN do ETNeXt, 8 no SVM do ETNeXt, 6 na forma
espectral, 5 no baseline Grollmisch e no envelope, 3 na CNN de Strantzalis e 2 nas features
clássicas e na CNN de mel. As versões reimplementadas do ETNeXt, que reporta 99,95 % no A2212
com seleção de atributos e avaliação no mesmo CV sobre segmentos, ficam em 0,905 (SVM) e 0,770
(kNN) quando a seleção é feita só no fold de treino e o split é por arquivo. Deixar um motor
inteiro de fora derruba tudo para 0,33–0,78 (coluna "motor fora"): nenhuma pipeline generaliza
para um motor que não viu, e o rolamento, que só existe no motor 4, é por construção impossível
de prever quando o motor 4 sai do treino.

## O HMM de estados ajuda quando as probabilidades são calibradas

Nas sequências concatenadas (T5), o decodificador de estados do violão reduz as trocas espúrias
das pipelines cujas probabilidades por janela são calibradas. Na p11 a 5 dB de ruído rosa, as
trocas espúrias caem de 3,7 por minuto (argmax, que é a "p13") para 0,6 (Viterbi offline,
penalidade 2) e 1,6 (filtro causal), com a acurácia praticamente igual e com mais trocas
verdadeiras perdidas (0,03 → 0,80 e 0,47 por sequência). O mesmo padrão aparece na p10 e na p21.
Nas CNNs, no baseline Grollmisch e na p20b o HMM não muda nada, porque as saídas são quase
one-hot e a penalidade de troca fixa não vence uma emissão tão confiante (calibrar essas
probabilidades, por temperatura, é o passo óbvio e não foi feito). Com janelas de 3 s, o atraso mediano de detecção ficou quase sempre em zero; a resolução é
grossa demais para comparar com os 3,8–4,6 s de resposta reportados por Strantzalis et al., e
isso precisa ser refeito com janelas de 1 s.

## A bancada sintética valida o rastreador e expõe dois limites

Na bancada com verdade conhecida (T7, 10 dB), a grade original de 10 cents travava em 4/3 da
rotação quando a energia está nos harmônicos múltiplos de quatro, como no IDMT; com 2 cents o
erro mediano cai para 0,02–0,04 % nos casos nominal, sag, swell e interrupção. Foi esse resultado,
e não a validação, que motivou trocar a grade padrão; no treino do IDMT a rotação de *good*
passou de 34,3 Hz (4/3 × 25,7) para 25,7 Hz, compatível com o fundamental fraco visível a
25,5 Hz. O Viterbi de rotação herdado do baixo atrapalha mudanças rápidas: no sag, a versão sem
Viterbi acompanha 96 % dos quadros dentro de 5 %, contra 65 % com Viterbi, e a comparação na
validação ruidosa vai na mesma direção (p12 acima da p11 a 5 dB, diferença frágil; T4). Flicker
de cerca de 1 Hz não é seguido por nenhuma variante (janela de 0,5 s longa demais), e o caso
*broken* sintético fica sempre em 4/3·f, porque a frequência do chaveamento (27,8 Hz) cai dentro
da faixa de rotação.

## Detecção de anomalia treinada só no normal

A Mahalanobis com encolhimento sobre 32 features de log-mel (p30, cerca de 4 kB) separa normal e
anômalo no A2212 com AUC 0,955, e 0,944 no único teste não confundido, saudável × hélice nos
motores 1 e 2 (T6). O autoencoder pequeno (p30b) chega a 0,989 no rolamento, mas isso é a
identidade do motor 4. No IDMT tudo dá AUC 1,0, que mede a troca de motor e não a falha. Com 10 dB
de ruído rosa a AUC continua 1,0, mas todo normal ruidoso passa do limiar: o detector marca "fora
da distribuição", não "falha".

## Custo

As estimativas analíticas (T9, Figura `pareto_custo.png`) põem a p21 (envelope, 3,1 MFLOP/s,
24 kB de RAM) e o ETNeXt (0,8 MFLOP/s) numa boa relação entre custo e robustez, e a CNN de
Strantzalis em 3,6 MFLOP/s com 7 703 parâmetros, como no artigo. O rastreador com grade de
2 cents custa cerca de 16 MFLOP/s e 167 kB de RAM, o dobro da versão de 10 cents; cabe no ESP32,
mas não é a opção barata. Nada foi medido no hardware.
