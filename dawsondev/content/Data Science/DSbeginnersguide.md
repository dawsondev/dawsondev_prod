+++
title = "A beginners guide to Data Science"
tags = ["Data Science","Machine Learning","Educational",]
date = "1012-01-02"
+++

### How to get started with all things Data Science?

In this post I wanted to share what resources helped me most on the journey to learn the most important concepts in Data Science and Machine Learning. While I am no means an expert and this field is unbelievably large, I managed to learn a lot about these topics through my course at university and in my free time and just wanted to make a *guided* list of the various knowledge sources I found most helpful as a beginner. This is more a roadmap with linked resources since the topic is way too large to make a detailed guide on. One reason why I wanted to make this is because existing detailed guides, while many are brilliant like [the Python Data Science Handbook](https://jakevdp.github.io/PythonDataScienceHandbook/), they can be a little bit off putting to beginners and in my opinion the early chapters are often a bit too broad in scope if you are really just interested in diving into implementation.

#### Math Prerequisites

I think one misconception is that you need a very strong mathematical background for utilizing Data Science. Yes it helps, but to know the necessary foundations should be enough in the scope of many Data Science Use Cases and that is basically what our professor told us at the start of the semester as I was sitting in the first Data Science Lecture, thinking we are about to get nuked with maths.

Note that I am talking about practical Data Science here, obviously, if you are interested in the deep theoretical concepts behind Data Science this can get very math heavy, very quickly. 

So for basic foundations it should be enough to have a fairly solid understanding of *statistics* mathematics. Having spent a master semester as a tutor for undergrads in this subject, I can say that this is an often glossed over area of maths unfortunately, and many students treat it as unimportant. However, none of it is particularly super complicated, and it is actually quite useful. You should know how to read data from different data visualizations (bar charts, scatter plots, boxplots etc) and what it means when a lot of data is missing, when you have outliers and know essential concepts like standard deviations and what different types of data even exist, and the corresponding limitations that they carry (for example that nominal data needs to be treated differently in analysis than say ordinal data, etc.)

So continuing on with this guide, I will assume you are familiar with the basics of statistical maths. If not and you do not care, you might as well continue on too.

#### Technical Prerequisites

While there are many ways to go about fooling around with Machine Learning and feature engineering Data, by the far the most common practical environment in most educational and industrial settings is powered by python. The great thing about python is that it is fairly straightforward to get into and easier to get into than many other languages like C# or C++ for example which trade ease of use for slightly better performance (However these performance differences can scale and thus their existence is justified).

There are many resources to learn python from and since this is not the focus of this guide I won't stay on this topic too long but check out [W3schools Tutorial](https://www.w3schools.com/python/default.asp) for example as a starting guide and its also good practice to have a [Cheat Sheet](https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://www.pythoncheatsheet.org/&ved=2ahUKEwiHk-D_jJuIAxXi9AIHHYDHHFMQFnoECAgQAQ&usg=AOvVaw296SC0gjzKe0lbpl7-revS) handy. 

You should know the basics, it will be enough to get started. 

#### Data 101: Learning Pandas 

Pandas is a python library that is essential when working with data. Almost any guide on Data Science will tell you to start here or get to this part very early on.  

The best way to learn the basics of pandas quickly is with data wrangling challenges. [Tomas Beuzen's Excercises](https://www.tomasbeuzen.com/python-programming-for-data-science/practice-exercises/chapter8-wrangling-basics-practice.html) are perfect for this and there are many more out there. Just fumble around with some pandas tables until you get the most common methods down.

#### Data 102: Visualization

When building a (good) machine learning model, you will need well structured and feature engineered data, and for that you will need a good idea of what the data you have in front of you actually means. And for that, you need visualizations because we humans are not computers and need fancy pictures to properly interpret data on any meaningful level, also most of us prefer not to go insane. 

You have multiple options here (Python):
- seaborn (my fav)
- matplotlib
- plotly(.express)
...many many more

it doesnt really matter beyond the fact that it should allow most types of data visualizations but virtually all of them do, and on a comparative level. Seaborn is kinda cool in that it offers a few extra things here and there, but all can do the job. Plot some graphs and get familiar with the library you choose.

#### The core of implementing Data Science Use Cases - Feature Engineering

To build a great model, you need great features. Even the fanciest algorithm (with the most exhaustive grid search and bazillion cross-validations) won’t save you if your input features are garbage – it’s the classic “garbage in, garbage out” situation. Feature engineering is the step where you take raw data and transform it into informative inputs for your models. This often requires creativity, domain knowledge, and a lot of data cleaning.

For example, suppose you’re predicting house prices: features like the number of bedrooms or the location are meaningful, whereas something like a random property ID number is useless for prediction. In practice, feature engineering usually involves tasks like:

- **Handling missing data and outliers**: Deciding how to fill in or ignore missing values, and whether to remove or cap extreme outliers (e.g. replacing nulls with the mean, or dropping anomalies).
- **Encoding categorical variables**: Converting non-numeric data (like categories or labels) into a usable numeric form. For instance, you might use one-hot encoding for a column like “City” to create binary flags, or use label encoding for ordinal data.
- **Scaling and normalizing features**: Many algorithms perform better when numerical features are on a similar scale. You might apply normalization (scaling values between 0 and 1) or standardization (zero mean, unit variance) especially for methods like SVMs or neural networks.
- **Creating new features**: This is where domain insight shines. You can combine or transform existing data to make new features. For example, from a date column you could extract “day of week” if that might influence the outcome, or from a person’s full name you could create a feature for name length, etc. Sometimes a simple ratio between two raw features is more predictive than each feature alone.

The key point is that better features make better models. This stage can easily take up the majority of a data scientist’s time. Don’t be afraid to iterate: visualize your engineered features (see how distributions look, how they correlate with the target), and try different transformations. Over time, you’ll develop an intuition for what makes a feature useful. Keep in mind that if your data is unstructured (text, images, etc.), feature engineering might involve more advanced steps (like turning text into numeric vectors, or extracting image features via a neural network), but the core idea remains the same: you’re preparing quality inputs for your model.

#### Training Your First Machine Learning Model

Once you have clean, well-engineered data, the next step is to actually train a machine learning model. This is where the magic happens – the algorithm finds patterns in your features to make predictions. In practical terms, you’ll be using a library like [scikit-learn](https://scikit-learn.org/) (the go-to Python library for ML) to train models with just a few lines of code. The basic workflow is:

- Split your data into a training set and a test set (e.g. 80/20). The training set is what the model learns from, and the test set is held aside to evaluate how the model performs on unseen data.
- Choose an algorithm and train (`fit`) your model on the training data.
- Evaluate the model’s performance on the test set using appropriate metrics (accuracy, precision/recall, RMSE, etc., depending on the task).

For beginners, it’s wise to start with simpler, well-understood algorithms. Some popular model choices include:

- **Linear Regression** – for predicting continuous numerical values (e.g. predicting a house price from features). It’s simple and provides a nice baseline.
- **Logistic Regression** – despite its name, it’s used for classification (such as spam vs. not spam). It outputs probabilities and is great for binary classification tasks.
- **Decision Trees** – intuitive models that split data based on feature values. They work for classification or regression and are easy to interpret (you can visualize the tree).
- **Random Forests** – an ensemble of many decision trees; often a good default choice as they handle nonlinear patterns and usually improve on a single decision tree’s accuracy.
- **k-Nearest Neighbors (kNN)** – a simple method that classifies by looking at the closest data points in the training set. Easy to understand, though it can be slow on large data.

Each algorithm has its strengths and weaknesses, and there are many more (SVMs, Naive Bayes, gradient boosting like XGBoost, etc.), but the ones above are enough to get your feet wet. Using scikit-learn, you can swap between algorithms with minimal code changes, since they share a consistent API (`model.fit()`, `model.predict()`, etc.). As you experiment, try to start simple – get a basic model working end-to-end. This will give you a baseline to improve upon.

#### Model Validation and Tuning

Now, about that “more on that later” – let’s talk about how to properly validate a model and tune it. It’s very important to ensure your model isn’t just memorizing the training data (overfitting) and actually generalizes well. The basic train/test split is a start, but a more robust technique is **cross-validation**.

*What is cross-validation?* In simple terms, it means you cut your data into several folds and train/test the model multiple times, each time using a different fold as the test set and the rest as training data. For example, in 5-fold cross-validation, you’d train and test 5 times on different splits of the data, then average the performance. This way, your model gets validated on every data point at least once, giving you a more reliable estimate of how it will perform in the real world. Cross-validation is great for getting the most out of a limited dataset, and it helps catch issues like overfitting – if your model performs well on every fold, it’s likely generalizing rather than just memorizing one particular train/test split.

Hand in hand with validation is the concept of **hyperparameter tuning**. Hyperparameters are the settings of a model that you configure (as opposed to parameters that the model learns from data). Examples include the depth of a decision tree, the number of neighbors in kNN, or the learning rate in a neural network. Choosing good hyperparameters can significantly improve a model’s performance. One common approach is **grid search**, which is basically a brute-force method: you define a set of possible values for each hyperparameter, then try every combination to see which yields the best result (often using cross-validation for each combination). Libraries like scikit-learn provide convenient tools for this (e.g. `GridSearchCV` which performs cross-validated grid search automatically).

Tuning hyperparameters with techniques like grid search (or more sophisticated ones like random search or Bayesian optimization) is powerful, but it can be computationally expensive – hence the joking reference to a “cocaine-fueled grid search” burning through countless combinations. As a beginner, start with sensible defaults and only tune a few important hyperparameters. Remember, no amount of tweaking can save a bad feature set. Always return to your features if the model isn’t performing well, rather than blindly tuning. Validation techniques will help you judge if an improvement is real or just random chance.

#### Getting Started with Computer Vision

So far, we’ve focused on data science in the context of structured data (tables of numbers, categories, etc.), but what if you want to work with images? **Computer Vision** is an exciting subfield of data science where the goal is to enable machines to interpret visual data (images or video). Getting started with computer vision as a beginner might seem daunting, but it has become much more accessible in recent years thanks to ready-to-use tools and pre-trained models.

First, understand that images aren’t like rows in a spreadsheet – you can’t feed raw JPEGs into a typical scikit-learn model. Instead, you’ll typically use **deep learning** (neural networks) for most vision tasks. Don’t worry, you don’t need a PhD in AI to try it out. High-level libraries like [TensorFlow](https://www.tensorflow.org/) (with Keras) and [PyTorch](https://pytorch.org/) provide relatively beginner-friendly ways to build and train neural networks for images. In fact, a common “hello world” for computer vision is training a simple Convolutional Neural Network (CNN) to classify images in the [MNIST dataset](https://www.tensorflow.org/datasets/catalog/mnist) (handwritten digits) or the [Fashion-MNIST dataset](https://github.com/zalandoresearch/fashion-mnist) (images of clothing). These are small grayscale image datasets that come pre-labeled, so you can focus on learning how the model architecture and training process works.

If you want to work on your **own image data**, you’ll need to gather and label images for the task at hand. This is where tools like [Label Studio](https://labelstud.io/) come in handy. Label Studio is an open-source annotation tool that makes it easy to **label images** (and other data types too) for machine learning. For example, if you’re building a model to recognize different types of animals in photos, you’d upload your images into Label Studio and draw bounding boxes or assign class labels to each image. This labeled dataset can then be fed into a training script. Data labeling can be time-consuming, but it’s a crucial step – your model can only learn from what you show it.

When it comes to training computer vision models, a good strategy for beginners is to leverage **transfer learning**. This means you take a model that has already been pre-trained on a large image dataset (like ImageNet, which has millions of images of many objects) and fine-tune it on your smaller, specific dataset. The pre-trained model has already learned to detect generic patterns and features (edges, shapes, textures), which you can adapt to your problem with much less data and compute than training from scratch. Both Keras and PyTorch have a zoo of pre-trained models (e.g. ResNet, MobileNet, VGG, etc.) that you can import and fine-tune. There are plenty of tutorials out there to guide you through this process once you’re ready.

Finally, keep in mind that computer vision is a big field. **Start simple**: image classification (assigning a label to an entire image) is far easier to begin with than, say, object detection (locating multiple objects in an image) or image segmentation (precisely outlining objects). Once you have one or two basic image projects under your belt, you can progressively tackle more complex tasks. And just as with the rest of data science, **practice is key**. You might consider joining computer vision competitions on platforms like [Kaggle](https://www.kaggle.com/), or just exploring public datasets and trying to build something fun (like a doodle recognizer or a plant species identifier). It’s a fantastic way to apply the skills from this guide in a new domain.


