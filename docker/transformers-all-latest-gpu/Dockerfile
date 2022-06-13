FROM nvidia/cuda:11.2.2-cudnn8-devel-ubuntu20.04
LABEL maintainer="Hugging Face"

ARG DEBIAN_FRONTEND=noninteractive

# The following `ARG` are mainly used to specify the versions explicitly & directly in this docker file, and not meant
# to be used as arguments for docker build (so far).

ARG PYTORCH='1.11.0'
# (not always a valid torch version)
ARG INTEL_TORCH_EXT='1.11.0'
# Example: `cu102`, `cu113`, etc.
ARG CUDA='cu113'

RUN apt update
RUN apt install -y git libsndfile1-dev tesseract-ocr espeak-ng python3 python3-pip ffmpeg git-lfs
RUN git lfs install
RUN python3 -m pip install --no-cache-dir --upgrade pip

ARG REF=main
RUN git clone https://github.com/huggingface/transformers && cd transformers && git checkout $REF
RUN python3 -m pip install --no-cache-dir -e ./transformers[dev,onnxruntime]

# TODO: Handle these in a python utility script
RUN [ ${#PYTORCH} -gt 0 -a ${#PYTORCH} != "pre" ] && VERSION='torch=='$PYTORCH'.*' ||  VERSION='torch'
RUN [ ${#PYTORCH} != "pre" ] && python3 -m pip install --no-cache-dir -U $VERSION --extra-index-url https://download.pytorch.org/whl/$CUDA || python3 -m pip install --no-cache-dir -U --pre torch --extra-index-url https://download.pytorch.org/whl/nightly/$CUDA

# latest `torchvision` and `torchaudio`
# TODO: We might need to specify proper versions that work with a specific torch version (especially for past CI)
RUN python3 -m pip install --no-cache-dir -U torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/$CUDA

RUN python3 -m pip install --no-cache-dir -U tensorflow
RUN python3 -m pip uninstall -y flax jax

# Use installed torch version for `torch-scatter` to avid to deal with PYTORCH='pre'.
# If torch is nightly version, the link is likely to be invalid, but the installation falls back to the latest torch-scatter
RUN python3 -m pip install --no-cache-dir torch-scatter -f https://data.pyg.org/whl/torch-$(python3 -c "from torch import version; print(version.__version__.split('+')[0])")+$CUDA.html
RUN python3 -m pip install --no-cache-dir intel_extension_for_pytorch==$INTEL_TORCH_EXT+cpu -f https://software.intel.com/ipex-whl-stable

RUN python3 -m pip install --no-cache-dir git+https://github.com/facebookresearch/detectron2.git pytesseract https://github.com/kpu/kenlm/archive/master.zip
RUN python3 -m pip install -U "itsdangerous<2.1.0"

RUN python3 -m pip install --no-cache-dir git+https://github.com/huggingface/accelerate@main#egg=accelerate

# When installing in editable mode, `transformers` is not recognized as a package.
# this line must be added in order for python to be aware of transformers.
RUN cd transformers && python3 setup.py develop
