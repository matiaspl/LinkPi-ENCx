# Hardware models and pinouts

## Models and power
| Model | CPU |
|---|---|
| MINI	| HI3516CV610	|
| ENC1,ENC1V2, TBS 2603SE |	HI3520DV400	|
| ENCSH	| HI3521DV100	|
| ENC2, ENC5, ENC9 | 	HI3531DV100	|
| ENC5, ENC9	| HI3531DV100	|
| ENC2V2, ENC4 |	HI3531DV200 |
| ENC1V3, ENCS1, ENC4S, ENCSHV2, REC1, VGA1, UVC2	| SS524V100 |
| ENC2V2, ENC2V3, ENC4, ENC4SP, ENC5V2, ENC8	| SS528V100	|
| DEC2	| SS626V100	|
| ENC1Pro, ENCS1Pro	| SS928V100 |


* the encoders can be powered with 5V using a powerbank or a USB power and a USB-A to DC 5.5x2.5mm barrel connector cable, provided they can deliver ~2 amps. (Carl Mills@EnDeCo)
* there's no power regulator between USB-C and regular USB on ENC1V2, so if you provide 9/12V over USB-C tthe device will boot up, but the same voltage will most likely fry whatever's connected to the other USB ports (Carl Mills@EnDeCo)

**UPDATE 8.04.2022**: by the looks of it TinyENC1 seems to actually have a bit different firmware with no telnet/ssh access enabled by default. If anyone's interested in getting inside those boxes consider donating one or sending me the recovery firmware.

**UPDATE 9.11.2023**: The most recent boxes use a new more powerful CPU, the SS5248V100. From what I read this is a drop-in replacement for Hi35xx chips, namely the Hi3520), more RAM and even larger flash storage.

## F3520D pinout

* J1 - USB 0, J4 - USB 1

| 1 | 2 | 3 | 4 |
|---|---|---|---|
| +5V | USB- | USB+ | GND |

* P5 - VGA

| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|----|
| +5V | NC | NC | VSync | HSync | GND | Red | Green | Blue | GND |

* J16 - UART 0

| 1  | 2  | 3   |
|----|----|-----|
| TX | GND | RX |
