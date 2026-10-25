# Bloqueios

Registro de tudo que travou, com a evidência. Atualizado pela sessão principal.

## B1 · Nenhum ESP32 conectado (23/10/2026, 22h)

`Get-PnpDevice -PresentOnly -Class Ports` lista só `Porta de comunicação (COM1)`
(`ACPI\PNP0501\0`, a serial da placa-mãe). Não há CP210x, CH340 nem USB-JTAG da Espressif.
Sem o chip, as etapas 4.4 (equivalência no ESP32) e 5 (medições no ESP32 físico) não rodam.

**Atualização 23/10, 22h40:** o Erick conecta o chip até sábado 24/10. Até lá, firmware e
scripts de medição ficam prontos e testados só no build.

## B2 · ESP-IDF não instalado (23/10/2026, 22h)

**Atualização 23/10, 22h40:** autorizado o download. Procurado em `Artigo3(MotorSound)`
(fora de `.venv`, `.git`, `data`) e em `C:\Users\erick` até profundidade 4: não havia instalação.
Instalando ESP-IDF v5.5.6 (última tag estável 5.x em `git ls-remote`) em
`C:\Espressif\esp-idf-v5.5.6`, fora do repositório.

**Resolvido 24/10, ~00h:** `install.bat esp32,esp32s3` baixou as ferramentas, mas falhou ao
criar o ambiente Python ("This script was called from a virtual environment") porque o
`python` do PATH era o `.venv` do projeto. Refeito com
`idf_tools.py install-python-env` usando o CPython 3.12.13 do uv (fora de venv). O ambiente
é carregado por `firmware/idf_env.bat`. Validação: `examples/get-started/hello_world`
copiado para `C:\Espressif\hello_test`, `idf.py set-target esp32s3 && idf.py build && idf.py size`
→ "ESP-IDF v5.5.6", "Project build complete", exit 0.


`idf.py`, `esptool.py` e `xtensa-esp32*-gcc` não estão no PATH; não existem `C:\Espressif`,
`~/esp` nem `~/.espressif`. Instalar o IDF é um download de toolchain que não é base nem
pacote Python (CLAUDE.md do Artigo3, "Pare e pergunte"), então depende de autorização.
Sem o IDF não há `idf.py size` (categoria 3 do CLAUDE.md de `embarcados/`).

## B3 · Estrutura diferente da prevista (23/10/2026, 22h)

O prompt e o `CLAUDE.md` falam de `Artigo3(MotorSound)/embarcados/`, que não existia. O porte
do colaborador está no commit `e00f2a9` em `Embarcado/decifrae_core.{c,h}`; na árvore de
trabalho essa pasta (e `artigo-2027/`) foi apagada e reaparece, sem commit, em
`artigo-ICIT/Proposta_de_Artigo/Embarcado/`, junto com este `CLAUDE.md`. Os arquivos do
colaborador foram **copiados** (idênticos ao commit, conferido com `diff --strip-trailing-cr`)
para `embarcados/src/c/colaborador_original/`. A mudança de lugar do `artigo-2027/` não foi
tocada nem commitada.

## B4 · Sem distribuição TeX (24/10/2026, 9h30)

`pdflatex`, `latexmk`, `xelatex`, `tectonic`, `bibtex` não estão no PATH; não existem
`C:\Program Files\MiKTeX` nem `C:\texlive`. O LaTeX IEEEtran de `paper/` é escrito mas não
compilado aqui; a contagem de páginas (6 pp regular / 4 pp WiP) não pode ser verificada sem
compilar. Saídas: compilar no Overleaf, ou autorizar a instalação de uma distribuição TeX
(download fora de bases e pacotes Python).

**Resolvido 24/10, ~9h45:** o Erick autorizou. MiKTeX 25.12 instalado com
`winget install --id MiKTeX.MiKTeX --scope user --silent` em
`%LOCALAPPDATA%\Programs\MiKTeX\miktex\bin\x64` (não está no PATH global; scripts acrescentam),
com `initexmf --set-config-value "[MPM]AutoInstall=1"`. Teste: documento
`\documentclass[conference]{IEEEtran}` com booktabs/graphicx/amsmath → "Output written on
t.pdf (1 page)".

## B1 · resolvido (24/10)

O Erick conectou um ESP32 na COM3 (conversor CH9102). Lido do chip: ESP32 clássico rev. 3.1,
240 MHz, flash de 4 MB, **sem PSRAM** (`results/manifests/bancada.json`). Ele só tem essa placa.

## B5 · ESP32 sem PSRAM: a referência não cabe (24/10)

O workspace contíguo da `p11_c_ref` é maior que qualquer bloco livre do heap
(`results/csv/esp32_memoria_esp32_20261024-182937.csv`). O Erick autorizou uma variante nomeada:
`p11_c_lowmem` (`src/c/p11_lowmem.*`). Problemas encontrados no chip e corrigidos, um por commit:
fragmentação do pool (melhor encaixe, pedidos do maior ao menor, depois plano conjunto do init e
da fase A por busca com retrocesso); IRAM não aceita `lsi`/`ssi` (LoadStoreError em
`p11_preparar`), então só as tabelas de giro, lidas por palavra inteira, vão para a IRAM; reserva
de heap 12 → 7 KB. Resultado: cabe em janelas de 1 s; a janela de 3 s do IDMT não cabe nesta placa.
Medir o IDMT no chip exige uma placa com PSRAM.

## B6 · Conexão USB instável e reset sem GPIO0 (24–25/10)

O reset automático da placa puxa o EN mas não o GPIO0 (`Wrong boot mode 0x13`), e a porta caiu
várias vezes durante a gravação ("Acesso negado", "chip stopped responding", COM3 sumindo). Saída:
o Erick segura o BOOT enquanto `firmware/gravar_boot_espera.py` pulsa o EN pela RTS e grava só o
app comprimido; a 921600 baud a serial perdia bytes, resolvido com confirmação "@@A" a cada 1 KB.

## Não é bloqueio, mas registro

ICE esporádico do GCC (24/10, ~10h): o primeiro build `esp32s3 ref` depois de adicionar a
ESP-DSP 1.5.2 caiu com "internal compiler error: Segmentation fault" em
`managed_components/espressif__esp-dsp/modules/conv/float/dspi_conv_f32_ansi.c:184`; o mesmo
arquivo compilou nos outros três builds e a repetição do `esp32s3 ref` saiu "FW OK". Tratado
como falha transitória do compilador (xtensa-esp-elf 14.2.0_20260121); se reaparecer, investigar.


Compilador no PC: não há gcc/clang no PATH, mas o Visual Studio 2022 Build Tools está
instalado (MSVC 14.44.35207, CMake 3.31.6-msvc6, Windows SDK 10.0.26100.0) e compila.
