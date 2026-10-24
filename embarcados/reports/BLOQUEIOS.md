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

## Não é bloqueio, mas registro

Compilador no PC: não há gcc/clang no PATH, mas o Visual Studio 2022 Build Tools está
instalado (MSVC 14.44.35207, CMake 3.31.6-msvc6, Windows SDK 10.0.26100.0) e compila.
