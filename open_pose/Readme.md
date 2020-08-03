# Docker Image for OpenPose

 * [OpenPose repository](https://github.com/CMU-Perceptual-Computing-Lab/openpose)

## Build notes

  * based on https://github.com/esemeniuc/openpose-docker

## Prerequesites

  * you'll need a Nvidia GPU, CUDA and [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) to run the image

## Run

If you want to run OpenPose on your own data, don't forget to mount your data folders and enable GPU support. Here is an example:

```bash
# extract keypoints from all images in a particular folder
docker run -v /path/for/results:/results -v /path_wit/:/input --gpus=1 openpose \
        build/examples/openpose/openpose.bin \
          --write_json=/results \
          --image_dir=/input \
          --render_pose=0 \
          --display=0 \
          --face \
          --hand
```
