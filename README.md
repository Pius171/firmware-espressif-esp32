# Edge Impulse firmware for Espressif ESP32 (I2S Microphone)
This fork has been modified to work with an esp32 wroom 32 board connected to the INMP441 i2s microphone, using the pinout below:


## Wiring Diagram

| INMP441 Pin | ESP32 Pin (GPIO) | Function |
| --- | --- | --- |
| **VDD** | **3.3V** | Power (Do not use 5V) |
| **GND** | **GND** | Ground |
| **SCK** | **GPIO 26** | Serial Clock (BCLK) |
| **WS** | **GPIO 32** | Word Select (LRCK) |
| **SD** | **GPIO 33** | Serial Data (OUT) |
| **L/R** | **GND** | Left/Right Channel Select |

---

## How to
You can either just flash the exisiting binary file, build the source code yourself or clone the original repo from edge impulse.
### Flashing existing binary file using esptool
- clone this repo to your pc
-  
```cmd
esptool.py --chip esp32 --port COM3 --baud 460800 write_flash -z 0x1000 "C:\Users\DELL\Documents\firmware-espressif-esp32\build\ei_firmware_esp32.bin"
```
- run edge impulse dameaon `edge-impulse-daemon`

### Building the source code
- clone this repo to your pc
- download and install esp-idf V5.1.1
- run `idf.py build`
- run `idf.py -p COMX flash` to flash your firmware to your esp32

### Using the source code from edge impulse
#### Step 1:  clone this repo https://github.com/edgeimpulse/firmware-espressif-esp32.git to your pc

#### Step 2: update the partition.csv file with the conetents below
```csv
# Name,   Type, SubType, Offset,  Size, Flags
nvs,      data, nvs,     ,        0x6000,
phy_init, data, phy,     ,        0x1000,
factory,  app,  factory, ,        0x1F0000,
data,     data, fat,     ,        0x200000,
```

*Note: This gives you a large 2MB data partition to store your audio samples from the INMP441.*

---

#### Step 3: Configure Flash and Partitions in Menuconfig

You need to tell the build system to use the full 4MB of your chip and your new CSV file.

1. Open configuration: `idf.py menuconfig`
2. **Set Flash Size:**
* Go to **Serial Flasher Config** -> **Flash size**.
* Select **4MB**.


3. **Set Partition Table:**
* Go to **Partition Table**.
* Change **Partition Table Config** to **Custom partition table CSV**.
* Ensure **Custom partition table CSV file** is set to `partitions.csv`.


4. **Save and Exit:** Press `S` to save, then `Esc` until you are back at the command line.

---

#### Step 4: Deep Clean and Flash

Since the previous flash was using a 2MB layout on 4MB hardware, we should wipe the chip entirely to prevent "ghost" data from causing issues.

1. **Erase everything:**
```bash
idf.py erase-flash

```


2. **Build and Flash fresh:**
```bash
idf.py build flash monitor

```



---

#### Step 4: Run the Edge Impulse daemon

Once the `monitor` shows the ESP32 is running without crashing, exit the monitor (`Ctrl + ]`) and run the daemon:

```bash
edge-impulse-daemon

```

