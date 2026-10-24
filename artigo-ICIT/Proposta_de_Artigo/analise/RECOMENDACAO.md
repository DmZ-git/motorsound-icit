# Recomendação: bases para o artigo completo de 2027

Escrito em 23/10/2026 ao fim da auditoria. Os números citados estão em
`analise/tabelas_auditoria.md` ([T1] a [T7]) e em `analise/COMPARATIVO_BASES.md`. A
metodologia está em `analise/METODOLOGIA.md`, e os resultados em `analise/RESULTADOS.md`.

## 1. Qual base ou combinação usar

Nenhuma base pública sustenta sozinha uma comparação de métodos sem viés com falhas de motor. A
recomendação é um artigo que combina quatro bases, cada uma no papel que ela de fato permite
cumprir, e não um artigo de "classificação de falhas" numa base só.

O MIMII (ventilador e bomba, 6 dB) é a base principal para avaliar métodos. É a única em que
toda unidade tem normal e anomalia, com várias unidades por tipo de máquina, ruído de fábrica
real e taxa de 16 kHz, a mesma do microcontrolador. Ela permite deixar uma máquina de fora com
8 unidades [T1]. O ToyCar entra como segunda base do mesmo tipo, com 6 carrinhos que têm as duas
classes [T1], para mostrar que os achados não são próprios do MIMII. A MaFaulDa entra para o
tacômetro acústico e as features de ordem. É a única com rotação medida em todos os arquivos, e
nela o rastreador ficou a até 2 % do tacômetro em 0,999 dos arquivos com microfone íntegro
[T5]. A UOEMD e a IM-VACD entram como casos de estudo da auditoria (classe = motor nas 8 classes
[T1]), como fonte de falhas elétricas reais e como material para misturar motores com velocidade
conhecida.

Com essa combinação, o artigo mais forte é a continuação natural do trabalho do ICIT. Ele teria
duas partes. A primeira é uma auditoria de avaliação em oito bases. Ela mostraria que classe =
unidade por inteiro em quatro delas (UOEMD, IM-VACD, KAIST e IDMT) e em parte em outras duas
(A2212 e MaFaulDa) [T1], que a unidade é reconhecível pelo espectro mesmo quando não está
confundida (0,996 no MIMII e 0,868 no ToyCar, com split por gravação [T6]) e que o nível é um
atalho dentro da máquina no MIMII (mediana de 0,772 contra 0,5 [T2]). A segunda é o tacômetro
acústico validado contra tacômetro real (MaFaulDa) e contra o setpoint do inversor
(UOEMD/IM-VACD), com o protocolo correto (máquina fora) aplicado às features de ordem no MIMII e
no ToyCar.

## 2. Cada caminho, com a base que serve e a que não serve

Para o caminho 2, sobre quais distúrbios de alimentação são audíveis, nenhuma base serve como
validação. A UOEMD e a IM-VACD têm desequilíbrio de tensão e falta de fase reais, mas numa
classe que mistura os dois e que vive num motor só. O MIMII tem a anomalia "voltage change" nos
ventiladores, mas sem rótulo por arquivo. O ToyCar varia a tensão da fonte CC como condição de
operação, não como distúrbio. O IDMT produz os estados por tensão e carga, mas com classe =
motor. O caminho 2 só fica sólido com uma base própria com fonte programável. Até lá, a UOEMD e o
ToyCar servem para um estudo exploratório, declarando a confusão.

Para o caminho 4, o tacômetro acústico, a MaFaulDa serve em velocidade constante: 1951
varreduras de 12 a 62 Hz com tacômetro [T1, T5]. A UOEMD e a IM-VACD servem como segunda
verificação, contra o setpoint do inversor, mas com o escorregamento embutido no erro e com o
erro de oitava a 30 Hz na UOEMD [T5]. Nenhuma base tem rampa com verdade de rotação: as rampas da
UOEMD não têm tacômetro, e a ficha mostra que os extremos declarados não coincidem com a janela
gravada. MIMII, ToyCar e KAIST não servem.

Para o caminho 5, vários motores com um microfone, servem a UOEMD e a IM-VACD, com 8 motores do
mesmo modelo em velocidades conhecidas e, na IM-VACD, 3 celulares. Elas permitem misturas
sintéticas de dois ou três motores com a rotação de cada um conhecida pelo setpoint. O MIMII e o
ToyCar servem para misturas com ruído real, mas sem verdade de rotação. A MaFaulDa e a KAIST não
servem, por serem uma bancada só.

Para o caminho 6, a auditoria de avaliação, servem o MIMII e o ToyCar como bases em que o
protocolo correto é possível. A UOEMD, a IM-VACD, a KAIST e a MaFaulDa servem como casos de
estudo do protocolo que infla: com rótulos permutados por gravação e split aleatório, a forma
espectral dá 1,000 na KAIST, 0,598 na IM-VACD e 0,467 na UOEMD, contra acasos de 0,333 e 0,125
[T3]. A literatura dessas bases usa split aleatório ou temporal dentro do arquivo, segundo as
fichas.

