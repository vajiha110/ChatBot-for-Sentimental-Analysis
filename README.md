﻿# ChatBot-for-Sentimental-Analysis
# App.py
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import make_pipeline
from sklearn.metrics import accuracy_score, classification_report
import nltk
from nltk.corpus import stopwords
import string
import joblib
from flask import Flask, request, render_template


# Download NLTK data
nltk.download('stopwords') 
 
# Load the dataset
df = pd.read_csv('C:/Users/Sea Farer/Downloads/Compressed/IMDB Dataset.csv.zip')
df

# Preprocess the data
df['sentiment'] = df['sentiment'].map({'positive': 1, 'negative': 0})
X = df['review']
y = df['sentiment']
df['sentiment'][1454]

# Split the data into training and test sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Create a pipeline with TfidfVectorizer and MultinomialNB
model = make_pipeline(TfidfVectorizer(), MultinomialNB())

# Train the model
model.fit(X_train, y_train)

# Evaluate the model
y_pred = model.predict(X_test)

# Print accuracy
accuracy = accuracy_score(y_test, y_pred)
print(f'Accuracy: {accuracy}')

# Print classification report
report = classification_report(y_test, y_pred, target_names=['Negative', 'Positive'])
print(report)

# Save the model
joblib.dump(model, 'sentiment_model.pkl')


from flask import Flask, request, render_template


app = Flask(__name__)

# Load the trained model
model = joblib.load('sentiment_model.pkl')

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/predict', methods=['POST'])
def predict():
    if request.method == 'POST':
        user_input = request.form['text']
        data = [user_input]
        prediction = model.predict(data)
        sentiment = 'Positive' if prediction == 1 else 'Negative'
        return render_template('result.html', prediction=sentiment)

if __name__ == '__main__':
    app.run(debug=True)


# index.html
<!DOCTYPE html>
<html>
<head>
    <title>Sentiment Analysis Bot</title>
</head>
<body>
    <h1>Welcome to the Sentiment Analysis Bot</h1>
    <form action="/predict" method="post">
        <label for="text">Enter text:</label>
        <input type="text" id="text" name="text">
        <input type="submit" value="Analyze Sentiment">
    </form>
</body>
</html>

# result.html
<!DOCTYPE html>
<html>
<head>
    <title>Sentiment Analysis Result</title>
</head>
<body>
    <h1>Sentiment Analysis Result</h1>
    <p>The sentiment of the entered text is: {{ prediction }}</p>
    <a href="/">Analyze another text</a>
</body>
</html>

