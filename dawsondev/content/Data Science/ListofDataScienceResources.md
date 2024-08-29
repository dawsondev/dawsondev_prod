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

This is the heart of all quality Data Science work, especially as a prerequisite step to using a machine learning model. You can have the most cocaine-fueled grid searched model with bazillion cross validations (more on that later) and what not but all that is useless if your features are low quality.

*What is a feature?* - A feature is essentially a column in a data table,  for example, specifically a pandas dataframe column with say a bunch of numbers in every row. The bunch of numbers can be very meaningful or absolutely useless, and thus features can be very meaningful or absolutely useless.

TO BE CONTINUED