Para o caminho 8, a anomalia sem rótulo com features harmônicas, servem o MIMII e o ToyCar, com
normal abundante, várias unidades e ruído real. O atalho de nível dentro da máquina no MIMII
precisa entrar como baseline obrigatório [T2]. As demais bases não servem, porque não têm normal
suficiente em mais de uma unidade.

Para as features de ordem, a MaFaulDa é a única com falhas mecânicas reais e rotação medida, mas
não tem falha elétrica. A UOEMD e a IM-VACD têm falhas mecânicas e elétricas com rotação
rastreável pelo setpoint, mas com classe = motor, então uma feature de ordem que acerte nelas
pode estar reconhecendo o motor. A KAIST mostra as linhas de pista externa e interna, segundo a
ficha, mas numa velocidade e com uma gravação por classe. MIMII e ToyCar não têm rotação.

## 3. Alguma base resolve "classe = motor"?

Para detecção de anomalia, sim: o MIMII e o ToyCar têm normal e anomalia na mesma unidade [T1].
Para diagnóstico de falha com tipo identificado, especialmente falha elétrica, nenhuma resolve.
As duas bases com falhas elétricas reais (UOEMD e IM-VACD) são diagonais, e as que resolvem a
confusão não rotulam o tipo de falha nem têm rotação. Também nenhuma base registra a sessão de
gravação de forma que permita descartar classe = sessão.

Uma base própria precisaria ter, no mínimo, o seguinte. Pelo menos quatro motores do mesmo modelo
(seis é melhor, para deixar um de fora com alguma estatística), cada um gravado em todas as
classes. As falhas precisam ser reversíveis ou trocáveis entre motores: distúrbios de tensão
pela fonte programável, que se aplicam a qualquer motor; desbalanceamento por massa;
desalinhamento por calço; e rolamentos defeituosos montados em rodízio nos vários motores, para
que cada peça defeituosa apareça em mais de um motor e cada motor tenha mais de uma peça. O
tacômetro ou encoder deve ser gravado no mesmo conversor do microfone, sincronizado, a pelo
menos 16 kHz, com velocidade constante em vários pontos e rampas lentas e rápidas. O ganho do
microfone deve ser fixo e calibrado, com um tom de referência por sessão, para que o nível seja
comparável e o atalho de nível possa ser medido e não só suspeitado. Cada classe deve ser gravada
em pelo menos duas sessões em dias diferentes, em ordem aleatória, com gravações do motor
saudável no início e no fim de cada sessão, para separar classe de sessão. O ruído de fundo deve
ser gravado à parte, sem motor, para misturas controladas, e os metadados de cada gravação
(motor, peça, tensão, carga, rotação, sessão, hora) registrados num arquivo e não só no nome.
Com fonte programável, a mesma bancada cobre os distúrbios de Lee & Shen (afundamento, elevação,
harmônicos, desequilíbrio, flicker) com verdade, que é o que o caminho 2 exige.

## 4. Riscos e o que não foi verificado

O MIMII DUE foi baixado, mas ficou fora da auditoria por ter licença não comercial, até decisão
do autor. Do MIMII original, só se usou 6 dB; 0 e −6 dB não foram baixados. Não se sabe se
normal e anomalia do MIMII foram gravados na mesma sessão, e o atalho de nível dentro da máquina
pode ser físico ou de sessão; a base não permite separar os dois. A licença da MaFaulDa não foi
encontrada. O vazamento entre rotações vizinhas da mesma varredura na MaFaulDa foi apontado na
ficha, mas não foi medido: o split por gravação da auditoria trata cada arquivo como gravação, e
um split por varredura (pasta) não foi rodado. Os espectros médios da MaFaulDa incluem os
arquivos de microfone defeituoso. O rastreador não foi testado no MIMII nem no ToyCar, por falta
de verdade de rotação, nem nas rampas da UOEMD. Que a UOEMD e a IM-VACD sejam os mesmos oito
motores físicos é provável, mas não está confirmado. No ToyCar, há trechos de ruído repetidos
entre arquivos, inclusive de carros diferentes, e os rótulos de atributos do conjunto de
avaliação discordam do áudio, segundo a ficha. Os classificadores da auditoria são simples, e
uma acurácia baixa do controle de nível não prova que não exista outro atalho. Por fim, das
referências que usam cada base, os artigos das próprias bases foram lidos inteiros, exceto o da
IM-VACD (MSSP 2026), cujo texto completo deu acesso negado e do qual só se leu o resumo; a
MaFaulDa não tem artigo de descrição oficial. As demais leituras estão registradas nas fichas como texto completo ou só resumo.
