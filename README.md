---
Title: Sentiment Analysis, LDA topic modeling and Stock price trend forecast (by Group "Pioneers")
Date: 2025-03-10 12:00
Category: Reflective Report
Tags: Group Pioneers
---

## Sentiment Analysis and Model Comparison
### 1. Importing Libraries
First, we import the necessary libraries. We use `matplotlib` and `seaborn` for data visualization, `sklearn` for machine - learning tasks such as classification, feature extraction, and model evaluation, `numpy` for numerical operations, `lightgbm` for the LightGBM classifier, and `joblib` for saving models.

```python
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.metrics import confusion_matrix, classification_report
import numpy as np
from sklearn.ensemble import RandomForestClassifier
from lightgbm import LGBMClassifier
from sklearn.metrics import accuracy_score
import joblib
from sklearn.preprocessing import StandardScaler
```
### 2. Data Preprocessing
The text data needs to be preprocessed before it can be used for analysis. The `preprocess_text` function converts the text to lowercase, removes punctuation, tokenizes the text, removes stop - words, and performs lemmatization.

```python
def preprocess_text(text):
    # Convert to lowercase
    text = text.lower()
    # Remove punctuation
    text = re.sub(r'[^\w\s]', '', text)
    # Tokenize
    tokens = text.split()
    # Remove stop words
    stop_words = set(stopwords.words('english'))
    tokens = [token for token in tokens if token not in stop_words]
    # Lemmatize
    lemmatizer = WordNetLemmatizer()
    tokens = [lemmatizer.lemmatize(token) for token in tokens]
    return ' '.join(tokens)

merged['processed_content'] = merged['cleaned_content'].apply(preprocess_text)
```
### 3. Sentiment Analysis and Label Generation
We use the `TextBlob` library to perform sentiment analysis on the text. The `get_sentiment` function assigns a 'positive' or 'negative' label based on the polarity of the text.

```python
def get_sentiment(text):
    analysis = TextBlob(text)
    if analysis.sentiment.polarity > 0:
        return 'positive'
    else:
        return 'negative'

merged['sentiment_label'] = merged['cleaned_content'].apply(get_sentiment)
```

### 4. Label Encoding
We encode the sentiment labels using `LabelEncoder` so that they can be used as target variables in machine - learning models.

```python
label_encoder = LabelEncoder()
merged['sentiment_label_encoded'] = label_encoder.fit_transform(merged['sentiment_label'])
```
### 5. Feature Extraction and Data Splitting
We use `CountVectorizer` to extract features from the preprocessed text. Then we split the data into training and test sets.

```python
vectorizer = CountVectorizer()
X = vectorizer.fit_transform(merged['processed_content'])
X = X.astype(np.float64)
y = merged['sentiment_label_encoded']
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```

### 6. Model Training and Evaluation
We define a list of models including Logistic Regression, KNN, CART, Random Forest, and LightGBM. We train each model, make predictions on the test set, calculate the accuracy, and save the confusion matrix and predictions.


```python
models = {
    'Logistic Regression': LogisticRegression(),
    'KNN': KNeighborsClassifier(),
    'CART': DecisionTreeClassifier(),
    'Random Forest': RandomForestClassifier(),
    'LightGBM': LGBMClassifier()
}

results = {}
confusion_matrices = {}
predictions = {}

for model_name, model in models.items():
    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)
    accuracy = accuracy_score(y_test, y_pred)
    results[model_name] = accuracy
    predictions[model_name] = y_pred
    confusion_matrices[model_name] = confusion_matrix(y_test, y_pred)
    print(f'{model_name} accuracy: {accuracy}')
    joblib.dump(model, f'{model_name.replace(" ", "_")}_model.joblib')
```

### 7. Data Visualization
We use different types of plots to visualize the performance of the models:
- **Bar Chart for Accuracy Comparison**: Compares the accuracy of different models.
```python
plt.figure(figsize=(10, 6))
plt.bar(results.keys(), results.values())
plt.title('Model Accuracy Comparison')
plt.xlabel('Models')
plt.ylabel('Accuracy Score')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```
- **Confusion Matrix Heatmap**: Shows the confusion matrix of the Random Forest model.
<img width="400" alt="image" src="https://github.com/user-attachments/assets/2963037f-7279-4914-8688-3bb65178697b" />

