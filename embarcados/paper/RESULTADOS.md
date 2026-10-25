# Resultados (artigo ICIT 2027, `embarcados/`)

> Gerado por `embarcados/src/python/gerar_artigo_textos.py` a partir de `paper/numeros.json`
> (commit 8624cbe). Não editar à mão. Validação, salvo a seção do teste oficial do IDMT (lido uma vez).
> Os números do ESP32 vêm da bancada física, com a variante `p11_c_lowmem`.

## Equivalência Python × C (medição no PC)

Em double, as 26 verificações de etapa passam em todas as 164 janelas e nas
240 configurações de sequência (26 de 26); a maior razão
erro/tolerância é 0,55. Caminho de rotação, decisão e todas as saídas do HMM
coincidem exatamente. Em float32, que é o build do microcontrolador, as 150 de
150 janelas de áudio real (limpas e ruidosas) passam em todas as etapas. As falhas
restantes estão em 2 janelas, ambas o caso de borda sintético de nível constante
com ruído (uma por modelo: borda_dc_ruido3s_ma2212, borda_dc_ruido3s_midmt). Ali o envelope é quase constante e o espectro de
modulação fica no piso de arredondamento do float32 nas duas implementações, de modo que a posição
do pico é indeterminada nessa precisão: o pico da referência vale 4,0 vezes a
potência da referência no bin escolhido pelo C, então não é empate. A frequência de modulação muda
37,1 Hz. Sob a regra congelada o comparador marcou PASS_EMPATE em `env_mod` e
`features` e FAIL em `z`; adotamos a leitura conservadora e contamos as três etapas
(env_mod, features, z) como FAIL nesses casos, o que deixa 23 de
26 etapas passando em todas as janelas (erro máximo de z 1,61). As
tolerâncias e o comparador não foram alterados. Cossenos, probabilidades e decisão passam em
todas as janelas (164 de 164). Com a variante nomeada `p11_c_sosd`, as
26 verificações passam; ela reduz o erro do envelope dizimado (máximo relativo ao pico, mediana entre
janelas) de 1,2 × 10^-5 para 1,2 × 10^-7, porque os polos do passa-baixa ficam perto
do círculo unitário e o arredondamento em float32 se acumula no estado do filtro. A afirmação que os
dados sustentam é concordância etapa a etapa dentro de tolerâncias congeladas antes da comparação
formal, com a exceção acima; não é equivalência incondicional. A auditoria independente (agente F)
reproduziu a comparação a partir de um build limpo, e as correções dela estão aplicadas aqui.

A variante `p11_c_lowmem` foi comparada com as mesmas tolerâncias do float32, primeiro no PC e
depois no chip. No PC, as 79 janelas de 1 s do ouro passam nas 18
etapas comparadas (18 sem falha; maior razão erro/tolerância 0,19;
decisão igual em 79 de 79). No ESP32, as mesmas 79 janelas
(75 de áudio real e 4 casos de borda) passam nas 8
saídas que o firmware devolve pela serial (caminho e escore do Viterbi, envelope de modulação,
features, z, cossenos, probabilidades e decisão), com 0 falhas, decisão igual em
79 de 79 e maior razão erro/tolerância 0,19. As
12 etapas intermediárias não voltam do chip e ficam ausentes, e as janelas de
3 s do IDMT não couberam na placa, então não há comparação no chip para elas. A
auditoria independente não cobriu os resultados do chip.

## Custo

Medido no PC, o C float32 leva 13,29 ms por janela de 3 s (p95
15,00 ms, máximo 15,80 ms) e 3,18 ms por janela de
1 s, contra 95,3 ms e 22,1 ms da referência Python; a razão
(7,2 vezes) inclui o custo do interpretador e não é uma comparação de algoritmos. No PC,
STFT por quadro com soma harmônica fica com 41 % da mediana, o Viterbi de rotação
com 40 % e o ramo do envelope com 17 %; features e cosseno são
desprezíveis. O fator de tempo real no PC (0,0044) não diz nada sobre o microcontrolador.

