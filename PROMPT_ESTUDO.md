# Prompt para o chat de estudo

> Cole no início de um chat novo e anexe o arquivo `CONTEXTO_ESTUDO_MOTORSOUND.md`.
> Se quiser, anexe também `PLANO_PIPELINE_MOTOR.md` e o `motor_acoustic.zip`.

---

Amanhã tenho uma reunião com meu orientador sobre um artigo novo. O documento anexado tem
todo o contexto: a situação, o que o orientador pediu, as pipelines antigas do meu projeto
(Decifrae), a pipeline nova, a teoria e os artigos. Leia ele inteiro antes de responder.

Quero **entender a fundo** o que estamos fazendo para conseguir conduzir a discussão, não só
apresentar. Me ensine passo a passo, de forma visual e detalhada, seguindo esta ordem:

1. **O problema físico:** o som de um motor como série harmônica, por que ele parece uma
   corda de baixo, e como as três classes do IDMT (good, broken, heavyload) aparecem no som.
2. **CQT ou STFT:** pente harmônico linear, escala logarítmica, chroma dobrando oitavas,
   resolução Δf = fs/N, janela longa em sinal estacionário, interpolação parabólica.
3. **Beat tracking → janelas fixas + HMM:** o papel que a batida tinha no violão e quem assume
   esse papel agora; compromisso entre latência e estabilidade.
4. **A pipeline nova, etapa por etapa:** o que entra, o que sai, de onde veio (baixo ou
   violão) e por quê.
5. **Teoria:** soma harmônica e erro de sub-oitava; YIN/pYIN; HMM, Viterbi e filtro forward;
   espectro de modulação; vazamento e avaliação honesta; restrições do ESP32.
6. **Os artigos:** que afirmação do meu argumento cada um sustenta.
7. **O código base:** o que cada módulo faz, ligado às etapas do passo 4.

Regras:

- **Um passo por vez.** No fim de cada passo, me faça 2 ou 3 perguntas para checar se entendi
  e espere minha resposta antes de seguir. Se eu errar, explique de novo por outro ângulo.
- **Seja visual.** Use diagramas, gráficos e desenhos sempre que ajudarem (o pente harmônico
  na STFT contra a CQT, a grade tempo-frequência, a treliça do Viterbi, o fluxo da pipeline,
  a curva de atraso × janela). Use exemplos numéricos com os valores do documento.
- **Ligue sempre ao que eu já conheço**: o baixo (pYIN, Viterbi por corda, erro de oitava,
  cross-talk) e o violão (chroma, cosseno, Viterbi de acordes, beat tracking).
- **Não invente resultado.** O documento marca o que é hipótese e o que ainda não foi medido;
  mantenha essa distinção. Se algo não estiver no documento e você não tiver certeza, diga.
- **Ao final do passo 7**, me ajude a montar:
  - um roteiro de uma página para a reunião, na ordem em que o orientador pediu
    (artigos, soluções dos problemas, mudanças na pipeline, código base, teoria);
  - as respostas curtas para as decisões da Parte 8 do documento, com o meu raciocínio;
  - as 5 perguntas mais difíceis que o orientador pode fazer, e como responder.

Pode começar pelo passo 1.
