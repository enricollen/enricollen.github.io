---
title: 🐦 Italian Birds Classifier
date: 2023-04-25 21:30 +0200
categories: [Personal Projects, Italian Birds Classifier]
tags: [deep learning, CNN, classification, computer vision]
---

# Bird Species Recognition API 🦉
Welcome to the Bird Species Recognition API, designed to aid amateur wildlife photographers and bird enthusiasts in identifying Italian bird species. This Python Flask application simplifies bird species identification, making it an invaluable tool during field observations.

### 💡 Motivation
As an amateur wildlife photographer, I often encounter beautiful birds, the species of which are unknown to me. This sparked my interest in developing a tool to assist both myself and others in identifying these birds directly in the field or later at home.

### 🏗️ Architecture
- **Layer 1**: Utilizes **YOLOv3 Tiny** for object detection to confirm the presence of a bird in the image. If a bird is detected, the image proceeds to the next layer.
- **Layer 2**: Uses a **CNN (Convolutional Neural Network)** based on EfficientNet, fine-tuned for 387 Italian bird species, achieving an average accuracy of 80.05%. The model was trained on 69,000 copyright-free images.

### 📸 Usage Tips
To achieve the best results:
- **Upload clear images**: Preferably portraits or side views of birds.
- **Avoid ambiguous shots**: The model performs best with distinct, well-framed images.
- **Example of a good photo to submit**: 

<img src="assets/img/posts/birds_classifier/codirosso.jpg" alt="good_example_to_submit" width="200" height="200">

### ⚠️ Known Limitations
- The model may struggle with misclassifications, particularly:
  - Between very similar species or different families.
  - In distinguishing between males and females across different species.
  - Birds in unusual positions or obscured by foliage might be missed by the detection.
- The **light version** of the model running on a Raspberry Pi compromises some accuracy to ensure operability on constrained devices.

## 🌐 API Endpoints

### 1. **Predict Bird Species**
Upload a bird image to receive the predicted species name.

- **Method**: `POST`
- **Endpoint**: `/bird-prediction`
- **Form Data Key**: `file` (attach your bird image here)

#### Request Example
`curl -X POST -F "file=@path_to_your_bird_image.jpg" http://italianbirds.duckdns.org:5000/bird-prediction`

### Successful Response Example:
`{
  "predicted_species": "Accipiter Nisus"
}`

### 2. **Predict Bird Species with Probabilities**
- **Method**: `POST`
- **Endpoint**: `/bird-probabilities-prediction`
- **Form Data Key**: `file` (attach your bird image here)

#### Request Example:
`curl -X POST -F "file=@path_to_your_bird_image.jpg" http://italianbirds.duckdns.org:5000/bird-probabilities-prediction`

### Successful Response Example:
`{
  "predicted_species": {
    "Accipiter Nisus": 0.95,
    "Buteo Buteo": 0.04,
    "Falco Peregrinus": 0.01
  }
}`

## 🛠️ Setup and Local Deployment
- Clone the repository and navigate to the project directory.
- Install dependencies: `pip install -r requirements.txt`
- Run the application: `python app.py`
- Access the API locally at `http://localhost:5000`

## 🔗 GitHub Repository
Visit the project repository [here](https://github.com/enricollen/italian-birds-classifier-api-tflite-version) for accessing the codebase.

## 🤖 Live Demo
Feel fre to try out the APIs or the live demo hosted on my Raspberry Pi at `http://italianbirds.duckdns.org:5000/`.