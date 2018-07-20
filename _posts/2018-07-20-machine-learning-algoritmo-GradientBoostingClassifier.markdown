---
published: true
title: GradientBoosting Classifier
layout: post
---

GradientBoosting Classifier é um algoritmo que você pode encontrar na biblioteca do [sklearn](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.GradientBoostingClassifier.html)

Ele é como podemos ver pelo nome uma biblioteca de classificação, ele produz um modelo de previsão na forma de um conjunto de modelos de previsão fracos, ele constrói um modelo em modo de estágio em sequência, como outros métodos de otimização e os generaliza ao permitir a otimização de uma função de perda arbitrariamente diferenciável.

### Exemplo aplicado

```python
from pickle import dump, load

from sklearn.ensemble import GradientBoostingClassifier
from sklearn import datasets

from sklearn.metrics import confusion_matrix, classification_report
from sklearn.model_selection import train_test_split

#Importa o database
iris = datasets.load_iris()
X = iris.data
Y = iris.target

#Seta algumas variáveis
seed = 7
test_size = 0.25

#Divide a base de dados
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=test_size)

#Treina o algoritimo
model = GradientBoostingClassifier(n_estimators=50, learning_rate=1, random_state=seed)
model.fit(X_train, Y_train)

#Testa para ver a sua veracidade
predictions = model.predict(X_test)
print(confusion_matrix(Y_test, predictions))
print(classification_report(Y_test, predictions))

#Salva para uso futuro
filename = 'binary_class_model.sav'
dump(model, open(filename, 'wb'))

#Le o arquivo do disco
loaded_model = load(open('binary_class_model.sav', 'rb'))

```
    [[10  0  0]
     [ 0 15  2]
     [ 0  1 10]]
                 precision    recall  f1-score   support
              0       1.00      1.00      1.00        10
              1       0.94      0.88      0.91        17
              2       0.83      0.91      0.87        11
    avg / total       0.92      0.92      0.92        38