A estimativa analítica da arena em float32 é 1,86 MB para 3 s e
0,96 MB para 1 s (1,52 e 0,61 MB
sem a tabela de interpolação harmônica, que ocupa 0,35 MB); os buffers do Hilbert
dominam (fase do envelope = 64 % do total em 3 s). Em double
seriam 3,60 e 1,79 MB. Numa unidade só, a arena
(1820 e 936 KiB) é pelo menos 5,5 e
2,8 vezes a região DIRAM interna inteira que o build reporta para o ESP32-S3
(334 KiB, dos quais só 280 KiB sobram depois da alocação
estática; a IRAM do S3 está 100 % usada), e 10,3 e
5,3 vezes a DRAM do ESP32 (176 KiB): a arena contígua da
referência (`p11_c_ref`) exige PSRAM. No build do firmware da referência, a imagem ocupa
209 KiB (ESP32, 20 % da partição) e 233 KiB (ESP32-S3,
23 %); o uso estático de RAM interna é 15 KiB de DRAM mais
58 KiB de IRAM no ESP32 e 54 KiB de DIRAM no ESP32-S3. O firmware da
variante `p11_c_lowmem` para o ESP32 ocupa 215 KiB (21 % da partição;
130 KiB de código e 20 KiB de dados constantes na flash) e usa,
de forma estática, 35 de 176 KiB de DRAM e 58 de
128 KiB de IRAM. A memória de trabalho é alocada em tempo de execução e não entra
nesses números.

Medido no ESP32 (ESP32, 240 MHz, 0 MB de PSRAM), a referência não
cabe: a arena estimada é pelo menos 3,0 vezes (1 s) e
5,9 vezes (3 s) a RAM interna que o firmware consegue entregar ao
pool (308 KiB, medida). Com a variante `p11_c_lowmem`, a janela de 1 s do
A2212 cabe: o pico do pool é 303 308 bytes (296 KiB, 96 % do
pool), nada vai para PSRAM, o heap interno mínimo livre durante o processamento é
7 584 bytes e a folga mínima de pilha é 20 536 bytes. A janela de
3 s do IDMT (48 000 amostras) não cabe nesta placa: o plano de alocação não
encontra lugar com 308 KiB livres, e por isso latência, memória e
equivalência do IDMT no chip ficam como "não coube", sem número. A latência mediana por
janela de 1 s é 2647,5 ms (p95 2647,6 ms, máximo
2647,7 ms, 200 janelas), ou seja, um fator de tempo real de
2,65: o chip leva 2,65 vezes a duração do áudio e não opera em tempo real nesta
placa. A STFT por quadro com soma harmônica leva 2345,5 ms (89 % da
mediana), o ramo do envelope 199,3 ms (8 %), o Viterbi de rotação
90,7 ms (3 %), as features 10,1 ms e a normalização
1,8 ms; cosseno e saliência no caminho são desprezíveis. No PC, a etapa de rastreio
da referência (com tabela) fica com 39 % do tempo do C. Atribuímos o peso dela no
chip ao fato de a variante recalcular as posições harmônicas em double sem tabela, e o ESP32 só tem double
por emulação em software; o tempo da variante no PC não foi medido. Os
caminhos para fechar a diferença são uma variante que faça essas contas em float (que teria de ser
verificada como variante nomeada) ou uma placa com PSRAM, onde a tabela da referência cabe.
Energia não foi medida.

## Classificação nos três protocolos

No IDMT todos os métodos ficam perto do teto nos dois protocolos. O baseline de nível com RMS
normalizado, que só vê o fator de crista, chega a 0,991 de acurácia balanceada nos
blocos (0,982–0,997), e a p11 e as clássicas chegam a
1,000 e 1,000. Como cada classe é uma gravação de um motor, esses
números medem a identidade da gravação e não ordenam os métodos.

No A2212 com split por arquivo, as clássicas chegam a 0,981
(0,950–0,998), a p11 a 0,816
(0,747–0,881) e o nível bruto a 0,608. A
p11 erra sobretudo em Healthy (sensibilidade 0,577). O split aleatório
infla a acurácia balanceada em 0,033 na p11 e 0,017 nas clássicas. Com o
motor fora, todos caem: as clássicas para 0,613 e a p11 para
0,456, com sensibilidade de Bearing 0,000 por construção.
Sem o fold do M4 (Healthy × Propeller em motor não visto), as clássicas ficam em
0,920, o nível com RMS normalizado em 0,814 e a p11 em
0,685. Comparando igual por igual (Healthy e Propeller dos motores M1–M3 no
split por arquivo: clássicas 0,972, p11 0,753, nível com RMS
normalizado 0,843), tirar o motor custa 0,052 às clássicas e
0,068 à p11; a queda grande do protocolo de motor fora completo vem sobretudo da
classe Bearing, que perde o seu único motor de treino. A p11 não supera as features clássicas em
nenhum protocolo (empata no teto do IDMT) e, com motor fora, a estimativa pontual dela fica abaixo
também do baseline de nível, com intervalos sobrepostos. ROC-AUC macro da p11 por arquivo:
0,935; das clássicas: 0,996. No protocolo de motor fora a AUC junta
scores de modelos de folds diferentes e, no fold do M4, p_Bearing = 0 é imposto; por isso não a
usamos para ordenar métodos. Em toda linha com Bearing, a classe está confundida com o motor M4.