```python
plt.figure(figsize=(8, 6))
sns.heatmap(confusion_matrices['Random Forest'], 
           annot=True, 
           fmt='d',
           cmap='Blues',
           xticklabels=['Negative', 'Positive'],
           yticklabels=['Negative', 'Positive'])
plt.title('Confusion Matrix - Random Forest')
plt.xlabel('Predicted')
plt.ylabel('Actual')
plt.show()


```
- **Side - by - Side Bar Chart**: Compares the performance of models based on different metrics (in this case, just accuracy).
```python
metrics = {'Accuracy': results}
df_metrics = pd.DataFrame(metrics)
plt.figure(figsize=(10, 6))
df_metrics.plot(kind='bar')
plt.title('Model Performance Comparison')
plt.xlabel('Models')
plt.ylabel('Score')
plt.xticks(rotation=45)
plt.legend(loc='best')
plt.tight_layout()
plt.show()
```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/522cc5cf-8eec-4a12-b592-f6209e3587c0" />

## LDA analysis
The analysis of news texts is performed using LDA topic modeling. First, the texts are preprocessed (lowercasing, removing punctuation, tokenization, lemmatization, and stopword removal). Then, an LDA model is trained to extract 3 topics and retrieve the top 10 keywords for each topic. Next, the topic distribution for each document is calculated, and high-confidence documents (with topic probability > 0.23) are selected and converted into numerical vectors. Finally, t-SNE is applied for dimensionality reduction to map the topic distributions of high-confidence documents into a 2D space, and a scatter plot is used for visualization, with different colors representing different topics, providing an intuitive display of the document clustering based on topics.
### Code Example:
```python
import pandas as pd
import nltk
from nltk.corpus import stopwords
from nltk.stem import WordNetLemmatizer
import re
from gensim import corpora, models
from sklearn.manifold import TSNE
import matplotlib.pyplot as plt
from wordcloud import WordCloud

custom_stopwords = set(stopwords.words('english')).union({'also', 'u'})

def preprocess_text(text):
    text = text.lower()
    text = re.sub(r'[^\w\s]', '', text)
    tokens = text.split()
    lemmatizer = WordNetLemmatizer()
    tokens = [lemmatizer.lemmatize(token) for token in tokens] 
    tokens = [token for token in tokens if token not in custom_stopwords]
    return tokens

news_df['processed_content'] = news_df['content'].apply(preprocess_text)
dictionary = corpora.Dictionary(news_df['processed_content'])
corpus = [dictionary.doc2bow(text) for text in news_df['processed_content']]
num_topics = 3
lda_model = models.LdaModel(corpus, num_topics=num_topics, id2word=dictionary, passes=15)

num_words = 10
topic_words = {}
for topic_id in range(num_topics):
    top_words = lda_model.show_topic(topic_id, topn=num_words)
    topic_words[topic_id] = [word for word, _ in top_words]

topic_df = pd.DataFrame(topic_words)
topic_df.columns = [f'Topic {i}' for i in range(num_topics)]
print(topic_df)

doc_topics = [lda_model.get_document_topics(doc) for doc in corpus]
high_confidence_docs = []
for i, topics in enumerate(doc_topics):
    max_prob = max(topics, key=lambda x: x[1])[1]
    if max_prob > 0.23:
        high_confidence_docs.append(i)

high_confidence_probs = [doc_topics[i] for i in high_confidence_docs]
high_confidence_vectors = []
for probs in high_confidence_probs:
    vector = [0] * num_topics
    for topic, prob in probs:
        vector[topic] = prob
    high_confidence_vectors.append(vector)

import numpy as np
high_confidence_vectors = np.array(high_confidence_vectors)

tsne = TSNE(n_components=2, random_state=42)
reduced_vectors = tsne.fit_transform(high_confidence_vectors)

topic_labels = [max(topics, key=lambda x: x[1])[0] for topics in high_confidence_probs]

plt.figure(figsize=(10, 8))
scatter = plt.scatter(reduced_vectors[:, 0], reduced_vectors[:, 1], c=topic_labels, cmap='viridis')
legend = plt.legend(*scatter.legend_elements(), title="Topics")
plt.gca().add_artist(legend)
plt.title('LDA Topic Visualization (High Confidence Predictions)')
plt.xlabel('t-SNE Dimension 1')
plt.ylabel('t-SNE Dimension 2')
plt.show()
```

