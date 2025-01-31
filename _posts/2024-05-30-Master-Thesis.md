---
title: 🎭 Master Thesis - Combining Deepfake Detection with Fake News Analysis
date: 2024-05-30 21:30 +0200
categories: [Master Thesis, On the Effectiveness of Deepfake Detection on Multimodal Fake News]
tags: [thesis, deepfake, generative AI]
---
Thesis Title: 

*On the Effectiveness of Deepfake Detection on Multimodal Fake News*

My Master Thesis, conducted in collaboration with ISTI-CNR of Pisa, focused on the topic of Deepfake Detection. Today, the misinformation spreads rapidly, detecting deepfakes and fake news has become crucial. However, there was a lack of research on combining these two areas. 
My thesis aims to bridge this gap by proposing a new type of analysis that combines the detection of deepfakes and fake news.
It is publicly available [HERE](https://etd.adm.unipi.it/t/etd-05082024-174758/).

# Objectives: 🎯
The primary aims of my thesis were to:
* Propose an innovative analysis about the effectiveness of deepfake detection in the context of fake news.
* Create a brand new dataset that integrates both deepfake images and fake news text, named DeepFakeNews.
* Develop a multimodal deepfake detector, robust towards fake news context.

# Contributions: 🔥
* Innovative Analysis: Proposed an innovative analysis to evaluate the effectiveness of deepfake detection tools when used in conjunction with fake news.
* DeepFakeNews Dataset: Introduced a new public dataset designed to support the detection of both deepfakes and fake news. This dataset, is an expansion of a pre-existing fake news detection dataset called [Fakeddit](https://fakeddit.netlify.app/). 

DeepFakeNews is publicly available on Zenodo ([HERE](https://zenodo.org/records/11186584)) and includes:
- Three CSV files for training, testing, and validation.
- Zip files containing image splits.
- 509,916 images, balanced between 254,958 authentic and 254,958 deepfake images generated via Stable Diffusion 2, Dreamlike, and GLIDE models.

* Multimodal Deepfake Detection Model: Proposed a robust multimodal detection model combining ResNet50 for image analysis and BERT for text analysis, achieving an impressive accuracy of 93.3%.

# Dataset Overview: 📊
The DeepFakeNews dataset was made to address modern misinformation challenges, that include both visual and textual deception.

## Applications: 🔍
This dataset is ideal for:
- Standard deepfake detection.
- Fake news detection.
- Combined multimodal analyses of visual and textual content, providing a robust framework for evaluating detection systems.

## Impact: 📈
Since its release, the DeepFakeNews dataset has been downloaded nearly 100 times, serving as a crucial benchmark for ongoing research in digital misinformation detection.

## 🔗 GitHub Repository
Visit the thesis repository [here](https://github.com/enricollen/DeepfakeDetection) for accessing the codebase and models.