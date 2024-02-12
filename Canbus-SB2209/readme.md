# Jak nainstalovat CAN desku EBB SB2209 od Bigteetrech
1. Ujisti se ze deska neni pripojena pres CAN rozhrani
2. Na CAN desce vloz jumper na pozici 5V
3. Propoj CAN desku a Raspberry USB kabelem
4. Nastav CAN desku do DFU modu (stisknout a drzet tlacitko BOOT pote stisknout a uvolnit tlacitko RESET a uvolnit tlacitko BOOT)

Overime desku ze je v rezimu DFU nasledujicim prikazem:
```
lsusb
```
Obdrzime obdobny vypis:
```
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 006: ID 0483:df11 STMicroelectronics STM Device in DFU Mode
Bus 001 Device 005: ID 1d50:614e OpenMoko, Inc. stm32f446xx
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```
Zajima nas pouze polozka u ktere je uvedeno ze konkretni deska je v DFU mode, pokud na vypisu nevidime desku v DFU modu opakujeme krok 4 a prikaz pro novy vypis.
