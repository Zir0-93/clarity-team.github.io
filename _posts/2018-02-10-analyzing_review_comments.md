--- 
title:  "Analyzing GitHub Code Review Comments using Text Classification"
image: /images/code-reviews.png
date:   2018-05-04 15:04:23
categories: [machine-learning]
tags: [machine learning, probabilistic, code reviews, NLP, language models]
excerpt_separator: <!--more-->
---
A code review is a form of code inspection where a developer assesses code for style, defects, and other standards prior to integration into a code base. As part of the code review process on GitHub, developers may leave comments on portions of the unified diff of a GitHub pull request. These comments are extremely valuable in factilitating technical discussion amongst developers, and in allowing developers to get feedback on their code submissions. In an effort to better understand code reivewing habbits, we develop an SVM classifier to classify over 30 000 GitHub review comments based on the main topic
addressed by each comment. E.g. a review comment suggesting an improved variable name might fall under the subject, 
`naming`.
<!--more-->

![sample_comment](/images/review_example.png)

## Review Comment Categories

The list of categories incorporated by our classifier are summarized in the table below. This list was developed based on a manual survey of 
approximately 1000 GitHub review comments taken from the top ten most forked repositories on GitHub. We selected this range of repositories in the hope that the most forked repositories exhibit standard open source code reviewing practices. The selected 
categories reflect the most frequently occurring topics encountered in the surveyed review comments. Majority of the categories 
are related to code level concepts (e.g. variable naming, exception handling); however, certain review comments 
that did not naturally fall into any existing categories and were unrelated to the overall goal of code reviewing were placed in
the "other" category. In situations where a review comment discussed more than one subject, it was given a classification according 
to the topic it spent the most words discussing.

| Category                        | Label | Further Explanation                                                                                 |
|---------------------------------|-------|-----------------------------------------------------------------------------------------------------|
| Readability                     | 1     | Comments related to readability, style, general project conventions.                                |
| Naming                          | 2     |                                                                                                     |
| Documentation                   | 3     | Comments related to licenses, package info, module documentation, commenting.                       |
| Error/Resource Handling         | 4     | Comments related to exception/resource handling, program failure,  termination analysis, resource . |
| Control Structures/Program Flow | 5     | Comments related to usage of loops, if-statements, placement of individual lines of code.           |
| Visibility/ Access              | 6     | Comments related to access level for classes, fields, methods and local variables.                  |
| Efficiency / Optimization       | 7     |                                                                                                     |
| Code Organization/ Refactoring  | 8     | Comments related to extracting code from methods and classes, moving large chunks of code around.   |
| Concurrency                     | 9    | Comments related to threads, synchronization, parallelism.                                          |
| High Level Method Semantics & Design                           | 10    | Comments relating to method design and semantics.                                                           |
| High Level Class Semantics & Design                           | 11    | Comments relating to class design and semantics.                                                           |
| Testing                           | 12    |                                                           |
| Other                           | 13    | Comments not relating to categories 1-12.                                                           |

## SVM Classifier Implementation

We now discuss our SVM text classifier implementation. This experiment represents a typical supervised learning classification exercise.
We first load our training data from the local directory which consists of two files. The first file contains a review comment on each
line, while the second file contains manually determined classifications for each review comment on each line.

```python
with open('review_comments.txt') as f:
    comments = f.readlines()
    
with open('review_comments_labels.txt') as g:
    classifications = g.readlines()
 ```
 Next, we preprocess the data in multiple steps to prepare it for use by our proposed SVM classifier. First, we remove all formatting characters from each comment that are associated with the Markdown syntax. Markdown is a lightweight
 markup language with plain text formatting syntax. It is designed to be easily converted to HTML and many other formats
 using a tool by the same name. Markdown is often used to format readme files, for writing messages in online discussion forums, 
 and in GitHub code review comments. This step is neccessary as the additional formatting related characters introduced by the 
 Markdown stadard will negatively impact our classifier's ability to recognize identical words.
 
 ```python
 import re 

for index, comment in enumerate(comments):
    comments[index] = re.sub("\*|\[|\]|#|\!|`|,|\.|\"|;", "", comments[index])
    comments[index] = re.sub("\.|\(|\)|<|>", " ", comments[index])
    comments[index] = ' '.join(comments[index].split())
