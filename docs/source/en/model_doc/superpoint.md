<!--Copyright 2024 The HuggingFace Team. All rights reserved.

Licensed under the MIT License; you may not use this file except in compliance with
the License.

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on
an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
specific language governing permissions and limitations under the License.

⚠️ Note that this file is in Markdown but contain specific syntax for our doc-builder (similar to MDX) that may not be
rendered properly in your Markdown viewer.


-->

# SuperPoint

<div class="flex flex-wrap space-x-1">
<img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-DE3412?style=flat&logo=pytorch&logoColor=white">
</div>

## Overview

The SuperPoint model was proposed
in [SuperPoint: Self-Supervised Interest Point Detection and Description](https://huggingface.co/papers/1712.07629) by Daniel
DeTone, Tomasz Malisiewicz and Andrew Rabinovich.

This model is the result of a self-supervised training of a fully-convolutional network for interest point detection and
description. The model is able to detect interest points that are repeatable under homographic transformations and
provide a descriptor for each point. The use of the model in its own is limited, but it can be used as a feature
extractor for other tasks such as homography estimation, image matching, etc.

The abstract from the paper is the following:

*This paper presents a self-supervised framework for training interest point detectors and descriptors suitable for a
large number of multiple-view geometry problems in computer vision. As opposed to patch-based neural networks, our
fully-convolutional model operates on full-sized images and jointly computes pixel-level interest point locations and
associated descriptors in one forward pass. We introduce Homographic Adaptation, a multi-scale, multi-homography
approach for boosting interest point detection repeatability and performing cross-domain adaptation (e.g.,
synthetic-to-real). Our model, when trained on the MS-COCO generic image dataset using Homographic Adaptation, is able
to repeatedly detect a much richer set of interest points than the initial pre-adapted deep model and any other
traditional corner detector. The final system gives rise to state-of-the-art homography estimation results on HPatches
when compared to LIFT, SIFT and ORB.*

<img src="https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/transformers/model_doc/superpoint_architecture.png"
alt="drawing" width="500"/>

<small> SuperPoint overview. Taken from the <a href="https://huggingface.co/papers/1712.07629v4">original paper.</a> </small>

## Usage tips

Here is a quick example of using the model to detect interest points in an image:

```python
from transformers import AutoImageProcessor, SuperPointForKeypointDetection
import torch
from PIL import Image
import requests

url = "http://images.cocodataset.org/val2017/000000039769.jpg"
image = Image.open(requests.get(url, stream=True).raw)

processor = AutoImageProcessor.from_pretrained("magic-leap-community/superpoint")
model = SuperPointForKeypointDetection.from_pretrained("magic-leap-community/superpoint")

inputs = processor(image, return_tensors="pt")
outputs = model(**inputs)
```

The outputs contain the list of keypoint coordinates with their respective score and description (a 256-long vector).

You can also feed multiple images to the model. Due to the nature of SuperPoint, to output a dynamic number of keypoints,
you will need to use the mask attribute to retrieve the respective information :

```python
from transformers import AutoImageProcessor, SuperPointForKeypointDetection
import torch
from PIL import Image
import requests

url_image_1 = "http://images.cocodataset.org/val2017/000000039769.jpg"
image_1 = Image.open(requests.get(url_image_1, stream=True).raw)
url_image_2 = "http://images.cocodataset.org/test-stuff2017/000000000568.jpg"
image_2 = Image.open(requests.get(url_image_2, stream=True).raw)

images = [image_1, image_2]

processor = AutoImageProcessor.from_pretrained("magic-leap-community/superpoint")
model = SuperPointForKeypointDetection.from_pretrained("magic-leap-community/superpoint")

inputs = processor(images, return_tensors="pt")
outputs = model(**inputs)
image_sizes = [(image.height, image.width) for image in images]
outputs = processor.post_process_keypoint_detection(outputs, image_sizes)

for output in outputs:
    for keypoints, scores, descriptors in zip(output["keypoints"], output["scores"], output["descriptors"]):
        print(f"Keypoints: {keypoints}")
        print(f"Scores: {scores}")
        print(f"Descriptors: {descriptors}")
```

You can then print the keypoints on the image of your choice to visualize the result:
```python
import matplotlib.pyplot as plt

plt.axis("off")
plt.imshow(image_1)
plt.scatter(
    outputs[0]["keypoints"][:, 0],
    outputs[0]["keypoints"][:, 1],
    c=outputs[0]["scores"] * 100,
    s=outputs[0]["scores"] * 50,
    alpha=0.8
)
plt.savefig(f"output_image.png")
```
![image/png](https://cdn-uploads.huggingface.co/production/uploads/632885ba1558dac67c440aa8/ZtFmphEhx8tcbEQqOolyE.png)

This model was contributed by [stevenbucaille](https://huggingface.co/stevenbucaille).
The original code can be found [here](https://github.com/magicleap/SuperPointPretrainedNetwork).

## Resources

A list of official Hugging Face and community (indicated by 🌎) resources to help you get started with SuperPoint. If you're interested in submitting a resource to be included here, please feel free to open a Pull Request and we'll review it! The resource should ideally demonstrate something new instead of duplicating an existing resource.

- A notebook showcasing inference and visualization with SuperPoint can be found [here](https://github.com/NielsRogge/Transformers-Tutorials/blob/master/SuperPoint/Inference_with_SuperPoint_to_detect_interest_points_in_an_image.ipynb). 🌎

## SuperPointConfig

[[autodoc]] SuperPointConfig

## SuperPointImageProcessor

[[autodoc]] SuperPointImageProcessor

- preprocess
- post_process_keypoint_detection

## SuperPointForKeypointDetection

[[autodoc]] SuperPointForKeypointDetection

- forward
