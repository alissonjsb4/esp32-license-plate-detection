# Training and export

Google Colab notebook with the whole machine learning pipeline: data, architecture, training over 5 runs, evaluation, character segmentation and export to TensorFlow Lite for the ESP32.

- Notebook: `Trabalho_Final_PDI_MobileNetV1_Grid_Detector.ipynb`
- Colab: https://colab.research.google.com/drive/1DE-1luzYAd5wMEtShC5QgKr9YezZuzg7

## Notebook sections

Numbered as in the notebook:

- 1: setup (`IMG_SIZE=224`, MobileNet α = 0.5)
- 2: core functions (grid-to-box decoding, IoU)
- 3-4: loading and combining the datasets (Base 1 from the professor, Base 2 Brazil Plates)
- 5: MobileNetV1 with the grid detector head (7×7×5 output)
- 6: data augmentation
- 7-8: training and model selection (5 runs, mean and standard deviation)
- 9: test on external images
- 10: character segmentation (OpenCV)
- 11: evaluation on samples
- 12: export to TFLite (float32, dynamic range and INT8) and generation of the C++ header
- 13: Keras versus TFLite comparison (time, IoU, size)

## Data

- Base 1 (provided by the professor) and Base 2, Brazil Plates Detector (Roboflow, CC BY 4.0). Single class `plate`, YOLO annotations.
- Splits: Base 1 350/60, Base 2 200/12, weighted mix 972/72, plus 16 negative images (no plate) to reduce false positives.

## Retraining and export

1. Open the notebook in Colab with a GPU runtime.
2. Set the Google Drive paths in the first cells.
3. Run the training sections (5 runs) and keep the best model.
4. Section 12 writes the INT8 `.tflite` and the C++ header:
   ```bash
   xxd -i modelo_grid_224_alpha_0p5_int8.tflite > modelo_placas_grid_int8.h
   ```
5. Copy the `.h` to `firmware_esp32/Identificador_de_placas/` and the `.tflite` to `frontend/`.

## Final configuration

Adam, fine-tuning in 2 phases (12 + 50 epochs), batch 8, augmentation, `ReduceLROnPlateau` and `EarlyStopping`, 224×224 input, MobileNetV1 α = 0.5, post-training INT8 quantization.
