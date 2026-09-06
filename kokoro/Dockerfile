# base image tags are not orderable by renovate, bump this line by hand
FROM dustynv/kokoro-tts:fastapi-r36.4.0-cu128-24.04
# renovate: datasource=github-releases depName=remsky/Kokoro-FastAPI versioning=semver
ARG KOKORO_FASTAPI_REF=v0.8.2
ENV PIP_NO_CACHE_DIR=1 UV_NO_CACHE=1 UV_HTTP_TIMEOUT=300 \
    PIP_INDEX_URL=https://pypi.org/simple PIP_EXTRA_INDEX_URL= PIP_TRUSTED_HOST= \
    UV_INDEX_URL=https://pypi.org/simple UV_EXTRA_INDEX_URL=
RUN set -ex \
 && /opt/venv/bin/python -m pip install -q uv \
 && TORCH_VER=$(/opt/venv/bin/python -c "import torch; print(torch.__version__)") \
 && printf 'torch==%s\n' "$TORCH_VER" > /tmp/constraints.txt \
 && mv /opt/kokoro-fastapi /opt/kokoro-fastapi.base \
 && git clone --depth 1 --branch ${KOKORO_FASTAPI_REF} https://github.com/remsky/Kokoro-FastAPI /opt/kokoro-fastapi \
 && cd /opt/kokoro-fastapi \
 && /opt/venv/bin/uv pip install --python /opt/venv/bin/python --constraint /tmp/constraints.txt . \
 && mkdir -p api/src/models/v1_0 \
 && cp -a /opt/kokoro-fastapi.base/api/src/models/v1_0/. api/src/models/v1_0/ \
 && rm -rf /opt/kokoro-fastapi.base \
 && ls api/src/models/v1_0 \
 && /opt/venv/bin/python -c "import torch, kokoro, misaki; print('torch', torch.__version__, 'cuda', torch.cuda.is_available()); print('kokoro', kokoro.__version__)"
WORKDIR /opt/kokoro-fastapi
ENV PYTHONPATH=/opt/kokoro-fastapi:/opt/kokoro-fastapi/api PYTHONUNBUFFERED=1 \
    USE_GPU=true DEVICE=gpu \
    MODEL_DIR=/opt/kokoro-fastapi/api/src/models VOICES_DIR=/opt/kokoro-fastapi/api/src/voices/v1_0
ENTRYPOINT []
CMD ["/opt/venv/bin/python", "-m", "uvicorn", "api.src.main:app", "--host", "0.0.0.0", "--port", "8880"]
