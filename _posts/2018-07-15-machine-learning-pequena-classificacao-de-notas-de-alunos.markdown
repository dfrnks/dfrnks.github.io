---
published: true
title: Machine Learning, pequena classificação de notas de alunos.
layout: post
---

Esse exemplo de Machine Learning é um exemplo de classificação, ou seja, tenho determinados dados e quero saber se determinados dados parecidos o sua classificação.

O dataset utilizado, a base de dados utilizado é bem simples tem duas informações para auxiliar na classificação (nota e faltas) e duas classes (Aprovado ou não), então lá vamos nós.
## Download dataset
[Dataset](/files/posts/2018-07-15/dataset_classificacao_notas.data)

## Construindo o script de treinamento
##### Iniciando

Primeira coisa iremos importar algumas libs do python

```python
import numpy as np
import pandas as pd
from pickle import dump
from matplotlib import pyplot
from collections import Counter
```
##### Libs do sklearn

```python
from sklearn.model_selection import KFold, train_test_split, cross_val_score

# algoritmos de classificação
from sklearn.svm import SVC
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import confusion_matrix, classification_report
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier, AdaBoostClassifier
```
##### Importar a base de dados

```python
dataset = np.loadtxt('dataset_classificacao_notas.data', delimiter=',')
# Primeira informação é as notas, de 0 a 10
# Segunda é as faltas se for mais do que 18 faltas o aluno está reprovado
# Terceira coluna se está ou não aprovado, 0 ou 1
names=['notas','faltas','aprovado']

# Criamos um data frame para trabalhar de uma forma melhor
dataset=pd.DataFrame(dataset, columns=names)
print(dataset.head())
```
       notas  faltas  aprovado
    0   9.14     9.0       1.0
    1   9.87     6.0       1.0
    2   6.78     4.0       1.0
    3   6.55    36.0       0.0
    4   7.97    32.0       0.0


##### Informação da classificação
Para termos uma noção de como está distribuida as informações
```python
# Distribuição da variável aprovado
print(Counter(dataset['aprovado']))
```
	Counter({0.0: 4212, 1.0: 788})

##### Separando a variável de classificação da base
```python
# Extrai a variável aprovado para uma variável específica e a elimina da base
target = dataset['aprovado']
datasetFiltered = dataset.drop('aprovado', 1)
print(datasetFiltered.describe())
```
             	notas       faltas
    count  5000.000000  5000.000000
    mean      5.009686    24.517200
    std       2.900932    14.412496
    min       0.000000     0.000000
    25%       2.500000    12.000000
    50%       5.020000    25.000000
    75%       7.510000    37.000000
    max       9.990000    49.000000


```python
# Configura alguns parâmetros
seed      = 7
scoring   = 'recall'
n_splits  = 10
test_size = 0.25

kfold = KFold(n_splits=n_splits, random_state=seed)
```

```python
# Separa a base entre treino e teste de forma aleatória
X = datasetFiltered.values
Y = target
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=test_size)
```

```python
models = []
models.append(('ADA', AdaBoostClassifier()))
models.append(('GB', GradientBoostingClassifier()))
models.append(('RF', RandomForestClassifier()))
models.append(('CART', DecisionTreeClassifier()))
models.append(('SVM', SVC()))

# Avalia os algoritmos
results = []
names = []

for name, model in models:
    cv_results = cross_val_score(model, X_train, Y_train, cv=kfold, scoring=scoring)
    results.append(cv_results)
    names.append(name)
    msg="%s: %f (%f)" % (name, cv_results.mean(), cv_results.std())
    print(msg)

#compara os algoritmos
fig = pyplot.figure()
fig.suptitle('Algorithm Comparison')
ax = fig.add_subplot(111)
pyplot.boxplot(results)
ax.set_xticklabels(names)
pyplot.show()

```
Com base nos dados a cima, os 4 primeiros algoritimos tiveram o melhor resultado

    ADA: 1.000000 (0.000000)
    GB: 1.000000 (0.000000)
    RF: 1.000000 (0.000000)
    CART: 1.000000 (0.000000)
    SVM: 0.983109 (0.017251)

```python
# Cria o modelo base com o melhor algoritmo
# no caso pode ser qualquer um dos 4 primeiros
baseline = RandomForestClassifier(random_state=seed)
baseline.fit(X_train, Y_train)
```