<img width="400" alt="image" src="https://github.com/user-attachments/assets/40c400f9-8770-4586-8891-61103193bc61" />

A word cloud is generated and visualized for each LDA topic. First, the code iterates through each topic and retrieves the top `num_words` keywords and their weights, then stores these keywords and their frequencies in a dictionary. Next, the `WordCloud` is used to generate the word cloud, with the size of the words reflecting their importance within the topic. Finally, each topic’s word cloud is displayed using `matplotlib`, helping to visually represent the core vocabulary and its weight within each topic.

### Code:
```python
for topic_id in range(num_topics):
    top_words = lda_model.show_topic(topic_id, topn=num_words)
    word_freq = {word: freq for word, freq in top_words}
    wordcloud = WordCloud(background_color='white').generate_from_frequencies(word_freq)

    plt.figure()
    plt.imshow(wordcloud, interpolation='bilinear')
    plt.axis('off')
    plt.title(f'Topic {topic_id} Word Cloud')
    plt.show()
```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/1305fafb-ff6f-4809-bc41-dd6319881abf" />

<img width="349" alt="image" src="https://github.com/user-attachments/assets/0cb0ee38-5bd2-4490-afef-a3b707beecbf" />

<img width="349" alt="image" src="https://github.com/user-attachments/assets/3515e69b-6e84-4923-af45-b367f261afe7" />


## Stock price trend forecast
In this section, we try to predict the rise and fall trends of stock prices (classified as "rising", "flat", and "falling") through machine learning models, combine sentiment analysis of news texts (sentiment scores and labels) and structured technical indicators (such as BI, BI_MA), build a multimodal feature set, explore the correlation between market sentiment and stock price fluctuations, and ultimately improve prediction accuracy and provide support for investment decisions.
### 1. data preprocessing
The data cleaning phase processed missing values ​​(filled with mean for numerical features and mode for categorical features) and encoded the target variable into numerical form (0, 1, 2). Then, sentiment scores and one-hot encoded sentiment labels were used as sentiment features, and technical indicators (such as BI_MA) were retained as structural features. SMOTE oversampling was used to solve the class imbalance problem, and the data was split in chronological order (80% for training set and 20% for test set) to simulate the real prediction scenario.
#### * Some data preprocessing codes are shown below:
```python
price_related_cols = [
    'DlyPrc', 'DlyRetx', 'DlyVol',
    'DlyClose', 'DlyLow', 'DlyHigh', 'DlyOpen'
]

excluded_cols = [
    'date', 'title', 'content', 
    'HdrCUSIP', 'PERMNO', 'PERMCO', 
    'Ticker', 'sentiment_label'
] + price_related_cols  

features = data.drop(excluded_cols + ['Price_Change'], axis=1)

lags = 3
for col in ['sentiment_score', 'BI', 'BI_Simple']:
    for i in range(1, lags+1):
        features[f'{col}_lag{i}'] = data[col].shift(i)

features = features.iloc[lags:]
y = data['Price_Change'].iloc[lags:]

numeric_cols = features.select_dtypes(include=np.number).columns.tolist()
features[numeric_cols] = features[numeric_cols].fillna(features[numeric_cols].mean())
```
#### * Top 20 Features are as below :
<img width="400" alt="image" src="https://github.com/user-attachments/assets/ae5c4770-eb48-4102-b524-206d17791087" />

