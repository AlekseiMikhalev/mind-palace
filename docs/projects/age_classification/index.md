# Summary

The aim of this project is to make a research and find a solution for age classification based on face recognition.
Based on face images collected from the camera, the ML model should classify faces into one of the specific age ranges.

## Sources Review

The following articles were analysed:

1. [Gupta, S.K., Nain, N. Review: Single attribute and multi attribute facial gender and age estimation. Multimed Tools Appl (2022)](https://link.springer.com/article/10.1007/s11042-022-12678-6)

2. [ELKarazle, K.; Raman, V.; Then, P. Facial Age Estimation Using Machine Learning Techniques: An Overview. Big Data Cogn. Comput. 2022, 6, 128.](https://doi.org/10.3390/bdcc6040128)

## Research Results

The result of facial age estimation can be either age range (classification problem) or exact age value (regression problem).  
Main challenges in building efficient age prediction systems are:

1. Real-life factors of face imaging such as _resolution_, _sharpness_, _illumination_, _expression_, _occlusion_, _profile_, _frontal_ _view_, _constraint_ _environment_, _unconstraint_ _environment_, _longitudinal_, _race_, _hair_, _scale_, etc.
2. Datasets creation

   Existing datasets usually divided into the following types:

   - **_controlled_** (prepared in controlled environment with limited variability). Examples: [FG-NET](https://datasets.activeloop.ai/docs/ml/datasets/fgnet-dataset/), [MORPH](https://paperswithcode.com/dataset/morph), [VADANA](https://www.eecis.udel.edu/wiki/vims/index.php/Main/VADANA)

   - **_uncontrolled_** (prepared in real-life with different variability). Examples: [LFW](http://vis-www.cs.umass.edu/lfw/), [IMDB-WIKI](https://data.vision.ee.ethz.ch/cvl/rrothe/imdb-wiki/), [ChaLearn Looking at People (LAP)](https://chalearnlap.cvc.uab.cat/), [Specs on Face (SoF)](https://paperswithcode.com/dataset/sof), [MSU LFW+](http://biometrics.cse.msu.edu/Publications/Databases/MSU_LFW+/), [YGA](https://github.com/JingchunCheng/All-Age-Faces-Dataset)

   Steps for dataset creation:

   1. Image of faces collection
   2. Pre-processing
      - cropping
      - rotating
      - aligning
      - augmenting

Evaluation metrics used:

- **_accuracy_**:
  $(true positive + true negative)/(total test samples)$, and

- **_Mean Absolute_** **_Error_** **_(MAE)_**: mean value of the absolute differences between predicted age and real age (ground truth) of test samples

### Steps of model building

**1. Feature extraction**
Extract unique and distinguishable patterns related to particular age classes. That includes facial age features such as texture or edge relathioships of facial skin.

Global Features - apperance based feature extraction. The whole face is considered as feature space.

Local Features - geometry based feature extraction. Feature extraction from facial parts as eyebrow, nose, lips, etc.

**2. Building classification or regression model**

- _Handcrafted-based approach_

    based on combination of filters:

    - Histogram of Oriented Gradients (HOG) or Local Binary Pattern (LBP) to extract edges and shapes from facial image

- _Deep learning approach_
    
    based on CNNs. Examples of pre-trained models used for age estimation: 
    
    - VGG-16, VGG-19, ResNet50, AlexNet, Xception, GoogleLeNet, MobileNetV2 
