# base image tags are not orderable by renovate, bump this line by hand
FROM dustynv/speaches:r36.4.0-cu128-24.04
# renovate: datasource=github-releases depName=speaches-ai/speaches versioning=semver
ARG SPEACHES_REF=v0.9.0-rc.3
COPY constraints-lock.txt /tmp/constraints-lock.txt
ENV PIP_NO_CACHE_DIR=1 UV_NO_CACHE=1 UV_HTTP_TIMEOUT=300 \
    PIP_INDEX_URL=https://pypi.org/simple PIP_EXTRA_INDEX_URL= PIP_TRUSTED_HOST= \
    UV_INDEX_URL=https://pypi.org/simple UV_EXTRA_INDEX_URL=
RUN set -ex \
 && apt-get update && apt-get install -y --no-install-recommends libportaudio2 ffmpeg && rm -rf /var/lib/apt/lists/* \
 && /opt/venv/bin/python -m pip install -q uv \
 && ORT_VER=$(/opt/venv/bin/python -c "import onnxruntime; print(onnxruntime.__version__)") \
 && CT2_VER=$(/opt/venv/bin/python -c "import ctranslate2; print(ctranslate2.__version__)") \
 && SP=$(/opt/venv/bin/python -c "import sysconfig; print(sysconfig.get_paths()['purelib'])") \
 && if ! /opt/venv/bin/python -m pip show onnxruntime >/dev/null 2>&1; then \
      D="$SP/onnxruntime-${ORT_VER}.dist-info"; mkdir -p "$D"; \
      printf 'Metadata-Version: 2.1\nName: onnxruntime\nVersion: %s\n' "$ORT_VER" > "$D/METADATA"; \
      printf 'uv\n' > "$D/INSTALLER"; : > "$D/RECORD"; \
    fi \
 && NP_VER=$(/opt/venv/bin/python -c "import numpy; print(numpy.__version__)") \
 && printf 'onnxruntime==%s\nctranslate2==%s\nnumpy==%s\n' "$ORT_VER" "$CT2_VER" "$NP_VER" > /tmp/constraints.txt \
 && cat /tmp/constraints-lock.txt >> /tmp/constraints.txt \
 && rm -rf /opt/speaches \
 && git clone --depth 1 --branch ${SPEACHES_REF} https://github.com/speaches-ai/speaches /opt/speaches \
 && cd /opt/speaches \
 && sed -i 's|^requires-python.*|requires-python = ">=3.12"|; /^required-version/d' pyproject.toml \
 && sed -i 's|"ctranslate2[^"]*",|"ctranslate2",|; s|"faster-whisper[^"]*",|"faster-whisper",|; s|"kokoro-onnx\[gpu\][^"]*",|"kokoro-onnx",|; s|"numpy[^"]*",|"numpy",|; s|"piper-tts[^"]*",|"piper-tts",|' pyproject.toml \
 && /opt/venv/bin/uv pip install --python /opt/venv/bin/python --constraint /tmp/constraints.txt '.[ui]' \
 && /opt/venv/bin/python -c "import onnxruntime as o, ctranslate2 as c, speaches; print('ort', o.__version__, o.get_available_providers()); print('ct2', c.__version__, 'cuda devices', c.get_cuda_device_count())" \
 && ls /opt/speaches/model_aliases.json
WORKDIR /opt/speaches
ENV UVICORN_HOST=0.0.0.0 UVICORN_PORT=8000
ENTRYPOINT []
CMD ["/opt/venv/bin/python", "-m", "uvicorn", "--factory", "speaches.main:create_app", "--host", "0.0.0.0", "--port", "8000"]
