---
published: true
title: AdaBoost Classifier
layout: post
---

AdaBoost Classifier é um algoritmo que você pode encontrar na biblioteca do [sklearn](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.AdaBoostClassifier.html)

Ele é como podemos ver pelo nome uma biblioteca de classificação, seu funcionamento é da seguinte maneira, ele combina diversos algoritmos de classificação fracos e transforma todos num unico forte. Um algoritmo de classificação fraco é um algoritmo simples que tem um desempenho ruim, mas que funciona melhor do que ficar adivinhando respostas aleatórias.

Ele pode ser aplicado em qualquer algoritmo de classificação de dados, pois a sua tecnica é baseada em muitos classificadores diferentes e não num unico só, talvez ele possa te retornar um melhor resultado.

Para cada algoritmo que ele testa para classificar, com base no seu percentual o AdaBoost da um peso para ele, por exemplo 0 se ele atigir 50% de sucesso ou se for menos o peso fica negativo, caso for maior que 50% o peso fica possitivo. Algoritmos com peso negativo não são considerados na classificação pois o seus calculos são iguais ou piores do que adivinação para os dados testados.

![Gráfico](http://chrisjmccormick.files.wordpress.com/2013/12/adaboost_alphacurve.png)

### Exemplo aplicado

```python
from pickle import dump, load

from sklearn.ensemble import AdaBoostClassifier
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
model = AdaBoostClassifier(n_estimators=50, learning_rate=1, random_state=seed)
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
    [[14  0  0]
     [ 0 11  3]
     [ 0  1  9]]
                 precision    recall  f1-score   support
              0       1.00      1.00      1.00        14
              1       0.92      0.79      0.85        14
              2       0.75      0.90      0.82        10
    avg / total       0.90      0.89      0.90        38


#### Parâmetros do AdaBoost Classifier

Existe 4 parâmetros os mais importantes são base_estimator, n_estimators, and learning_rate.

- **base_estimator**: É o algoritmo utilizado para treinar os modelos fracos. O padrão é a arvore de decisão, a não ser que queira algo especifico, isso não precisa ser alterado.
- **n_estimators**: É o número de modelos que ele vai utilizar para o treino.
- **learning_rate**: É a taxa de aprendizado, um valor muito baixo demorará mais, mas as vezes com melhor resultado
- **loss**: È exclusivo da AdaBoostRegressor e define a função de perda para usar ao atualizar pesos. Por padrão, será utilizadao a função de perda linear, pode ser alterada para quadrada ou exponencial.

Fontes:

[http://mccormickml.com/2013/12/13/adaboost-tutorial/](http://mccormickml.com/2013/12/13/adaboost-tutorial/)
[https://medium.com/machine-learning-101/https-medium-com-savanpatel-chapter-6-adaboost-classifier-b945f330af06](https://medium.com/machine-learning-101/https-medium-com-savanpatel-chapter-6-adaboost-classifier-b945f330af06)
[https://chrisalbon.com/machine_learning/trees_and_forests/adaboost_classifier/](https://chrisalbon.com/machine_learning/trees_and_forests/adaboost_classifier/)