### 2. Model Training & Regressing
In the task of predicting stock price fluctuations, we selected three models: Random Forest, LightGBM and Logistic Regression, based on the following considerations: Random Forest can automatically process high-dimensional features, capture complex nonlinear relationships between features, and is suitable for mixed-type data; LightGBM has fast training speed, low memory usage, and supports efficient processing of high-dimensional sparse data (such as TF-IDF text features), which is suitable for optimizing the use of text features; the Logistic Regression model is simple and highly interpretable, and can clearly show the positive/negative impact of features, which is good for verifying the interpretability of sentiment features.
#### * Part of the model training code is as follows (taking the LightGBM model as an example):
```python
model = Pipeline([
    ('preprocessor', preprocessor),
    ('smote', SMOTE(random_state=42)),
    ('classifier', lgb.LGBMClassifier(
        objective='multiclass',  
        num_class=3,            
        boosting_type='gbdt',
        n_estimators=200,
        learning_rate=0.05,
        max_depth=5,
        class_weight='balanced', 
        random_state=42
    ))
])

df = df.sort_values('date').reset_index(drop=True)
X = df[['content', 'sentiment_score', 'sentiment_label', 'BI', 'BI_Simple', 'BI_MA', 'BI_Simple_MA']]
y = df['Price_Change']

train_size = int(len(X)*0.8)
X_train, X_test = X.iloc[:train_size], X.iloc[train_size:]
y_train, y_test = y.iloc[:train_size], y.iloc[train_size:]
model.fit(X_train, y_train)

y_pred = model.predict(X_test)
class_names = le.classes_

print("the report:")
print(classification_report(y_test, y_pred, target_names=class_names))
```
#### *Example of a regression report output:
<img width="400" alt="image" src="https://github.com/user-attachments/assets/0e8bb705-0cfa-46cb-8288-f013b10082dd" />

### 3. visualization & analysis
In the visualization of model results, we use the accuracy comparison chart to intuitively display the performance of each model, use gradient color matching and shadow texture to highlight the significant advantages of the model, and add a 65% baseline to clarify the optimization space; use the prediction probability distribution chart to analyze the model's prediction confidence for each category, explore the confidence interval of the high confidence area, and reveal the ambiguity of the model's classification of intermediate states; use the feature importance chart to reveal the sentiment score and text features, and verify the important contribution of sentiment analysis to stock price prediction. These visualization methods present the model performance in an intuitive and professional way.
#### * Random Forest Model Visualization Results：
<img width="400" alt="image" src="https://github.com/user-attachments/assets/72cb363c-8231-44ff-ad6c-342ecfbc2857" />

#### * LightGBM Model Visualization Results：
<img width="400" alt="image" src="https://github.com/user-attachments/assets/2654324a-8c9e-4110-92ae-9a50f2179374" />

#### * Logistic Model Visualization Results：
<img width="400" alt="image" src="https://github.com/user-attachments/assets/ec0c7125-fc5e-4ce1-9ad0-ea9554289098" />

#### * Summary of experimental results
This section verifies the effectiveness of multimodal features (sentiment analysis, technical indicators, text content) in predicting stock price rises and falls through random forest, LightGBM and logistic regression models. LightGBM performs best with an accuracy of 67%, and its ability to efficiently process high-dimensional text features is particularly outstanding, verifying the significant contribution of news keywords (such as "profit" and "volatility") to prediction; random forest (63% accuracy) shows the stability of mixed feature interactions, while logistic regression (61% accuracy) reveals the strong correlation between negative sentiment and stock price declines through interpretability (probability increased by 20%). However, all models have weak predictive ability for the "flat" category (F1-score <55%), indicating that the feature discrimination and sample size of the intermediate state still need to be optimized.

### 4. Future improvement
Future work will focus on three aspects: first, feature enhancement, through fine-grained text analysis (such as event extraction, industry keyword screening) and the introduction of interactive features of technical indicators and sentiment (such as sentiment score × volatility), to improve the discrimination of the "equal" category; second, model optimization, designing adaptive sampling strategies (such as customized SMOTE for the "equal" category) and exploring model fusion (such as LightGBM+logistic regression), balancing efficiency and interpretability; third, system implementation, building a real-time prediction framework, dynamically integrating news sentiment and market data, and providing investors with visual decision support tools. Through continuous iteration, the goal is to increase the prediction accuracy to more than 70%.
