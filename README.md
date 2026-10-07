# esp32-license-plate-detection

License-plate detection with MobileNetV1 running on an ESP32-S3. The network (MobileNetV1, α = 0.5, with a grid detection head and a 7×7×5 output) is trained in TensorFlow/Keras, quantized to INT8 and embedded on an ESP32-S3 with an OV5640 camera. Post-processing filters by confidence and segments the plate characters, and a local web frontend demonstrates the three modes the course required: image, video and real time. Final project for Digital Image Processing (TI0147) at the Federal University of Ceará, taught by Prof. Paulo Cesar Cortez, team of four.

![Pipeline](docs/imagens/pipeline.png)

## Usage

Frontend on a PC, running the same INT8 model through LiteRT:

    pip install ai-edge-litert opencv-python pillow pyserial numpy
    python frontend/frontend_apresentacao.py     # then open http://localhost:8000

Flashing the board: `firmware_esp32/README.md`. Training and export: `modelo_treinamento/README.md`. Full report, in Portuguese: `relatorio/Relatorio_Final.pdf`.

## How it works

1. Capture: the OV5640 delivers a JPEG.
2. Pre-processing: resize to 224×224 and normalize to [-1, 1].
3. Inference: each cell of the 7×7 grid predicts an objectness score and a box (cx, cy, w, h).
4. Decoding: the cell with the highest confidence gives the bounding box.
5. Filtering: a detection fires only when the confidence stays above the operating threshold for 2 consecutive frames. The threshold is calibrated in the field between 0.85 and 0.95 depending on the lighting, always above the no-plate range of 0.42-0.58; the firmware ships with 0.85.
6. Segmentation: the characters are isolated with OpenCV (CLAHE, black-hat, Otsu).
7. Output: the annotated image goes to Telegram from the ESP32, or to the frontend on the PC.

## Results

Detection on a still image, with the grid confidence map:

![Detection](docs/imagens/fig_deteccao.png)

Character segmentation:

![Segmentation](docs/imagens/fig_segmentacao.png)

Metrics over 5 training runs (mean ± standard deviation):

| Metric | Base 1 | Base 2 | Mix |
|---|---|---|---|
| Mean IoU | 0.768 ± 0.002 | 0.543 ± 0.030 | 0.731 ± 0.005 |
| Recall (IoU ≥ 0.5) | 0.973 | 0.667 | 0.922 |
| F1 at IoU 0.5 | 0.983 | 0.750 | 0.944 |

Same model on both platforms:

| Platform | Time per inference | Size |
|---|---|---|
| TFLite INT8 on a PC (LiteRT) | 4-7 ms | 2.35 MB |
| TFLite INT8 on the ESP32-S3 | about 117 s | 2.35 MB |

INT8 quantization kept the quality (IoU about the same as the Keras model). Adding negative images brought false positives to 0 of 4 at the 0.95 threshold. Measured on the ESP32, a well-framed real plate scores 0.98-0.99 and a scene without a plate scores 0.42-0.58.

## Notes

- One model file for both targets, so the PC versus ESP32 comparison isolates the platform.
- The ESP32 takes about two minutes per frame because of the model size, the reference kernels of TFLite Micro and running from PSRAM.
- Two consecutive frames above the threshold before firing trades latency for fewer false alarms.

## Team

| Member | Main work |
|---|---|
| Carlos Vinícius dos Santos Mesquita | segmentation, report, slides |
| Alisson Jaime Sales Barros | embedding on the ESP32, tests, frontend |
| Victor Guedes Alves Teixeira | dataset, training, model, segmentation |
| Kaique Ferreira Braga Silva | firmware, integration, Telegram bot |

## Hardware and environment

- Board: ESP32-S3 (16 MB flash, 8 MB OPI PSRAM, 240 MHz) with an OV5640 camera.
- Embedded: Arduino-ESP32 3.3.8 / ESP-IDF 5.5, `TensorFlowLite_ESP32` library.
- Training: Google Colab (TensorFlow 2.20.0, Python 3.12, NVIDIA Tesla T4).
- Frontend: Python 3 with `ai-edge-litert` and OpenCV.

## Layout

File and folder names are in Portuguese:

    firmware_esp32/        code that runs on the board: Arduino sketch, INT8 model header, partition table
    frontend/              local web app for the demo (image, video and real-time tabs)
    modelo_treinamento/    Colab notebook: training, evaluation and export
    relatorio/             final report in LaTeX and PDF, and the scripts that generate its figures
    slides/                presentation
    imagens_teste/         test images
    videos_teste/          test video
    docs/                  assignment statement and figures

## Links

- Training notebook (Colab): https://colab.research.google.com/drive/1DE-1luzYAd5wMEtShC5QgKr9YezZuzg7
- Datasets: Brazil Plates Detector (Roboflow, CC BY 4.0) and a set provided by the professor.