```python
#verifica a importância de cada variável no modelo
featureName = pd.DataFrame(datasetFiltered.columns, columns=['Feature'])
featureImp = pd.DataFrame(baseline.feature_importances_, columns=['Imp'])
print(pd.concat([featureName, featureImp], axis=1))
```
      Feature       Imp
    0   notas  0.570272
    1  faltas  0.429728

##### Testando o treinamento com a base de teste
```python
predictions = baseline.predict(X_test)

print(confusion_matrix(Y_test, predictions))
```
    [[1058    0]
     [   0  192]]

```python
print(classification_report(Y_test, predictions))
```
                 precision    recall  f1-score   support
            0.0       1.00      1.00      1.00      1058
            1.0       1.00      1.00      1.00       192
    avg / total       1.00      1.00      1.00      1250

##### Salvando o treinamento
Tudo certo? Salva o modelo num arquivo para uso posteriormente
```python
filename = 'binary_class_model.sav'
dump(baseline, open(filename, 'wb'))
```
#### **Algoritmo completo**

```python
import numpy as np
import pandas as pd

from pickle import dump

from matplotlib import pyplot
from collections import Counter
from sklearn.model_selection import KFold, train_test_split, cross_val_score

from sklearn.svm import SVC
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import confusion_matrix, classification_report
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier, AdaBoostClassifier

dataset = np.loadtxt('dataset_classificacao_notas.data', delimiter=',')

names=['notas','faltas','aprovado']

dataset=pd.DataFrame(dataset, columns=names)
print(dataset.head())


# Distribuição da variável target
print(Counter(dataset['aprovado']))

# dataset.plot()

# Extrai a variável target para uma variável específica e a elimina da base
target = dataset['aprovado']
datasetFiltered = dataset.drop('aprovado', 1)

print(datasetFiltered.describe())


# Configura alguns parâmetros
seed = 7
scoring = 'recall'
test_size = 0.25
n_splits = 10
kfold = KFold(n_splits=n_splits, random_state=seed)

# Separa a base entre treino e teste de forma aleatória
X = datasetFiltered.values
Y = target
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=test_size)


# pyplot.plot(sorted(X))
# pyplot.show()


models = []
models.append(('ADA', AdaBoostClassifier()))
models.append(('GB', GradientBoostingClassifier()))
models.append(('RF', RandomForestClassifier()))
models.append(('CART', DecisionTreeClassifier()))
models.append(('SVM', SVC()))

# Avalia os algoritmos
results = []
names = []

for name, model in models:
    cv_results = cross_val_score(model, X_train, Y_train, cv=kfold, scoring=scoring)
    results.append(cv_results)
    names.append(name)
    msg="%s: %f (%f)" % (name, cv_results.mean(), cv_results.std())
    print(msg)

#compara os algoritmos
fig = pyplot.figure()
fig.suptitle('Algorithm Comparison')
ax = fig.add_subplot(111)
pyplot.boxplot(results)
ax.set_xticklabels(names)
pyplot.show()

# Cria o modelo base com o melhor algoritmo
baseline = RandomForestClassifier(random_state=seed)
baseline.fit(X_train, Y_train)

#verifica a importância de cada variável no modelo
featureName = pd.DataFrame(datasetFiltered.columns, columns=['Feature'])
featureImp = pd.DataFrame(baseline.feature_importances_, columns=['Imp'])
print(pd.concat([featureName, featureImp], axis=1))

predictions = baseline.predict(X_test)

print(confusion_matrix(Y_test, predictions))
print(classification_report(Y_test, predictions))

filename = 'binary_class_model.sav'
dump(baseline, open(filename, 'wb'))
```

## Usando em produção

Para usarmos depois fazermos o seguinte, importamos o arquivo e chamamos o método `predict` passando os dados que desejamos classificar

```python
from pickle import load

import random

# load the model from disk
loaded_model = load(open('binary_class_model.sav', 'rb'))

nota = 6.0
faltas = 2
result = loaded_model.predict([
    [nota,faltas]
])
print(result)

nota = 7.8
faltas = 19
result = loaded_model.predict([
    [nota,faltas]
])

print(result)
```
    [ 1.]
    [ 0.]
