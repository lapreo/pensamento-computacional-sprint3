# Controle Inteligente de Sessão de Recarga — Sprint 03

Protótipo em **Raspberry Pi Pico + MicroPython** (simulado no Wokwi) que decide se uma
sessão de recarga é autorizada, reduzida ou bloqueada, comparando geração e consumo de
energia — inspirado no conceito de GoodWe Smart Energy Controller.

Este enunciado é o mesmo para **Pensamento Computacional e Automação com Python** e
**Solução em Energias Renováveis** (confirmado pelo grupo). Um único projeto atende às
duas entregas.

## Como rodar no Wokwi

1. Em [wokwi.com](https://wokwi.com), crie um novo projeto **Raspberry Pi Pico (MicroPython)**.
2. Substitua o `diagram.json` gerado automaticamente pelo `diagram.json` deste projeto
   (aba "diagram.json" no editor — cole o conteúdo).
3. Cole o conteúdo de `main.py` (versão interativa, com potenciômetros) OU de
   `demo_cenarios.py` (versão com os 3 cenários fixos do enunciado — **recomendada para
   gravar o vídeo**, já que garante os números exatos do enunciado sem precisar acertar
   o potenciômetro no momento certo) no arquivo `main.py` do projeto Wokwi. Copie também
   `logica.py` como um segundo arquivo no projeto (Wokwi permite múltiplos arquivos —
   botão "+" ao lado das abas).
4. Clique em ▶ (Play). O Monitor Serial (ícone de terminal) mostra a saída de texto.

Se o `env` do `diagram.json` (`micropython-20231227-v1.22.0`) não for aceito pela versão
atual do Wokwi, criem o projeto pela interface (que já define a versão correta
automaticamente) e apenas ajustem a fiação manualmente com base na tabela abaixo — a
lógica em `logica.py`/`main.py` não muda.

## Ligações

| Componente | Pino do Pico |
|---|---|
| LED verde (com resistor 220 Ω) | GP16 |
| LED amarelo (com resistor 220 Ω) | GP17 |
| LED vermelho (com resistor 220 Ω) | GP18 |
| Potenciômetro GERAÇÃO (pino SIG) | GP26 (ADC0) |
| Potenciômetro CONSUMO (pino SIG) | GP27 (ADC1) |

## Como rodar os testes (fora do Pico, em qualquer computador com Python)

```bash
python3 -m pytest -q   # 8 testes, incluem os 3 cenários exatos do enunciado
```

`logica.py` não importa `machine` nem nada específico do MicroPython — por isso os
testes rodam em Python comum, sem precisar do Pico nem do Wokwi.

## Threshold assumido (leiam antes de gravar o vídeo)

O enunciado dá três exemplos, mas não diz qual é o valor exato que separa "reduzida" de
"autorizada". Adotamos **2000 W** como a potência nominal de recarga (`logica.py`,
constante `POTENCIA_NOMINAL_RECARGA_W`): abaixo disso (mas acima de zero) é reduzida,
igual ou acima é autorizada. Os três exemplos do enunciado batem com esse valor (2500 W
autorizada, 300 W reduzida, -800 W bloqueada), mas qualquer número entre 301 W e 2500 W
funcionaria igualmente bem — se o professor tiver passado um valor oficial diferente,
troquem só essa constante.

## Como o Raspberry Pi Pico processa as informações (para explicar no vídeo)

- **Entrada (E/S):** os potenciômetros geram uma tensão proporcional à posição do cursor;
  o **ADC** do Pico (conversor analógico-digital, 12 bits reais, apresentado como 16 bits
  em `read_u16()`) converte essa tensão em um número de 0 a 65535, que o código escala
  para Watts.
- **Processamento:** o RP2040 (o chip do Pico) é um processador de 32 bits com dois
  núcleos ARM Cortex-M0+ (a simulação do Wokwi usa só um). A cada ciclo, ele executa a
  subtração `geração − consumo` e as comparações de `decidir_estado`, tudo em código de
  máquina interpretado pelo MicroPython.
- **Memória:** o programa (`main.py`, `logica.py`) fica gravado na memória Flash do Pico
  (2 MB); as variáveis (`geracao_w`, `disponivel_w` etc.) ficam na SRAM (264 KB) durante a
  execução e são perdidas ao desligar.
- **Saída (E/S):** os pinos GP16/17/18 são configurados como saída digital
  (`Pin.OUT`) e cada um liga um LED (nível alto = LED aceso); o `print()` envia texto pela
  porta serial USB, lido pelo Monitor Serial do computador.
- **Representação de dados:** o item 5 do enunciado (decimal/binário/hexadecimal) está em
  `representar()`, em `logica.py`. Quando a energia disponível é negativa (situação de
  bloqueio), ela é mostrada em **complemento de dois** de 16 bits — a forma padrão como
  processadores como o RP2040 representam números negativos em hardware, sem precisar de
  um bit "extra" só para o sinal.

## Onde está cada item do enunciado

| Item do enunciado | Onde |
|---|---|
| Energia disponível = Geração − Consumo | `logica.py`, `calcular_disponivel` |
| 3 situações (autorizada/reduzida/bloqueada) + LEDs | `logica.py` (`decidir_estado`) + `main.py`/`demo_cenarios.py` (`acender_led`) |
| Apresentação no Monitor Serial | `formatar_linha`, usado em `rodar_ciclo`/`rodar_cenario` |
| Representação decimal/binário/hexadecimal | `logica.py`, `representar` |
| Explicação do processamento pelo Pico | seção acima deste README |

## O que ainda falta (depende do grupo)

1. **Rodar de verdade no Wokwi** (ou no hardware físico) e conferir se o `diagram.json`
   carrega sem erro — escrevi e validei a sintaxe e os nomes de pino contra a
   documentação oficial, mas não tenho como abrir o simulador daqui para confirmar.
2. **Gravar o vídeo de até 5 minutos** mostrando: o protótipo, a ligação dos LEDs, as três
   situações (`demo_cenarios.py` facilita isso), os dados no Monitor Serial e a explicação
   da relação com Arquitetura de Computadores (usem a seção acima como roteiro).
3. **Confirmar o threshold** de 2000 W com o professor, se houver um valor oficial.
4. **Subir tudo num repositório GitHub** e preencher `ENTREGA.txt` com os links do
   repositório e do vídeo — é o único arquivo que vai para o portal.
