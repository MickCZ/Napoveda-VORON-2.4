# Jak nainstalovat CAN desku EBB SB2209 od Bigteetrech

## 1. DFU mode

1. Ujisti se ze deska není připojena přes CAN rozhraní
2. Na CAN desce vlož jumper na pozici 5V
3. Propoj CAN desku a Raspberry USB kabelem
4. Nastav CAN desku do DFU modu (stisknout a držet tlačítko BOOT poté stisknout a uvolnit tlačítko RESET a uvolnit tlačítko BOOT)

<figure><img src="../.gitbook/assets/SB2209_BOOT.png" alt=""><figcaption></figcaption></figure>

Oveříme desku že je v rezimu DFU následujicím příkazem:

```
lsusb
```

Obdržíme podobný výpis:

```
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 006: ID 0483:df11 STMicroelectronics STM Device in DFU Mode
Bus 001 Device 005: ID 1d50:614e OpenMoko, Inc. stm32f446xx
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Zajima nas pouze polozka u které je uvedeno ze konkretní deska je v DFU mode, pokud na výpisu nevidíme desku v DFU modu opakujeme krok 4 a příkaz pro nový výpis.

## 2. Klipper firmware

Přesuneme se do klipper složky a stáhneme poslední aktualizaci z gitu:

```
cd ~/klipper && git pull
```

Odstraníme předešlé kompilace:

```
make clean
```

Provedeme nastavení HW pro který firmware kompilujeme:

```
make menuconfig
```

Nastavíme takto:

![canboot](images/SB2209\_klipper.png)

Nezapomeňte dopsat: rychlost `500000` (nebo až `1000000`, musí byt stejná jak jsme uvedli v nastaveni rychlosti can sbernice pro can0 rozhrani)

Zmáčkneme `q` pro uložení a `y` pro potvrzení.

Zkompilujeme firmware:

```
make -j4
```

Ověříme že je zařízení stále v DFU modu příkazem:

```
lsusb
```

Musíme vidět podobný výsledek ID se může lišit:

```
Bus 001 Device 006: ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```

Můžeme nahrát vytvořený Klipper firmware:

```
make flash FLASH_DEVICE=0483:df11
```

Vypneme tiskárnu, odpojíme USB kabel od CAN desky a RPI, připojíme sběrnici CAN a nezapomeneme vložit jumper pro zakončení sběrnice na pozici <mark style="color:red;">**120R**</mark>.\
POZOR! důkladně zkontrolujte správnost zapojení CAN vodičů L/H a správnou polaritu 24V, jinak hrozí nevratné poškození desky. Zapneme tiskárnu.

## 3. Vytvoření Canbus interface:

Doinstalujeme balíčky, které budeme potřebovat:

```
sudo apt update && sudo apt install nano wget -y
```

Vytvoříme CAN rozhraní. Otevřeme soubor `/etc/network/interfaces.d/can0` pomocí textového editoru `nano`. Musíme použít `sudo`, protože se jedná o systémový soubor:

```
sudo nano /etc/network/interfaces.d/can0
```

A vložíme následující text, zde si nastavte rychlost jakou jste zvolili při kompilaci firmwaru, v mém případě 1000000. :

```
allow-hotplug can0
iface can0 can static
    bitrate 1000000
    up ifconfig $IFACE txqueuelen 1024
    pre-up ip link set can0 type can bitrate 1000000
    pre-up ip link set can0 txqueuelen 1024
```

Uložíme stisknutím kláves `Ctrl + O` (uložit soubor), `Enter` na potvrzení názvu souboru a `Ctrl + X` na zavření editoru (dole v editoru tyto zkratky můžete vidět).

Restartujeme RPI příkazem:

```
sudo reboot
```

### Zjištění canbus UUID

Přesuneme se do klipper složky:

```
cd ~/klipper
```

Zjistíme UUID příkazem:

```
python3 lib/canboot/flash_can.py -q
```

Dostaneme podobnou odpověď:

```
pi@Voron:~/klipper $ python3 lib/canboot/flash_can.py -q
Resetting all bootloader node IDs...
Checking for canboot nodes...
Detected UUID: aabc3898e436, Application: Klipper
Query Complete
```

Zjištěné moje CanBus UUID je: `aabc3898e436` vaše bude jiné, to své si zkopirujte !!!! Následně ho vložíte do souboru `printer.cfg`

`[mcu EBBCan]`\
`canbus_uuid: aabc3898e436`
