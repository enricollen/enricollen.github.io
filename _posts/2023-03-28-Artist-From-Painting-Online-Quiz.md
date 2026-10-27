---
title: 🎨❓ Artist Identification from Paintings - Online Quiz Game Against an AI
date: 2022-02-12 21:30 +0200
categories: [Personal Projects, Artist Identification from Paintings - Online Quiz Game Against an AI]
tags: [deep learning, CNN, classification, computer vision, game]
image: /assets/img/posts/artist_quiz_game/screenshot_1.png
---
As an extension of my Deep Learning academin project, I developed an engaging online quiz game where players test their knowledge of art by guessing the right artist from a given painting. The twist? You're playing against an AI model !

## Game Overview 🎮
The game consists of 10 rounds of painting guessing. In each round, the player is presented with an art picture and must choose the correct artist from four options. The choices are randomly selected from a pool of 11 famous artists, making each game unique and unpredictable.

### Game Mechanics
- **Rounds**: Each game has 10 rounds of guessing.
- **Choices**: For each painting, you must choose the artist from four different options.
- **Scoring**:
  - **Player**: You gain 1 point for each correct answer.
  - **AI**: The AI also makes its prediction for each painting, scoring 1 point for each correct guess.
- **End of Game**: At the end of the game, both your score and the AI's score are reported. You can choose to play again with new images and different artists.

### The AI Opponent 🤖

The AI model you're playing against is a VGG16 pre-trained ConvNet with a test accuracy of nearly 83% on a similar dataset. 

## Technology Stack
The game is implemented using Python and Flask.

## 🔗 GitHub Repository
Visit the project repository [here](https://github.com/enricollen/Artists-from-Paintings-Quiz) for accessing the CNN model and to the codebase.

## 🛠️ Setup and Local Deployment
- Clone the repository and navigate to the project directory.
- Install dependencies: `pip install -r requirements.txt`
- Run the application: `python artists_quiz.py`
- Navigate to `http://localhost:5000/`

Happy guessing!

## Screenshots 📸
Here is a screenshot illustrating the web game interface:
![Screenshot 1](/assets/img/posts/artist_quiz_game/screenshot_1.png)
![Screenshot 2](/assets/img/posts/artist_quiz_game/screenshot_2.png)