```

Next, we remove all stop words from the review comments. A stop word is a commonly used word
(such as “the”, “a”, “an”, “in”) that we would like to ignore. The reason is for this is that these words 
take up valuable processing time, but are not very relevant to the classification task at hand. We can remove them easily
by simply maintaing a list of words that are considered to be stop words. 
Additionally, we stem all the words in our review comments as well. Stemming is the process of reducing inflected
(or sometimes derived) words to their word stem, base or root form. E.g. A stemming algorithm reduces the words 
“fishing”, “fished”, and “fisher” to the root word, “fish”. The Natural Language Toolkit (NLTK) in python has a
list of stopwords stored in 16 different languages, as well as a stemmer implementation we can make use of.

```python
from nltk.stem.snowball import SnowballStemmer
stemmer = SnowballStemmer("english", ignore_stopwords=True)
```

The next step of our preprocessing stage is to convert the comment reviews into numerical feature vectors. This is required to
make our review comments amenable for machine learning algorithms. To do this, we will use the bag of words method, which 
represents a sentence using a feature vector developed based on the number of occurrences of each
term, known as *term frequency*. Thus, the comment “please rename this variable” is, in this view, identical
to the comment “rename this variable please”. We use the Scikit-learn Python library to create feature vectors for our 
review comments using the `CountVectorizer` module.


```python
# Extracting features from text files
from sklearn.feature_extraction.text import CountVectorizer
count_vect = CountVectorizer()
comments_train_counts = count_vect.fit_transform(comments)
comments_train_counts.shape
```
```(307, 1499)```

Moreover, further improvements can be made to this method of representing the review comment texts through the incorporation
of inverse document frequency statistic.
Consider a commonly occurring term like "the". A simple bag of words model based only on term frequency would tend to incorrectly
emphasize review comments which happen to use the word "the" more frequently, without giving enough weight to the more meaningful
terms like "variable" and "naming". This is problematic as the term "the" is not a good keyword to distinguish relevant and
non-relevant documents and terms, unlike the less-common words "variable" and "naming". Hence an inverse document frequency 
factor is incorporated to diminsh the weight of terms that occur very frequently in the review comment set and increases the
weight of terms that occur rarely.

Putting it all together, the weight the td-idf statistic assigns to a given term is:

1. Highest when the term occurs many times within a small number of review comments
2. Lower when the term occurs fewer times in a review comment, or occurs in many review comments
3. Lowest when the term occurs in virtually all review comments.

At this point, we may view each review comment as a vector with one component
corresponding to each term in the dictionary, together with a weight for each
component that is given by the equation above. For dictionary terms that do not occur in
a document, this weight is zero. This vector form will prove to be crucial to
the scoring and ranking capabilities of our SVM classifier.

```python
# TF-IDF
from sklearn.feature_extraction.text import TfidfTransformer
tfidf_transformer = TfidfTransformer()
comments_train_tfidf = tfidf_transformer.fit_transform(comments_train_counts)
comments_train_tfidf.shape
```
```(307, 1499)```


We dedicate 80% of our 1000 GitHub review comments data to the training set, which we use to train our SVM classifier. The remaining 20% of the 
data will be dedicated to the test set, which we use to test the performance of the developed classifier.

```python
from sklearn.model_selection import train_test_split

comment_train, comment_test, classification_train, classification_test = train_test_split(comments, classifications, test_size=0.2)
```
Lastly, we can run our SVM machine learning algorithm with the components developed so far. Additionally, our developed classifier
contains various parameters which can be tuned to obtain optimal performance. Scikit gives an extremely useful tool GridSearchCV 
with which performance tuning for our various parameters will be carried out. As we can see, our classifier scored an accuracy 
of 96% on the test data set.

```python
# Training Support Vector Machines - SVM and calculating its performance
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.linear_model import SGDClassifier

