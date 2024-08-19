# FIT_PM_FW_uPC

## Firmware timestamp & version
The build timestamp (32-bit) is located at the very end of the application code (e.g.: 0x2b7c), not far from the " Flash TImestamp:" string.  
The timestamp can be easily translated to firmware version YMD.Hm using the following rule:  
- Y: years, counting from 2020, so 2021 is 1,
- M: month
- D: day
- H: hour
- m: minutes  
All the values are coded using the following alphabet:  
> 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvw

---
## Firmware
### Boot

#### FPGA startup
FPGA status is stored at 0x215b with starting value equal zero.
1. If 5V is present (DA21_RST_N), then enable (DA11_EN->DA7_RUN, DA4_EN, DA13_EN) 1x LM21215AMHX-1, 2x LM21212MH-1, 1x LD49150PT10R, 1x LD49150PT12R, delay 2s.
2. Check whether .. ok, ...
3. Setup clock generator & FPGA:
    1. Send 0x00001808 (Register 8: Status/Control -> bit7 set [Normal Mode], bit8 set [exit Synchronization State]) to CDCE62005RGZT
    2. Set FPGA init pin (FPGA_INIT_B), clear then set FPGA program pin (FPGA_PROGRAM_B), delay 2,5s.	
4. Wait for FPGA done pin (FPGA_DONE_B) then:
    1. Set 0x2157 4-th bit
    2. Set FPGA reg 0x80 to 0xAF -> EEPROM values (0x21cf - 0x222e)
5. If xmega_PB7 is set (FPGA_RST) -> proceed setting FPGA regs:
    1. 0xF7 -> MCU timestamp
    2. 0x7C -> Hardcoded value: 0x0000
    3. 0x00 -> EEPROM value (0x222f)
    4. 0x01 to 0x0C -> EEPROM values (0x21b7 - 0x22ce)
    5. 0x25 to 0x3C -> EEPROM values (0x2187 - 0x21b6)
    6. 0x3D -> EEPROM value (0x2230)
    7. 0xBC -> MEM value (0x2160)
    8. 0xBD -> EEPROM value (0x2132)
    9. 0x7C -> Hardcoded value: 0xFFFF


---
## Hardware 
Some details about the peripherals the MCU ATxmega128a3 is communicating with.

| PIN 	| LABEL 	| PORT	| MODE			| PERIPHERAL 	|
|:---:	|:---:  	|:---:	| :---:			|:---:        	
|63  	|DA27_SCLK	|PA1 	| SPI CLK		|**ADT7311WTRZ Temperature Sensor**| 
|64	|DA27_DOUT	|PA2 	| SPI MISO		|		| 
|1	|DA27_DIN	|PA3  	| SPI MOSI		| 		|
|2	|DA27_CS	|PA4	| SPI CS		| 		|
|62	|		|PA0 	| OUT LED VD8 (red)	|**LEDS**	|
|3	|		|PA5 	| OUT LED VD12 (blue)	|		|
|4	|		|PA6 	| OUT LED VD13 (red)	|		|
|5	|		|PA7 	| OUT LED VD14 (green)	|		|
|6	|HCLK		|PB0	| OUT			|**THS788PFD (x3)**|
|7	|HDATA		|PB1	| IN/OUT		|		|
|8	|HSTROBE1	|PB2	| OUT CS		|		|
|9	|HSTROBE2	|PB3	| OUT CS		|		|
|10	|HSTROBE3	|PB4	| OUT CS		|		|		
|11	|THS788_RESET	|PB5	| OUT RST		|		|
|12	|THS788_OT_ALARM|PB6	| INT0 (overtemperature alarm)	|	|
|16	|SYNC_A		|PC0	| ???			|**  **|
|17	|SYNC_B		|PC1	| ???			|		|
|18	|SYNC_C		|PC2	| ???			| 		|
|19	|DD21_PWRDNB	|PC3	| ???			| **CDCF5801ADBQ(multiple)**|
|21 	|DA22_DIN	|PC5 	| SPI MOSI		| **CDCE62005RGZT & DAC8554IPW(multiple)**|
|22 	|DD3_SPI_MISO	|PC6 	| SPI MISO 		| **CDCE62005RGZT**|
|23 	|DD3_SPI_CLK	|PC7 	| SPI SCK		|		|
|50	|DD3_SPI_LE	|PF4	| SPI CS 		| 		|
|51	|DD3_SYNC	|PF5	| OUT 			|		|
|54	|DD3_PLL_LOCK	|PF6	| IN			|		|
|36 	|Xmega_PE0	|PE0 	| ??? 			| **FPGA**	|
|26 	|Xmega_PD0	|PD0 	| ??? 			|		|
|27 	|Xmega_PD1	|PD1 	| OUT (PROGRAM_B_0) 	|		|
|28	|Xmega_PD2	|PD2 	| ??? (INIT_B_0)	|		|
|29	|DD13D_AB	|PD3	| ??? (DONE_0)		|		|
|20	|DD18_60_12	|PC4	| ???			| 		|
|9	|Xmega_PB7	|PB7	| ???			|		|
|30	|DD28A_OUT	|PD4	| IN ??? 		| **HDMI13**	|
|31	|DD28C_OUT	|PD5 	| IN ???		| **HDMI16**	|
|32	|NC		|PD6	| OUT ???		| **HDMI18**	|
|33	|DD28B_OUT	|PD7	| IN ???		| **HDMI15**	|
|37	|DA21_RST_N	|PE1	| INT (nrst)		|**LTC2906ITS8**|
|38	|DD11A_A	|PE2	| OUT (enable)		|**Voltage enable**	|
|39	|DA7_PGOOD	|PE3	| IN  (outa & outb)	|**Voltage monitor** 	|
|40	|DD9_S  	|PE4	| SPI CS		|**N25Q032A11EF840 Serial Flash Memory**| 
|41	|DD9_DQ0	|PE5	| SPI MOSI		|		|
|42	|DD9_DQ1	|PE6	| SPI MISO		|		|
|43	|DD9_C		|PE7	| SPI CLK		|		|
|46	|DD7_T2IN	|PF0	| UART CTS		|**RJ45 Connector**|
|47	|DD7_R2OUT	|PF1	| UART RTS (INT0)	|		|
|48	|DD7_R1OUT	|PF2	| UART Rx 		|		|
|49	|DD7_T1IN	|PF3	| UART Tx		|		|
|55	|NC		|PF7	| ---		

