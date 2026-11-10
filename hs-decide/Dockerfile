# base image tags are not orderable by renovate, bump this line by hand
FROM dustynv/pytorch:2.7-r36.4.0-cu128-24.04
# Hugging Face revision of hindsight-ai/hs-decide-1-flash: weights and the serving package
ARG HS_DECIDE_REF=e2c789026dfd1f7608cafb2ef6a9bb31397af802
ENV PIP_NO_CACHE_DIR=1 PIP_CONFIG_FILE=/dev/null PIP_DISABLE_PIP_VERSION_CHECK=1 \
    PIP_INDEX_URL=https://pypi.org/simple PIP_EXTRA_INDEX_URL= PIP_TRUSTED_HOST=
# The base ships soundfile.py without libsndfile, and Transformers 5 imports it with
# every model, so the server dies at load.
# causal-conv1d comes from the Jetson index: it is the only build against this base's torch.
# triton 3.5 is the oldest with the gluon API that fla 0.6 imports.
RUN set -ex \
 && rm -f /opt/venv/lib/python3.12/site-packages/soundfile.py \
 && pip install -q "huggingface-hub==2.4.0" "transformers==5.20.2" "pydantic==2.14.3" \
      "fastapi==0.162.3" "uvicorn==0.54.0" "einops==0.8.2" \
      "triton==3.5.1" "fla-core==0.6.0" "flash-linear-attention==0.6.0" \
 && pip install -q --no-deps --index-url https://pypi.jetson-ai-lab.io/jp6/cu128 "causal_conv1d==1.5.0.post8" \
 && hf download hindsight-ai/hs-decide-1-flash --revision ${HS_DECIDE_REF} --local-dir /opt/hs-decide-1-flash \
 && rm -rf /opt/hs-decide-1-flash/.cache /opt/hs-decide-1-flash/assets \
 && pip install -q --no-deps /opt/hs-decide-1-flash/serving \
 && pip list 2>/dev/null | grep -iE '^(torch|triton|fla-core|causal.conv1d|transformers|hindsight-decision) '
# Triton autotunes the fla kernels once per input size bucket (~10 s each); a cache on a
# volume keeps the results across restarts.
ENV HF_HUB_OFFLINE=1 TRITON_CACHE_DIR=/cache/triton TRITON_CACHE_AUTOTUNING=1 \
    PORT=8010 MAX_REQUEST_TOKENS=65536
VOLUME /cache
ENTRYPOINT []
CMD ["sh", "-c", ": \"${HS_DECIDE_API_KEY:?set HS_DECIDE_API_KEY}\"; exec hindsight-decision-serve-transformers /opt/hs-decide-1-flash --host 0.0.0.0 --port \"$PORT\" --max-request-tokens \"$MAX_REQUEST_TOKENS\" --api-key \"$HS_DECIDE_API_KEY\""]
