---
title: "Misubishi encoders"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Misubishi encoders

Mitsubishi servos such HA-FF usually have OBA17-051 or OBA17-052 encoders.
STMBL has support for the `mit 02-4` and `mit 02-2` protocols.

## Electrical connection

The encoders are connected using a half-duplex UART (MR, MRR). Some older encoders only support a full-duplex UART (MR, MRR, MD, MDR).
To activate the full-duplex UART, set Pin `full_duplex` to 1.

| FB0 Pin | Mitsubishi Name |
|---------|-----------------|
| 1       | (MD) |
| 2       | (MDR) |
| 3       | 
| 4       | MRR |
| 5       | MR  |
| 6       | 
| 7       | P5  |
| 8       | LG  |

## Config

If your encoder ID is listed in the table below it will work by just linking the config template:
```python
link encm_fb0
```
If the encoder ID is not listed it probably will also work out of the box, because the default request `0x32` with 17 bit resolution should work for most encoders.
To check the position feedback, you can plot it on the Servoterm oscilloscope:
```python
term0.wave0 = encm0.pos
```
If your encoder doesn't work or does not produce a position feedback in the range of +- Pi, you can set the request and resolution manually:
```python
encm0.cmd = 50
encm0.res = 18
```
If you find a unknown encoder ID, please report it in an issue with the `cmd` and `res` settings.

### Known encoder IDs

| ID (hex) | ID (dec) | supported resolution | multi turn | resolution according to Mitsubishi |
|----------|----------|----------------------|------------|------------------------------------|
| 20 | 32 | 13 bit |   |        |
| 3D | 61 | 17 bit | x |        |
| 41 | 65 | 17 bit | x |        |
| 44 | 68 | 18 bit | x | 22 bit |

## Protocol description

The protocol uses a single byte request message, followed by a response from the encoder.

| Request | Description |
|---------|-------------|
| 0x02    | returns single turn data |
| 0x32    | returns single and multi turn data |
| 0x2A    | returns single and multi turn data |
| 0xA2    | returns single and multi turn data |
| 0x8A    | returns multi turn data |
| 0x92    | returns the encoder ID |
| 0x7A    | returns the encoder and motor ID |
| 0xBA    | clears alarms and returns single turn data |

The single turn data bits are returned at different significances, presumably to allow backwards compatibility with lower resolutions.

### Alarm and Status codes

| Alarm | Name                          | Checked at    | Details                       | Latched | Status code |
|-------|-------------------------------|---------------|-------------------------------|---------|-------------|
| A0    | CPU Alarm                     | power on      | Encoder internal data damaged | X       | ERROR       |
| A1    | unknown/not used              |               |                               |         |             |
| A2    | Data Alarm                    | each request  | Data per revolution error     |         | ERROR       |
| A3    | Encoder temperature Alarm     | 10 sec avg    | Encoder section hot (100°C)   |         | ERROR       |
| A4    | Encoder temperature Warning   | 30 min avg    | Encoder section hot (85°C)    |         | WARN        |
| A5    | Multi-turn Alarm              | each request  | Multi-turn count data error   |         | ERROR       |
| A6    | ABS Lost                      | power on      | Multi-turn backup data lost   | X       | ERROR       |
| A7    | Battery disconnected          | 10 sec avg    | Battery disconnected or low   |         | WARN        |

### Request 0x02
![](../../../images/mitsu_0x02.svg)
### Request 0x32
![](../../../images/mitsu_0x32.svg)
### Request 0x2A
![](../../../images/mitsu_0x2A.svg)
### Request 0xA2
![](../../../images/mitsu_0xA2.svg)
### Request 0x8A
![](../../../images/mitsu_0x8A.svg)
### Request 0x92
![](../../../images/mitsu_0x92.svg)
### Request 0x7A
![](../../../images/mitsu_0x7A.svg)
### Request 0xBA
![](../../../images/mitsu_0xBA.svg)

## Sources
https://web.archive.org/web/20170517130336/http://www.deltatau.com/manuals/pdfs/Acc-84E.pdf  
https://www.cnczone.com/forums/servo-motors-drives/98500-mitsubishi-ac-servo-encoder-outputs.html  
https://www.cnczone.com/forums/servo-motors-drives/316816-mitsubishi-ac-servo-encoder-protocol.html  
http://yurtaev.com/mr-j-faq.html  