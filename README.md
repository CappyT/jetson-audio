# jetson-audio

OpenAI-compatible speech containers for NVIDIA Jetson (JetPack 6.x, L4T r36.4, CUDA 12.x, GPU sm_87), built on top of the `dustynv/*` prebuilt images from jetson-containers so that the CUDA builds of ctranslate2, onnxruntime and torch are reused instead of compiled.

| Image | Upstream | Port | Purpose |
|---|---|---|---|
| `ghcr.io/cappyt/jetson-speaches` | [speaches-ai/speaches](https://github.com/speaches-ai/speaches) | 8000 | STT (`/v1/audio/transcriptions`, faster-whisper on GPU) and TTS (`/v1/audio/speech`, Kokoro ONNX) |
| `ghcr.io/cappyt/jetson-kokoro` | [remsky/Kokoro-FastAPI](https://github.com/remsky/Kokoro-FastAPI) | 8880 | TTS (`/v1/audio/speech`, Kokoro v1.0 on GPU via PyTorch, native opus) |

Why not the upstream images: their `linux/arm64` variants ship CPU-only `ctranslate2` / `onnxruntime` wheels and PyTorch SBSA wheels without sm_87 kernels, so they run without the GPU on Jetson.

## Build

Each directory has a `Dockerfile` with an `*_REF` build arg pinning the upstream git tag. Images are built natively on `ubuntu-24.04-arm` runners (no GPU needed to build; GPU correctness is verified on the device).

```bash
docker build --network host -t jetson-speaches speaches/
docker build --network host -t jetson-kokoro kokoro/
```

`--network host` works around the Docker bridge DNS on the Jetson host.

## Run

```bash
docker run -d --runtime nvidia -p 8000:8000 -v speaches-hf:/data/models/huggingface ghcr.io/cappyt/jetson-speaches:latest
docker run -d --runtime nvidia -p 8880:8880 ghcr.io/cappyt/jetson-kokoro:latest
```

Model cache for speaches lives in `/data/models/huggingface`; download models with `POST /v1/models/<model_id>` (e.g. `deepdml/faster-whisper-large-v3-turbo-ct2`, `speaches-ai/Kokoro-82M-v1.0-ONNX`).