text_clf_svm = Pipeline([('vect', CountVectorizer()), ('tfidf', TfidfTransformer()),
                         ('clf-svm', SGDClassifier(loss='hinge', penalty='l2',alpha=1e-3, max_iter=5, random_state=42))])

print(len(comment_test))
text_clf_svm = text_clf_svm.fit(comments, classifications)
predicted_svm = text_clf_svm.predict(comment_test)
np.mean(predicted_svm == classification_test)
```
``` 0.96 ```

## Classifying GitHub Review Comments
We now leverage the classifier developed in the previous section to classify over 30000 GitHub review comments from the top 100
most forked Java repositories on GitHub. GitHub exposes a REST API that allows developers to interact with the platform, which we will use to mine our Review Comments. In general,
an API provides an interface between two systems to interact with each other programmatically. Representational State Transfer (REST) 
is an architectural style that defines a set of constraints and properties based on HTTP. APIs that conform to the REST architectural 
style, or RESTful web services, provide interoperability between computer systems on the Internet. We first consume the GitHub REST API to
load repository data for the 100 most forked java repositories on GitHub.

```python
import urllib.request
import json

github_token='<token>'
# URL to consume GitHub REST API to retrieve the top 50 most forked repositories on GitHub
url = 'https://api.github.com/search/repositories?q=language:java&sort=forks&order=desc&per_page=100&page=1&access_token=' + github_token
# Execute the HTTP GET request
resp_text = urllib.request.urlopen(urllib.request.Request(url)).read().decode('UTF-8')
# load the JSON response in a python object
repos_json_obj = json.loads(resp_text)
```
Next, we use the GitHub REST API again to collect a list of all the review comments from each repository.

```python
from IPython.display import clear_output

review_comments = []

# loop through our list of 100 most forked java repositories..
for repo in repos_json_obj['items']:
    print("Retrieving review comments for repository:", repo['name'] )
    # i is the current page number
    for j in range(1, 11):
        # URL to consume GitHub REST API to retrieve 100 review comments
        url = 'https://api.github.com/repos/' + repo['owner']['login'] \
        + '/' + repo['name'] + '/pulls/comments?direction=desc&per_page=100&page=' + str(j) \
        + '&access_token=' + github_token
        # Execute the HTTP GET request and store response in object
        json_obj = json.loads(urllib
                              .request
                              .urlopen(urllib.request.Request(url))
                              .read()
                              .decode('UTF-8'))
        # Store all review comments from the response
        review_comments.extend(json_obj)
    clear_output() 
    
print ('Collected', str(len(review_comments)), 'review comments.')
```
```Collected 32512 review comments.```


We categorize each review comment using our SVM classifier, and generate a donut chart demonstrating our results.

```python
import matplotlib.pyplot as plt
import matplotlib
from palettable.tableau import Tableau_10

matplotlib.rcParams.update({'font.size': 18})

# Data to plot
labels = ['Readability', 'Naming', 'Documentation', 'Error/Resource Handling', 
'Control Structures/Code Flow', 'Visiblity', 'Efficiency', 'Code Organization/Refactoring',
'Concurrency', 'Method Design/Semantics', 'Class Design/Semantics', 'Testing', 'Other']


sizes = [0] * 13
explode = [0.03] * 13

# loop through review comments and score
for review_comment in review_comments:
    label = int(text_clf_svm.predict([review_comment['body']])[0])
    sizes[label - 1] += 1

# Create a circle for the center of the plot
my_circle=plt.Circle( (0,0), 0.75, color='white')

# Plot
plt.pie(sizes, labels=labels, colors=Tableau_10.hex_colors,
        autopct='%1.0f%%', explode=explode, shadow=True, startangle=140) 
fig = plt.gcf().set_size_inches(20,20) 
plt.gca().add_artist(my_circle)
plt.show()
```
{% include amcharts.editor.html %}