---
## Memory layout
0x1000 - 0x1023 : EEPROM, configuration. Copied at boot to: 0x2189 - 0x21ad  
  
| MEM 	| EEPROM| Bytes | Value 		|
| :--- 	| :---	| :---	| :---			|
| 0x2000|	| 2	| USART_F0 rx buffer head & tail pointers	|
| 0x2002|	| 2	| USART_F0 tx buffer head & tail pointers  	|
| 0x2005|	| 1	| USART_F0 status  	|
| 0x2006|	| 1	| FPGA communication request   	|
| 0x2007|	| 64	| USART_F0 rx buffer, 64 bytes     	|
| 0x2047|	| xx	| USART_F0 tx buffer, xx bytes       	|
| ...	|	|	|	|
| 0x214f|	| ?	| Some attenuator msg bufffer (ending here)	|
| 0x2156|	| ?	| Some usart f0 msg buffer (ending here)       	|
| 0x2157| 	| 1	| 5V on PORTE.1 present |
| 0x2158| 	| 1	| FPGA 0x7F status 	|
| 0x2159| 	| 1	| Timer THS788 status 	|
| 0x215a| 	| 1	| Timer THS788 value	|
| 0x215b| 	| 1	| Timer FPGA status	| 
| 0x215c| 	| 2	| Timer FPGA value 	| 
| ...	|	|	|	|
| 0x2156|	| ?	| Some usart f0 msg buffer (ending here)|
| ...	|	|	|	|
| 0x2160| 	| 2 	| #16#BC reg Board Temperature		|
| 0x2162| 	| 1 	| clock source settings |
| --- 	| ---	| ---	| --- EEPROM START ---	|
| 0x2163|0x1000 | 24 	| #16#B0 to #16#BB regs (0x2163 - 0x217A)|
| 0x217b|0x1018	| 10	|  regs (0x217B - 0x2182)|
| ...	|	|	|	|
| 0x2187|0x1024	| 48	| #16#25 to #16#3C regs	(0x2187 - 0x21b6)|
| 0x21b7|0x1054	| 24	| #16#01 to #16#C regs	(0x21b7 - 0x22ce)|
| 0x21cf|0x106c	| 96 	| #16#80 to #16#AF regs	(0x21cf - 0x222e)|
| 0x222f|0x10cc	| 2 	| #16#00 reg - Gate Time High	|
| 0x2230|0x10ce	| 2	| #16#3D reg - Amp1_sat	|
| 0x2232|0x10d0	| 2	| Board S/N (4x4bit)	|
| 0x2234|0x10d1	| 1	| ???	|
| --- 	| ---	| ---	| --- EEPROM END ---	|
| 0x2435| 	| 4	| DMA?	|
| 0x2439| 	| 4	| DMA?	|
| 0x243d| 	| 4	| DMA?	|
| 0x2441|	| 2	| #16#BE reg Last Restart Reason	|
| ...	|	|	|	|
| 0x2b76| 	| 8 	| Programming lock [0x78,0x56,0x34,0x12,0x98,0xBA,0xDC,0xFE ] |
| 0x2b92|	| 4	| Flash timestamp	|
| ...	|	|	|	|
| 0x3ffd| 	| 1	| Program status 	|

---  
## UART F0 commands
| CMD 		| DESC 		|
|:---: 		|:---  		|
| CA\r		| Clear alarms|
| CP\r		| Clear params |
| ON or OF	| EEPROM erase? |
| PA		| Flash ATxmega with new firmware |
| PC		| Flash default CDCE62005 settings|
| PF		| Flash FPGA with new firmware |
| RA		| Read FPGA mean amplitude settings (regs 0x64-0x7B)|
| RC		| Read channels 0x2163-0x2179 -> 0xB0-0xBB, 0x21B7-0x22CE -> 0x01-0x0C, 0x2187-0x21B6 -> 0x25-0x60 |
| RF		| Read channel settings (0x21cf-0x222e -> 0x80-0xAF) |
| RS 		| Read device status |
| RT		| Read FPGA TDC phase auto fine tuning result (0x3E-0x3F) & CH TDC raw data (0x40-0x4B) |
| RZ		| Read FPGA ADC baseline (0x0D-0x24) & dispersion (0x4C-0x63) |
| SCL		| Set threshold calibration (0xB0-0xBB)|
| SCS		| ???|
| SCT		| Set TDC values (0x01-0x0C) |
| SCR		| Set ADC range correction (0x25-0x3C)|
| SD		| Set CH ADC Delay |
| SL		| Set CH CFD Threshold |
| SO		| Set CH ADC Zero |
| SS		| Set TRG charge levels (0x3D)|
| ST		| Set TRG settings (0x00) |
| SV		| Set Board ID (0xBD)|
| SZ		| Set CH CFD zero |
| WR 		| Write settings to EEPROM|