## Teste oficial do IDMT

Com as configurações congeladas e commitadas antes, o teste oficial do IDMT foi lido uma única vez
(`src/python/teste_idmt.py`), com os modelos treinados em todo o `train/`. Nas 2021 janelas
de teste (27 gravações, 9 condições), a validação quase perfeita não se sustenta:
a acurácia balanceada é 0,704 [0,615, 0,789] para as features clássicas,
0,595 [0,471, 0,714] para a pipeline e 0,476 para o baseline de
nível normalizado, com AUC macro de 0,954, 0,869 e 0,728. A pipeline atribui
a good 72,6 % das janelas broken e 45,7 % das
heavy load, e só lidera sob ruído branco (0,954 contra 0,869 das
clássicas); nas outras condições fica abaixo das clássicas. Como o treino viu uma única sessão
limpa por classe e a classe continua igual ao motor, isso mede a transferência entre sessões e
ruídos dos mesmos três motores, não o reconhecimento do estado de operação.

## Rotação estimada (interpretabilidade)

No IDMT a rotação mediana rastreada é 25,76, 27,99 e 24,65
Hz para good, broken e heavyload, com IQR de 0,18, 0,03 e
0,20 Hz; 8,6 % das janelas good ficam acima de 1,5 vezes a
mediana da classe (todas entre 2,00 e 2,02 vezes), um erro de oitava.
Como classe = motor no IDMT, essas diferenças não podem ser atribuídas ao estado. No A2212, juntando
os 43 arquivos, a correlação de postos entre a mediana por arquivo e a velocidade
que dá nome aos arquivos é 0,02, mas dentro de cada motor ela é positiva (M1
0,30, M2 0,30, M3 0,54, M4
1,00; 14, 16, 6 e
7 arquivos): o valor global perto de zero vem de diferenças de nível entre
motores (efeito de Simpson). O nível absoluto não é comparável entre motores e 3,7 %
das janelas ficam a menos de 1 % do limite superior da faixa. A faixa e a grade foram
escolhidas no IDMT; no A2212 a saída do rastreador não deve ser lida como a velocidade do eixo. Arquivos do mesmo motor não são
independentes, então a correlação é descritiva.

## Decisão em fluxo

Numa sequência de 30 janelas de 1 s de validação do A2212
(10 por classe, Healthy → Propeller → Bearing), o HMM online com penalidade
2 detectou as 2 trocas com atraso de 0 janelas no C e
0 no Python, com decisões iguais (diferença máxima de probabilidade
2,7 × 10^-5), medido no PC. No ESP32, a mesma sequência gravada na flash do chip e
processada janela a janela pela variante `p11_c_lowmem` detectou as 2 trocas com
atraso de 0 janelas, e a decisão online do chip coincide com a do Python em
30 de 30 janelas; o passo do HMM leva 104 µs (mediana). As
janelas são as limpas do ouro, quantizadas em 16 bits na imagem (o PC processou o mesmo áudio
desquantizado), e Bearing é o motor M4; isso confere a implementação, não o detector.

## O que as bases permitem concluir

A validação do IDMT não sustenta nenhuma afirmação sobre discriminar estado de operação: uma
gravação por classe torna classe, motor e sessão a mesma variável, e o baseline de dois números é
tão bom quanto qualquer modelo. O teste oficial mostra como esse teto é frágil (acurácia
balanceada 0,595 para a p11 e 0,704 para as clássicas), mas nele a classe continua
igual ao motor. O A2212 com split por arquivo sustenta afirmações sobre o estado de
motores já vistos no treino, e ali as features clássicas são melhores que a p11. Generalizar para
um motor novo é a pergunta relevante para um monitor implantado, e com 4 motores,
um defeito de rolamento num só deles e faixas de velocidade diferentes por classe, os dados só
mostram que todo método piora. Nenhum experimento trata de predição de falha futura e nenhum usa
motores industriais. Nos nossos dados o split aleatório inflou pouco a acurácia do A2212
(0,017 a 0,033), e tirar o motor, igual por igual em Healthy × Propeller,
custou um valor parecido (0,052 às clássicas, 0,068 à p11); a queda
de 0,368 e 0,360 do protocolo de motor fora completo é dominada pela
classe Bearing, que só existe no M4.
