# Prebuilt wheels

`internal/ml_training/requirements.txt` in the Affinda monorepo installs
`detectron2-0.6-cp312-cp312-linux_x86_64.whl` from here for SageMaker training jobs.
The same file is vendored at `third_party/` in the monorepo; keep the two identical.

It is built from **upstream detectron2 `3ff5dd1cff4417af07097064813c9f28d7461d3c`**, not from
this branch's source (the 0.6 release plus a few patches), because our models need
`Backbone.padding_constraints` and newer config keys that 0.6 lacks. CPU-only, against
torch 2.6.0, in `manylinux_2_28` (needs GLIBC_2.14 / GLIBCXX_3.4.18):

```sh
docker run --rm -v "$OUT:/out" -e FORCE_CUDA=0 quay.io/pypa/manylinux_2_28_x86_64 bash -c '
  /opt/python/cp312-cp312/bin/python -m venv /v
  /v/bin/pip install "torch==2.6.0+cpu" "torchvision==0.21.0+cpu" --index-url https://download.pytorch.org/whl/cpu
  /v/bin/pip install setuptools wheel
  git clone https://github.com/facebookresearch/detectron2 /src
  cd /src && git checkout 3ff5dd1cff4417af07097064813c9f28d7461d3c
  /v/bin/pip wheel --no-build-isolation --no-deps -w /out .'
```

sha256: `b6a502ee6452549b1c64eb02e426fff41eef32d0988c9452e9436a4462fbcaa6`
