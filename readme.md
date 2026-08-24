# Qwen 3.5-4B CadQuery Fine Tuning

This repository is my 2nd (untimed) attempt of finetuning a model to generate CadQuery Python code from images of 3D CAD models. 

In this attempt, I use a vision language model in the hope that it can better translate a CAD image into executable CadQuery code, as compared to converting the image to embeddings and feeding the same to a regular language model

## Dataset

I use the [CADCODER/GenCAD-Code](https://huggingface.co/datasets/CADCODER/GenCAD-Code) dataset, which contains CAD images paired with CadQuery code.

The training set contains approximately **147k image/code pairs**.

Each training example consists of:

- An image of a CAD model
- The ground-truth CadQuery Python code

The model is trained to predict only the CadQuery response (image and instruction not included in the loss)

## Zero-Shot Accuracy

Over a held out set of 200 samples, the model was unable to successfully a single valid geometry. While the code produced by the model qualitatively looked good, it had a lot of syntax issues (i.e. calling cq.WorkPlane instead of cq.Workplane, making up methods that did not exist).

## Fine-Tuning

Finetuned Qwen3.5-4B using LoRA:

- **Model:** Qwen3.5-4B
- **Dataset:** CADCODER/GenCAD-Code
- **Method:** LoRA / supervised fine-tuning
- **Training data:** ~147k examples
- **Image resolution:** 448 × 448
- **LoRA rank:** 16
- **LoRA alpha:** 32
- **Learning rate:** 2e-4
- **Effective batch size:** 16

The training code and notebooks in this repository contain the preprocessing, training and evaluation pipeline.

## Evaluation

The generated CadQuery code is executed to reconstruct the CAD geometry. The generated geometry is then compared against the reference geometry using voxel IoU.

My current SFT checkpoint is **step 27618 (3 epochs)**.

### Results

Evaluation was performed on a held-out set of 200 samples.

| Metric | Result |
|---|---:|
| Mean IoU | 0.5548 |
| Median IoU | 0.5587 |
| Std. Dev. | 0.3931 |
| Valid samples | 186 / 200 |

Already, the model has seen a significant improvement over zero shot.

This is the current SFT baseline that I am using for further experiments.


## Reinforcement Learning

I am currently using this SFT model as the starting point for further fine-tuning with reinforcement learning.

The idea is to have the model generate CadQuery code, execute it to produce the corresponding geometry, and use the resulting voxel IoU as a reward.

This is currently a work in progress.