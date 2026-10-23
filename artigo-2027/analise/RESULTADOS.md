# Resultados da auditoria das bases

Todos os números deste documento estão em `analise/tabelas_auditoria.md` (gerado por
`scripts/consolidar.py`) ou em `analise/COMPARATIVO_BASES.md` (gerado por
`scripts/comparativo.py`). Ambos citam o CSV de origem em `out/experiments/`. Entre colchetes,
a seção da tabela de onde cada número vem: [T1] inventário, [T2] controle de nível,
[T3] rótulos permutados, [T4] espectros, [T5] rastreador, [T6] separabilidade por unidade, [T7] matrizes classe × unidade. As
acurácias são balanceadas, e o acaso é 1 sobre o número de classes. Nada aqui é resultado de
artigo; é diagnóstico das bases.

## Classe confundida com unidade e com sessão

Das seis bases auditadas, três repetem o defeito do IDMT por inteiro. Na UOEMD e na IM-VACD,
cada uma das 8 classes foi gravada num motor diferente, e nenhum motor aparece em mais de uma
classe [T1, T7]. As fichas confirmam isso pelo artigo da própria UOEMD ("only one motor having
each type of fault"). Na KAIST, as 5 condições são 5 gravações de 60 s, uma por rolamento
montado, feitas numa única tarde em sessões separadas [T1]. Nessas três bases, deixar uma
unidade de fora tira a classe inteira do treino. Por isso o controle de nível dá 0,000 no
split por unidade na UOEMD e na IM-VACD, contra acaso de 0,125 [T2]: o classificador nunca viu
a classe que precisa prever. A MaFaulDa é uma bancada única. Nela, os defeitos de rolamento
são uma peça por tipo, de modo que tipo de defeito e peça coincidem, e só 4 das 11 unidades
definidas na auditoria têm mais de uma classe [T1]. A ficha registra ainda que o normal e o
desbalanceamento leve foram gravados num dia e o desbalanceamento pesado e os desalinhamentos
em outro.

As duas bases de detecção de anomalia são as únicas em que toda unidade tem as duas classes. No
MIMII original, auditado com 4 ventiladores e 4 bombas a 6 dB, todas as 8 máquinas têm normal e
anômalo. No ToyCar, os 6 carrinhos do conjunto de desenvolvimento têm as duas classes, e os 6
carrinhos do treino adicional têm só normal [T1]. Nenhuma das duas, porém, registra a data ou a
sessão das gravações, e por isso a confusão entre classe e sessão não pôde ser descartada nelas.

## O atalho de nível

Com todas as unidades juntas, RMS e pico sozinhos separam pouco as classes na UOEMD (0,180
contra acaso de 0,125, split por gravação) e nada na IM-VACD (0,117 contra 0,125) [T2]. Em
bases diagonais, isso só diz que os motores não diferem muito em volume, porque classe e motor
são a mesma coisa. O quadro muda quando o classificador é treinado dentro de cada unidade. Nas
8 máquinas do MIMII, os dois números de nível dão uma acurácia mediana de 0,772 (de 0,572 a
0,856) contra acaso de 0,5, com split por gravação [T2], embora fiquem no acaso (0,518) com
todas as máquinas juntas. Isso concorda com a ficha, que mede AUC de 0,91 e 0,94 só com o RMS
em duas máquinas: a anomalia é, em geral, mais alta que o normal da mesma máquina, e um detector
treinado por máquina ganha esse atalho de graça. No ToyCar o mesmo teste dá 0,511 (de 0,444 a
0,591), sem atalho dentro da unidade [T2]. Com todos os carrinhos juntos, porém, o nível dá
0,642, e esse valor é o mesmo nos três splits. Na MaFaulDa, mesmo sem os arquivos de microfone
defeituoso, o nível dá 0,430 contra acaso de 0,100 com split por gravação [T2], em parte porque
os rolamentos na posição underhang são mais silenciosos que o normal, como registra a ficha.
Para comparação, no IDMT o nível dá 0,991 contra 0,333, e no A2212 0,562 contra 0,333, no
protocolo do trabalho do ICIT [T2].

## Rótulos permutados: a gravação é reconhecível

A permutação por segmento deu o acaso em todas as bases e em todos os splits [T3], o que valida
o código. A permutação por gravação, em que cada gravação recebe o rótulo de outra, separa as
bases em dois grupos. Com a forma espectral e split aleatório por segmento, a KAIST dá 1,000
contra acaso de 0,333, a IM-VACD 0,598 contra 0,125, a UOEMD 0,467 contra 0,125 e a MaFaulDa
0,166 contra 0,100. Nos splits agrupados, as quatro voltam ao acaso [T3]. Nessas bases, as
features reconhecem a gravação de onde o segmento veio, e qualquer número obtido com split
aleatório mede em parte essa identificação. No MIMII (0,514 contra 0,500) e no ToyCar (0,530
contra 0,500), o efeito é pequeno [T3], coerente com arquivos curtos e independentes de 10 s.

## Espectro: unidade pesa mais que classe

Nas bases em que a comparação é possível, a diferença de forma espectral entre unidades da
mesma classe é maior que a diferença entre classes da mesma unidade. No MIMII, as medianas são
3,15 dB entre máquinas e 1,54 dB entre normal e anômalo da mesma máquina. No ToyCar, 2,39 dB
contra 1,06 dB. Na MaFaulDa, 0,71 dB contra 0,55 dB [T4]. Na UOEMD e na IM-VACD, nenhum dos dois
tipos de par existe, porque a matriz é diagonal, e a comparação é impossível por construção
[T4]. As figuras `out/figuras/exp3_espectro_por_unidade_<base>.png` mostram o mesmo efeito: as
curvas de normal e anômalo de uma máquina do MIMII quase se sobrepõem, enquanto as máquinas
diferem entre si nos picos tonais. Na MaFaulDa, os espectros incluem os arquivos de microfone
defeituoso, que aparecem como curvas planas em −45 dB no painel de desbalanceamento, e os picos
em múltiplos de 60 Hz indicam zumbido da rede no canal do microfone.

O experimento 5 confirma isso com um classificador. Treinada para prever a unidade com split
por gravação, a forma espectral acerta 0,996 das 8 máquinas do MIMII (acaso 0,125) e 0,868 dos
12 carrinhos do ToyCar (acaso 0,083). Na UOEMD acerta 0,447 dos motores, na IM-VACD 0,439 e na
MaFaulDa 0,396 das unidades (acaso 0,091) [T6]. No MIMII, a identidade da máquina é praticamente
perfeita mesmo dentro de cada classe: 1,000 no normal e 0,981 no anômalo [T6]. Isso não invalida
o MIMII, porque nele cada máquina tem as duas classes. Mas impede comparar métodos treinados com
todas as máquinas sem testar uma máquina de fora.

## Rastreador de rotação

O rastreador do trabalho do ICIT foi testado com faixas de busca fixadas antes da execução. Na
MaFaulDa, com uma faixa única de 10,9 a 68,4 Hz para a base inteira e referência no tacômetro,
ele ficou a até 2 % da rotação medida em 0,967 dos 1951 arquivos, com erro relativo mediano
abaixo de 0,001. Sem os 166 arquivos de microfone defeituoso, a fração sobe para 0,999 dos 1785
restantes. Nos defeituosos, cai para 0,627 [T5]. É a primeira validação do rastreador contra uma
rotação medida, e ela não usa nenhuma informação do arquivo rastreado.

Na UOEMD e na IM-VACD, a referência é a rotação síncrona dada pelo setpoint do inversor (motor
de 2 polos), e o erro inclui o escorregamento. Na IM-VACD, o erro mediano foi de 0,024 a 15 Hz e
de 0,012 a 25 e 30 Hz, com relação mediana f_alim/f_rot de 1,025 e 1,012 [T5]. Esses valores são
compatíveis com o escorregamento estimado na ficha `docs/bases/imvacd.md` pelo 2× do áudio
(cerca de 2,3 % a 15 Hz e 1,1 % a 30 Hz), e por isso a fração dentro de
2 % do síncrono (0,566 no total) subestima o rastreador. Na UOEMD, o rastreador errou a oitava a
30 Hz: a relação mediana foi 2,018, e nenhum arquivo ficou dentro de 2 %. Como a faixa fixada
inclui f_alim/2, que explica os mesmos harmônicos, o rastreador travou na metade da rotação.
Contando as oitavas como acerto, num diagnóstico feito a posteriori, a fração na UOEMD sobe de
0,469 para 0,938 [T5]. A regra fixada antes não foi alterada depois desse resultado.

## Síntese

Nenhuma base reúne as cinco condições procuradas. As de detecção de anomalia (MIMII, ToyCar)
resolvem "classe = unidade", mas não têm rotação, nem tipo de falha por arquivo, nem falha
elétrica controlada, e a máquina é reconhecível quase perfeitamente pelo espectro. As de Ottawa
(UOEMD, IM-VACD) têm as falhas elétricas e de tensão e a frequência do inversor, mas classe =
motor em todas as classes. A MaFaulDa tem a verdade de rotação de melhor qualidade e validou o
rastreador, mas é uma bancada única, sem falha elétrica, com classe parcialmente confundida com
a sessão e um canal de microfone defeituoso em 166 arquivos. A KAIST não tem áudio suficiente
para nenhum uso além de controle negativo. As consequências para cada caminho de artigo estão
em `analise/RECOMENDACAO.md`.
