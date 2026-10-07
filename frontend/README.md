# Demo frontend

Local, offline web app that demonstrates the three required modes (image, video and real time) with the same model embedded on the ESP32, running on the PC through LiteRT (about 4 ms per inference). Segmentation uses OpenCV.

## Usage

```bash
pip install ai-edge-litert opencv-python pillow pyserial numpy
python frontend_apresentacao.py          # default port 8000
python frontend_apresentacao.py 9000     # another port
```

Then open http://localhost:8000. `ai-edge-litert` is the TensorFlow Lite runtime and also works on Python 3.14.

## Tabs

| Tab | What it does |
|---|---|
| Image | Pick or drop an image: detection (box and grid confidence map) and character segmentation, in RGB or gray3 mode |
| Video | Pick a video from `videos_teste/`: frame-by-frame detection streamed as MJPEG, with FPS |
| Real time | Connects to the ESP32 over serial (for example `COM3`) and shows the live confidence over time, the inference progress and the event log |
| Metrics | Tables with the measured results (5 runs, PC versus ESP32, effect of the negative images) |

## Notes

- The real-time tab opens the serial port, which resets the ESP32; the first frame takes about two minutes. The ESP32's Wi-Fi network must be up.
- The model is `modelo_grid_224_alpha_0p5_int8.tflite`, in this folder, the same file embedded on the board.
- Everything is served locally, with no internet, so the demo works anywhere.
