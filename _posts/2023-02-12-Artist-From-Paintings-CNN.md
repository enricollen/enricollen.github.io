---
title: 🎨 Artist Identification from Paintings
date: 2022-02-12 21:30 +0200
categories: [Master Degree Projects, Artist Identification from Paintings]
tags: [deep learning, CNN, ensemble, genetic algorithm, classification, computer vision]
---
The project aims to create an algorithm that can accurately identify the artist of a painting using Convolutional Neural Networks (CNNs). Techniques include building networks from scratch, utilizing pre-trained networks, and creating a final ensemble network composed of the most effective classifiers.

This project was originally develop for the course 'Computer Intelligence and Deep Learning' in MSc in Artificial Intelligence & Data Engineering at University of Pisa.

## Dataset 🖼️
The dataset used is [Best Artworks of All Time](https://www.kaggle.com/datasets/ikarus777/best-artworks-of-all-time) containing artworks of the 50 most influential artists, with about 8,000 unique images. The data was split into training, validation, and test sets with a ratio of 70% - 15% - 15%.

## CNN from Scratch 🧠
Various CNN architectures were tested, starting with a simple model and progressively adding complexity through additional layers, dropout, regularization, and varying strides.

### Results:
- **Base CNN**: Achieved a test accuracy of 56%.
- **Improved CNN**: Through various enhancements, including higher learning rates, dropout, and deeper dense layers, the accuracy improved to around 65%.

## Pre-trained Models 📚
1. **InceptionV3**: Achieved a test accuracy of 75%.
2. **VGG16**: Using the convolutional base of VGG16 with a custom classifier. Achieved a test accuracy of 80%.
3. **ResNet50**: Similar to VGG16, but with ResNet50's convolutional base. Achieved the highest test accuracy of 83%.

## Ensemble 🏆
Combining the predictions of VGG16, ResNet50, InceptionV3, and the best custom CNN model improved performance. The ensemble achieved a test accuracy of approximately 86%.

## Genetic Algorithm for Ensemble Optimization 🧬
A genetic algorithm (GA) is an optimization technique inspired by the process of natural selection. It is used to find approximate solutions to complex problems by evolving a population of candidate solutions over several generations. In this project, the GA was used to optimize the weights of the ensemble model to achieve better performance.

The GA was run for 15 iterations, each consisting of 10 generations with a population size of 100 chromosomes, achieving the highest test accuracy of approximately 87%. This was accomplished by allowing the GA to explore various combinations of weights, selecting the best-performing ones, and continuously improving the population of solutions through crossover and mutation.

## Explainability 🔍
Techniques like intermediate activations and heatmaps were used to understand the model's decision-making process, providing insights into the features learned by the network.

## 🔗 GitHub Repository
Visit the project repository [here](https://github.com/TommyTheHuman/CIDL-ArtistClassification) for project documentation and access to the codebase.