# Docker Image for DensePose

 * [DensePose repository](https://github.com/facebookresearch/Densepose)

## Build notes

  * DensePose requires Caffe2, which got merged into PyTorch. This is why the Dockerfile clones PyTorch and compiles it.
  * DensePose requires Python2.7 🙄

## Prerequesites

  * you'll need a Nvidia GPU, CUDA and [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) to run the image

## Run

If you want to run DensePose on your own data, don't forget to mount your data folders and enable GPU support. Here is an example:

```bash
docker run -v /path/to/your/data/densepose_results:/results -v /path/to/your/data/:/input --gpus 1 cschell/densepose \
   python tools/infer_simple.py \         ─╯
     --cfg configs/DensePose_ResNet101_FPN_s1x-e2e.yaml \
     --output-dir /results/ \
     --image-ext jpg \
     --wts  /weights/DensePose_ResNet101_FPN_s1x-e2e.pkl \
    /input/
```
