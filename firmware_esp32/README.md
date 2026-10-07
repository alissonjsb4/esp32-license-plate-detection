# ESP32-S3 firmware

The code that runs on the board (Arduino): camera capture, inference with TensorFlow Lite Micro, bounding box drawing and sending the photo to Telegram.

    Identificador_de_placas/
        Identificador_de_placas.ino   main sketch
        modelo_placas_grid_int8.h     INT8 model as a C++ array (about 2.35 MB)
        partitions.csv                partition table (5 MB app)

## Credentials

The top of the `.ino` has placeholders to replace with your own network and bot:

```cpp
const char* ssid     = "SUA_REDE";
const char* password = "SUA_SENHA";
#define BOT_TOKEN "SEU_TOKEN_DO_BOT_TELEGRAM"
#define CHAT_ID   "SEU_CHAT_ID"
```

Never commit a real bot token to a public repository.

## Flashing (Arduino IDE 2.x)

1. Install the esp32 core by Espressif (3.3.8) from the Boards Manager.
2. Open the `Identificador_de_placas/` folder (the `.ino` must sit in a folder with the same name).
3. Select ESP32S3 Dev Module and set PSRAM to OPI PSRAM, Flash Size to 16MB, Partition Scheme to Custom (it uses the sketch's `partitions.csv`), USB CDC On Boot to Enabled and CPU Frequency to 240 MHz.
4. Connect the board through the native USB port (USB-Serial/JTAG) and upload.

The board has two USB connectors. Upload through the native one, which enters download mode by itself; the other one (UART bridge) shows the serial output at 115200 baud but may fail to upload with "Wrong boot mode".

With arduino-cli:

```bash
FQBN="esp32:esp32:esp32s3:PSRAM=opi,FlashSize=16M,PartitionScheme=custom,CDCOnBoot=default,CPUFreq=240"
arduino-cli compile --fqbn "$FQBN" Identificador_de_placas
arduino-cli upload  --fqbn "$FQBN" -p <NATIVE_USB_PORT> Identificador_de_placas
```

## Operation notes

- `setup()` waits for Wi-Fi before starting the camera and the model, so the configured network has to be up or the board stays at boot.
- Embedded inference takes about two minutes per frame (large model, TFLite Micro reference kernels, PSRAM). That is expected.
- Trigger: confidence at or above `CONFIANCA_MINIMA` (0.85 in the code, calibrated between 0.85 and 0.95) for 2 consecutive frames sends the photo to Telegram.
