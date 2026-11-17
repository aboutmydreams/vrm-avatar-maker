MediaPipe Tasks Vision 1.1.1 (npm `@mediapipe/tasks-vision`, Apache-2.0, see LICENSE):
`vision_bundle.mjs` and `wasm/vision_wasm_internal.{js,wasm}` (SIMD build only).

`face_landmarker.task`: Google's Face Landmarker model (float16, version 1), from
https://storage.googleapis.com/mediapipe-models/face_landmarker/face_landmarker/float16/1/face_landmarker.task
(Apache-2.0, per the MediaPipe model card).

`hair_segmenter.tflite`: Google's hair segmentation model (float32), from
https://storage.googleapis.com/mediapipe-models/image_segmenter/hair_segmenter/float32/latest/hair_segmenter.tflite
(Apache-2.0, per the MediaPipe hair segmentation model card). Used to tell whether there's hair on
top of the head, and its color.

Loaded only when someone uses "Start from a photo"; the photo stays in the browser.
