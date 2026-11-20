# uk-sc-integration

Contrato e implementação do link entre o **Raspberry Pi Zero 2W** (que controla as telas e
o driver de vídeo) e o **Arduino** (que varre a malha de 112 teclas).

Este repositório é a **fonte de verdade do protocolo serial** e contém as **duas pontas**:
a biblioteca portável em C que roda no Pi e o firmware que roda no AVR.

## O que faz

- Define o frame: `SOF | VER | TYPE | LEN | PAYLOAD | CRC16`.
- Implementa de-framing com resync, validação de CRC e timeout intra-frame.
- Implementa o firmware AVR: varredura 8×14, debounce por timestamp, diodos, eventos por
  transição.
- Publica heartbeat e trata a perda de link liberando todas as teclas.

## Construir

```bash
# núcleo portável + testes host
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build

# firmware
arduino-cli compile --fqbn arduino:avr:uno --warnings all --output-dir firmware/build firmware/
```

## Testar

```bash
ctest --test-dir build --output-on-failure
```

Casos obrigatórios do parser: frame válido, CRC inválido, truncado, `LEN` máximo, `SOF`
dentro do payload, ruído antes do frame, timeout intra-frame.

## Gravar no hardware

```bash
arduino-cli upload --fqbn arduino:avr:uno --port /dev/ttyACM0 firmware/
```

Upload é passo de bancada, nunca de CI.

## Fronteira

- **Não** desenha nada na tela.
- **Não** conhece HID nem o host.
- Mudança no frame exige ADR em `../docs/adr/` e bump de versão de protocolo.

## Estado

Fase 0 — definição de arquitetura. Decisões pendentes em
[`../docs/context-map.md`](../docs/context-map.md) §12.
