# Introducción:

## Autores

- Rafael López García 
- Carlos Luis Gento de Celis

## Descripcion de los ficheros:

En este repositorio podemos encontrar los siguientes ficheros y directorios:

- DC_Presentacion.pptx: Presentación Power Point utilizada para la realización del video.
- DC_Presentacion.pdf: Misma presentacion en formato PDF.
- LimpiezaAnalasisDatosMini.mp4: Video presentación de la practica realizada.
- PRA2_respuestas.rmd: fichero en formato "R Markdown" que contiene el codigo fuente y las explicaciones utilizadas para la elaboración del informe completo de la práctica.
- PRA2_respuestas.pdf: fichero PDF que contiene el informe completo con todos los pasos realizados en la práctica.
- Directorio "data": directorio que contiene los ficheros de datos utilizados en la realización de los datos.

# Descripción del dataset

## Origen de los datos, agradecimientos y preguntas que se pretenden responder

El dataset elegido para esta práctica es Cleveland Heart Disease, del sitio web UCI Machine Learning Repository. Estos datos se pueden obtener a través de esta URL: [*https://archive.ics.uci.edu/ml/datasets/Heart+Disease*](https://archive.ics.uci.edu/ml/datasets/Heart+Disease). Los datos fueron recogidos por la Cleveland Clinic Foundation por Robert Detrano.

El dataset elegido contiene 14 atributos con información demográfica y médica de pacientes a los que se les ha detectado la presencia de enfermedad del corazón y de pacientes que estaban sanos. En ese sentido, este dataset contiene información valiosa que puede ser utilizada para detectar la presencia de enfermedad en el corazón a partir de los resultados de diferentes pruebas médicas. Por tanto, la pregunta que queremos intentar responder es qué características demográficas de los pacientes y sus resultados médicos nos pueden ser factores de riesgo o de protección frente a una enfermedad del corazón y, por tanto, podrían ayudar a detectar su presencia.

## Descripción de las variables

El dataset que vamos a utilizar contiene las siguientes 14 variables:

**age**: Edad de la observación en años.

**sex**: Sexo de la observación.  
  - Valor 0: Mujer  
  - Valor 1: Hombre

**cp**: Tipo de dolor de pecho.  
  - Valor 1: típico angina  
  - Valor 2: atípico angina  
  - Valor 3: Sin dolor en angina  
  - Valor 4: asintomático
      
**trestbps**: presión arterial en reposo (en mm Hg al ingreso en el hospital)

**chol**: colesterol sérica en mg / dl

**fbs**: azúcar en sangre en ayunas > 120 ml/dl.  
  - Valor 0: No  
  - Valor 1: Sí

**restecg**: resultados electro en reposo.  
  - Valor 0: normal  
  - Valor 1: tener anomalías en la onda ST-T (inversiones de la onda T y / o elevación o depresión del ST> 0,05 mV)  
  - Valor 2: muestra hipertrofia ventricular izquierda probable o definitiva según los criterios de Estes
  
**thalach**:  frecuencia cardíaca máxima alcanzada durante la prueba de esfuerzo con Talio

**exang**: angina inducida por ejercicio.  
  - Valor 0: No  
  - Valor 1: Sí

**oldpeak**: Depresión del ST inducida por el ejercicio en relación con el reposo

**slope**: la pendiente del segmento ST de ejercicio pico  
  - Valor 1: Creciente  
  - Valor 2: Plana  
  - Valor 3: Decreciente  

**ca**: número de vasos principales (0-3) coloreados por la fluoroscopia

**thal**: Resultado de la prueba de esfuerzo con Talio.  
  - Valor 3: normal  
  - Valor 6: defecto fijo  
  - Valor 7: defecto reversible

**target**: Diagnóstico de enferdad del corazón.  
  - Valor 0: No hay presencia riesgo de infarto  
  - Valores 1 a 4: Sí hay presencia de riesgo de infarto

#  Integración y selección de los datos de interés

En primer lugar cargamos las librerías que vamos a utilizar durante la práctica:
```{r message= FALSE, warning=FALSE}
if (!require('knitr')) install.packages('knitr', dependencies=TRUE); library(knitr)
if (!require('mlr')) install.packages('mlr', dependencies=TRUE); library(mlr)
if (!require('tidyverse')) install.packages('tidyverse', dependencies=TRUE); library(tidyverse)
if (!require('GGally')) install.packages('GGally', dependencies=TRUE); library(GGally)
if (!require('cowplot')) install.packages('cowplot', dependencies=TRUE); library(cowplot)
if (!require('dplyr')) install.packages('dplyr', dependencies=TRUE); library(dplyr)
if (!require('tidyr')) install.packages('tidyr', dependencies=TRUE); library(tidyr)
if (!require('readr')) install.packages('readr', dependencies=TRUE); library(readr)
if (!require('magrittr')) install.packages('magrittr', dependencies=TRUE); library(magrittr)
if (!require('moments')) install.packages('moments', dependencies=TRUE); library(moments)
if (!require('modeest')) install.packages('modeest', dependencies=TRUE); library(modeest)
if (!require('factoextra')) install.packages('factoextra', dependencies=TRUE); library(factoextra)
if (!require('corrplot')) install.packages('corrplot', dependencies=TRUE); library(corrplot)
if (!require('caTools')) install.packages('caTools', dependencies=TRUE); library(caTools)
if (!require('VIM')) install.packages('VIM', dependencies=TRUE); library(VIM)
if (!require('ROCR')) install.packages('ROCR', dependencies=TRUE); library(ROCR)
if (!require('rpart')) install.packages('rpart', dependencies=TRUE); library(rpart)
if (!require('rpart.plot')) install.packages('rpart.plot', dependencies=TRUE); library(rpart.plot)
if (!require('caret')) install.packages('caret', dependencies=TRUE); library(caret)
if (!require('randomForest')) install.packages('randomForest', dependencies=TRUE); library(randomForest)
```

Cargamos el fichero de datos de cleveland y añadimos los nombres de las variables y comprobamos que se ha cargado correctamente:
```{r message= FALSE, warning=FALSE}
df <- read.csv('data/processed.cleveland.data', na = "?", stringsAsFactors = FALSE, header = FALSE)
names(df) <- c('age', 'sex', 'cp', 'trestbps', 'chol', 'fbs', 'restecg', 'thalach', 'exang', 'oldpeak', 'slope', 'ca', 'thal', 'target')
str(df)
```

Prescindimos de la variable *ca* ya que no es una característica propia del cuerpo humano y no parece relevante para nuestro estudio.
```{r message= FALSE, warning=FALSE}
df <- df[,-12]
```

Finalmente, en la fase previa a la limpieza de datos, tenemos un fichero con 303 observaciones y 13 variables.

# Limpieza de los datos.

Empezamos esta sección observando una tabla-resumen con los estadísticos descriptivos de cada variable:
```{r message=FALSE, warning=FALSE}
summarizeColumns(df) %>% knitr::kable(
  caption = "Resumen de estadíticos descriptivos")
```

En la tabla anterior observamos lo siguiente:  
- Sólo una de las 13 variables contiene observaciones con valores perdidos (missing).  
- La variable target tiene 4 valores, aunque sabemos que puede recodificarse en 2 categorías: el valor 0 indica que no hay enfermedad cardíaca, y el resto de valores sí indican la presencia de enfermedad cardíaca.  
- En dicha tabla no se identifican variables categóricas, pero hay varias variables que claramente lo son, como por ejemplo sex o cp.  
- A simple vista no se detectan valores atípicos en las variables que sabemos que son puramente numéricas, aunque más adelante lo comprobaremos con más detalle.

## Tratamiento de valores perdidos

Tal y como vimos en la Table 1, la variable thal tiene 2 obseraciones con valores missing. Al tratarse sólo de 2 observaciones podríamos optar tanto por borrar dichos registros del dataset como imputarlos.

Por lo general, lo más adecuado sería proceder a la imputación de valores missing, ya que de esta manera no se pierde información. Sin embargo, hay que recordar que la imputación es un proceso que debe realizarse con cautela ya que puede afectar a los resultados obtenidos en los análisis, y realizarla de manera adecuada puede suponer una importante inversión de tiempo. En nuestro caso, al tratarse sólo de dos valores, los valores imputados a priori no van a alterar mucho los resultados. Para no perder esas dos observaciones, vamos a imputarlos por el método de kNN:
```{r message= FALSE, warning=FALSE}
set.seed(1)
df$thal <- kNN(df)$thal
```

En caso de haber tenido más observaciones con valores missing, lo más adecuado sería considerar diferentes métodos de imputación alternativos (sustituir por medidas de tendencia central, kNN, modelos de regresión, imputación múltiple...), ver cuál es el más adecuado para nuestros datos y, a medida que se va avanzando en la imputación, realizar comprobaciones sobre los valores imputados para comprobar si afectan a aspectos fundamentales de los datos, como por ejemplo la distribución de las variables imputadas. Pero en nuestro caso no es necesario 

Tras el tratamiento de valores missing, tenemos un dataset con 13 variables y 303 registros completos:

```{r message= FALSE, warning=FALSE}
cat(c("Número de registros:", nrow(df), "\nNúmero de variables:", ncol(df)))
```

## Variable *target*

Como hemos comentado anteriormente, la variable target presenta 4 valores que se pueden recodificar a valor 0 (no hay enfermedad cardíaca) o valor 1 (sí hay enfermedad cardíaca). Creamos la nueva variable *disease* a partir de *target*:
```{r message= FALSE, warning=FALSE}
# Variable target2 para modelos
df$target2[df$target == 0] <- 0
df$target2[df$target == 1] <- 1
df$target2[df$target == 2] <- 1
df$target2[df$target == 3] <- 1
df$target2[df$target == 4] <- 1
# Variable target para exploración de datos
df$target <- factor(df$target2, labels = c("Sin enfermedad", "Con enfermedad"))
```

En la siguiente tabla tenemos la tabulación con las frecuencias absolutas de la variable *target*:
```{r message= FALSE, warning=FALSE}
summary(df$target) %>% knitr::kable(caption = "Presencia de enfermedad del corazón",col.names = "Observaciones")
```

## Variables categóricas

Como hemos comentado anteriormente, en nuestro dataset hay algunas variables que se han cargado como numéricas, pero realmente se trata de variables categóricas. Estas variables son *sex*, *cp*, *fbs*, *restecg*, *exang*, *slope* y *thal*.

Siguiendo la información sobre categorías comentada en el apartado de descripción de las variables, transformamos estas variables en categóricas:
```{r message= FALSE, warning=FALSE}
df <- df %>% mutate(sex = factor(sex, levels = c(0, 1), labels = c("Mujer", "Hombre")))
df <- df %>% mutate(cp = factor(cp, levels = c(1, 2, 3, 4), labels = c("tipico angina", "atipico angina","Sin dolor en angina", "asintomatico")))
df <- df %>% mutate(fbs = factor(fbs, levels = c(0, 1), labels = c("Azucar en ayunas =< 120 mg/dl", "Azucar en ayunas > 120 mg/dl")))
df <- df %>% mutate(restecg = factor(restecg, levels = c(0, 1, 2), labels = c("Normal", "Anomalia en la onda ST-T", "Hipertrofia Ventricular"))) 
df <- df %>% mutate(exang = factor(exang, levels = c(0, 1), labels = c("No", "Sí"))) 
df <- df %>% mutate(slope = factor(slope, levels = c(1, 2, 3), labels = c("Creciente", "Plana", "Decreciente"))) 
df <- df %>% mutate(thal = factor(thal, levels = c(3, 6, 7), labels = c("Normal", "Solucionado", "Reversible"))) 
```

## Identificación y tratamiento de valores extremos.

En los datos hay varias variables numéricas que presentan valores que pueden ser considerados como valores extremos, como por ejemplo la variable chol:
```{r message= FALSE, warning=FALSE, fig.width=5.5, fig.height=3.5}
boxplot(df$chol, main="BoxPlot Chol", col="#336699")
```

Sin embargo, esto no quiere decir que haya que tratarlo. Eso dependerá del tipo de análisis que se quiera hacer, cómo de importante puede ser tener en cuenta valores extremos para dicho análisis y también del conocimiento previo sobre la materia. En el caso de este trabajo en particular puede ser relevante es relevante mantenerlos, ya que se busca detectar posibles patrones en los datos que puedan ayudar a diagnosticar el riesgo de padecer una enfermedad del corazón. Un valor extremo en alguna variable puede ser un indicador muy fuerte de que existe ese riesgo. Por esa razón no trataremos estos valores extremos.

```{r}
write.csv(data, 'data/heart_disease_clean.csv')
```

# Análisis de los datos.

El análosos de los datos se va a dividir en 3 ejercicios:

1) **Análisis exploratorio** de las variables sex, cp, thalach y thal, y su relación con la variable target
2) **Modelo de regresión logística** con target como variable dependiente
3) **Modelo de árbol de decisión** con target como variable dependiente

## Análisis exploratorio de los datos y su relación con la presencia de enfermedad del corazón

### sex

Calculamos la tabla de contingencia entre sex y target y la visualizamos gráficamente:
```{r warning=FALSE, fig.width=5.5, fig.height=3.5}
tc <- table(df$sex,df$target)
tc %>% kable(caption = "Tabla de contingencia de sex vs. target")
plot(tc, col = c("#F8766D","#00BFC4"), main = "sex vs. target")
```

En el grupo de hombres hay más pacientes con presencia de enfermedad del corazón. Vamos a comprobar si existe relación de independencia entre ambas variables. Para ello realizamos el test de independencia de chi-cuadrado entre sex y target:
```{r warning=FALSE}
chisq.test(tc, correct = FALSE)
```

Existe dependencia entre ambas variables. Por tanto, parece que el sexo tiene una relación significativa con la presencia de enfermedad del corazón, y en concreto los hombres suelen presentar más presencia de enfermedad del corazón que las mujeres.

### cp

Calculamos la tabla de contingencia entre cp y target y la visualizamos gráficamente:
```{r warning=FALSE,fig.width=6, fig.height=4}
tc <- table(df$cp,df$target)
tc %>% kable(caption = "Tabla de contingencia de cp vs. target")
plot(tc, col = c("#F8766D","#00BFC4"), main = "cp vs. target")
```

L a mayoría de los pacientes no mostraron síntomas o dolor de angina. En el grupo de asintomáticos parece haber una proporción mucho mayor de pacientes que presentan enfermedad del corazón. En el resto de grupos no parece haber una diferencia muy grande. Esta observación la confirmamos haciendo un test de independencia de chi-cuadrado del grupo de asintomáticos frente al resto de grupos:
```{r warning=FALSE}
df$cp_asintomatico <- ifelse( df$cp == "asintomatico", "yes", "no")
tc_cpasint <- table(df$cp_asintomatico,df$target)
chisq.test(tc_cpasint, correct = FALSE)
```

### thalach

Visualizamos el histograma de la frecuencia cardíaca:
```{r message= FALSE, warning=FALSE,fig.width=5.5, fig.height=3.5}
hist(df$thalach,col = "#336699",border = "black",prob = FALSE,xlab = "Pulsaciones",ylab = "Número de registros",main = "Distribución de frecuencia cardíaca máxima alcanzada")
```

En el histograma vemos que su distribución es ligeramente asimétrica y escorada hacia la parte superior se su distribución.

Ahora visualizamos los diagramas de caja de su distribución para cada grupo de target:
```{r message= FALSE, warning=FALSE,fig.width=5.5, fig.height=3.5}
ggplot(df, aes(x = target, y = thalach))+geom_boxplot(color="#000000", fill="#336699")+ggtitle("BoxPlot de thalach por grupos de target") +theme(plot.title = element_text(hjust = 0.5, face = "bold"))+ ylab("Pulsaciones") + xlab("Presencia de enfermedad")
```

Los diagramas de caja sugieren que la frecuencia cardíaca de los pacientes que no presentan enfermedad es mayor que la de los pacientes que sí presentan enfermedad del corazón.

Testeamos la normalidad de las submuestras de target y la homogeneidad de sus varianzas:
```{r warning=FALSE}
# Test de normalidad para target = 1
shapiro.test(subset(df, target2==1)$thalach)
# Test de normalidad para target = 0
shapiro.test(subset(df, target2==0)$thalach)
# Test de homogeneidad de varianzas
fligner.test(thalach ~ target, data = df)
```

No ha normalidad en la distribución de frecuencia cardíaca en la submuestra de pacientes sin enfermedad y tampoco homogeneidad en las varianzas. Por tanto aplicamos el test de Wilcoxon para comprobar la igualdad de medias entre los grupos de pacientes con enfermedad y sin enfermedad:
```{r warning=FALSE}
wilcox.test(df$thalach ~ df$target, alternative = "greater")
```

El resultado del test concluye que la frecuencia cardíaca de los pacientes que no presentan enfermedad son significativamente mayores que la de los pacientes que sí presentan enfermedad cardíaca.

### thal
 
Calculamos la tabla de contingencia entre thal y target y la visualizamos gráficamente:
```{r warning=FALSE}
tc <- table(df$thal,df$target)
tc %>% kable(caption = "Tabla de contingencia de thal vs. target")
plot(tc, col = c("#F8766D","#00BFC4"), main = "thal vs. target")
```

La mayoría de pacientes, en concreto 167, obtuvieron un resultado normal en la prueba de esfuerzo con Talio. También se ve que los grupos con resultado Solucionado y Reversible presentan más pacientes con enfermedad del corazón que los de resultado Normal. Visto esto, realizaremos un test de independencia chi-cuadrado entre thal y target para el grupo de resultado normal frente al resto de grupos:
```{r warning=FALSE}
df$thal_nonormal <- ifelse( df$thal != "Normal", "yes", "no")
tc_thalnonormal <- table(df$thal_nonormal,df$target)
chisq.test(tc_thalnonormal, correct = FALSE)
```

El resultado del test confirma que existe dependencia entre target y thal, y en concreto que los pacientes que presentan un resultado Normal en la prueba de esfuerzo con Ralio suelen no tener enfermedad del corazón.

## Regresión Logística

Planteamos un modelo de regresión logística con target2 (presencia de enfermedad del corazón) como variable dependiente del modelo y como variables independientes las otras 12 variables del dataset.

Separamos el conjunto de datos en muestras de entrenamiento y test. Dejaremos el 75% de la muestra para el conjunto de entrenamiento y el 25% restante para el conjunto de test.
```{r message= FALSE, warning=FALSE}
set.seed(123)
split=sample.split(df$target2, SplitRatio = 0.75)
qualityTrain=subset(df,split == TRUE)
qualityTest=subset(df,split == FALSE)
nrow(qualityTrain)
nrow(qualityTest)
```

### Estimación del modelo
Estimamos el modelo logit:
```{r message= FALSE, warning=FALSE}
datasetlog=glm(target2 ~ age+sex+cp+trestbps+chol+fbs+restecg+thalach+exang+oldpeak+slope+thal, data=qualityTrain, family = binomial)
summary(datasetlog)
```

En este caso, como se ha podido comprobar existen muchas variables que no son significativas al 5%. Las eliminaremos de nuestro modelo. Las variables que eliminamos en las siguientes iteraciones son age, trestbps, chol, fbs, restecg y slope. Tras esto, estimamos de nuevo el modelo con la nueva especificación:
```{r message= FALSE, warning=FALSE}
datasetlog_end=glm(target2 ~ sex+cp+thalach+oldpeak+thal, data=qualityTrain, family = binomial)
summary(datasetlog_end)
```

El AIC de este último modelo es 179.15, que es inferior al AIC del modelo original, que era 184.37. Esto hace que este segundo modelo de regresión logísta reducido sea preferible al extendido.

De este modelo logit se detectan 4 factores de riesgo (OR superiores a 1): el sexo del paciente (sex), el tipo de dolor en el pecho (cp), la depresión del ST inducida por el ejercicio en relación con el reposo (oldpeak), y el resultado de la prueba de esfuerzo con Talio (thal). Por otro lado, se detecta un factor de protección (OR inferior a 1) en la frecuencia cardíaca máxima alcanzada en la prueba de esfuerzo con Talio (thallach).

### Curva ROC y Accuracy
Ahora analizamos como de bueno es nuestro modelo para entender la capacidad predictora que tiene. Para ello utilizamos un método muy común que es la representación de la Curva ROC:
```{r message= FALSE, warning=FALSE,fig.width=6, fig.height=4}
# Predicciones del modelo
predictTrain=predict(datasetlog_end, type="response")
# Curva ROC
ROCRpred=prediction(predictTrain, qualityTrain$target2)
ROCRperf=performance(ROCRpred,'tpr','fpr')
plot(ROCRperf, colorize=TRUE, print.cutoffs.at=seq(0,1,by=0.1), text.adj=c(-0.2,1.7))
```

Teniendo en cuenta la Curva ROC obtenida, un umbral de clasificación de 0.6 parece ser bastante bueno, ya que parece que es el que corresponde al punto de la curva ROC que está más alejado de la diagonal.

Ahora medimos el Área Bajo la Curva (AUC):
```{r message= FALSE, warning=FALSE}
auc = as.numeric(ROCR::performance(ROCRpred, 'auc')@y.values)
auc
```

El AUC de nuestro modelo es 0.92, lo cual indica que el modelo discrimina de manera excelente.

Finalmente, vamos a medir la accuracy de nuestro modelo tomando 0.6 como umbral de clasificación:
```{r message= FALSE, warning=FALSE}
predictTest=predict(datasetlog, newdata = qualityTest,type = "response")
confMatrix <- table(qualityTest$target,predictTest >=0.6)
confMatrix %>% kable( caption = "Matrix de confusión modelo logit (0.6)")
accuracy <- (confMatrix[1]+confMatrix[4])/sum(confMatrix)
cat(c("Accuracy del modelo logit: ", accuracy))
```

En este caso el total de aciertos es de 58 de un total 76, lo que supone un accuracy del 76,32% con el dataset de test del que disponemos.

## Árbol de decisión

Ahora planteamos un modelo basado en un árbol de decisión, de nuevo con target2 como variable dependiente.

### Construcción del árbol de decisión

Creamos el modelo y dibujamos el árbol de decisión:
```{r message= FALSE, warning=FALSE}
tree<-rpart(target2 ~ age+sex+trestbps+chol+fbs+restecg+thalach+exang+oldpeak+ slope+cp+thal,method = "class",data = qualityTrain)
rpart.plot(tree)
```

En este caso, podemos ver que las variables que el modelo tiene en cuenta a la hora de clasificar a los pacientes son thal, thalach, cp, age, slope y sex.

### Análisis del árbol de decisión

Para analizar el modelo calculamos su matriz de confusión y obtenemos su accuracy:
```{r message= FALSE, warning=FALSE}
# Predicciones con la muestra test
qualityTest$pred <- predict(tree, qualityTest,type="class")
qualityTest$target2 <- as.factor(qualityTest$target2)
# Matriz de confusión
confusionMatrix(qualityTest$pred, qualityTest$target2)
```

En esta ocasión estamos obteniendo un accuracy para nuestro dataset de test del 67.11%, algo más de 9 puntos porcentuales por debajo que el del modelo logit del apartado anterior.

### Mejora del modelo: Random Forest

Como mejora al arbol de decisión vamos a plantear el uso de un modelo Random Forest que, en general, presenta mejores resultados que el obtenido por un árbol de decisión simple.
```{r message= FALSE, warning=FALSE}
qualityTrain$target2 <- as.factor(qualityTrain$target2)
set.seed(100)
model_rf<-randomForest(target2 ~ age+sex+trestbps+chol+fbs+restecg+thalach+ exang+oldpeak+slope+cp+thal,data = qualityTrain)
model_rf
```

En este caso el algoritmo de Random forest ha utilizado un total de 500 arboles utilizando un 3 variables en cada split.

### Análisis del Random Forest

Al igual que con el modelo de árbol de decisión, obtenemos la matriz de confusión del Random Forest y calculamos su accuracy:
```{r message= FALSE, warning=FALSE}
# Predicciones con la muestra test
qualityTest$pred <- predict(model_rf, qualityTest,type="class")
# Matriz de confusión
confusionMatrix(qualityTest$pred, qualityTest$target2)
```

Con el Random Forest hemos obtenido una mejora del accuracy con respecto al modelo del árbol de decisión simple, subiendo de 67.11% a 68.42%. Sin embargo, el accuracy de este modelo sigue siendo inferior al obtenido por el modelo de regresión logística, por lo que en caso de elegir alguno para hacer una clasificiación, nos quedaríamos con el modelo de regresión logística.

\newpage

# Conclusiones

El análisis de las variables sex, cp, thalach y thal nos ha llevado a la conclusión de que la presencia de enfermedad del corazón es más prevalente en hombres que en mujeres (sex), más en pacientes que no presentan síntomas de dolor en el pecho frente a otros que sí la presentan (cp), en pacientes con menor frecuencia cardíaca (thalach) y finalmente también entre los que presentan unos resultados de Solucionado o Reversible en la prueba de esfuezo con Talio (thal).

Del modelo de regresión logística se obtienen conclusiones similares. Se han detectado cuatro como factores de riesgo (sexHombre, cpasintomatico, oldpeak y thalReversible), y un factor de protección (tallach). La AUC del modelo es 0.92 y, se calculó una accuracy de 76.32% con un umbral de clasificación de 0.6.

Finalmente, en el modelo de árbol de decisión las variables que han sido relevantes para clasificar que un paciente tenia enfermedad del corazón han sido en primer lugar el valor del resultado de la prueba de esfuerzo con Talio (thal), la presión arterial (thalach), la edad del paciente (age), el sexo (sex), el tipo de dolor en el pecho, en concreto ser asintomático (cp), y la pendiente del segmento ST (slope). Adicionalmente se ha obtenido una mejora de este árbol de decisión utilizando el algoritmo de Random Forest, pero la accuracy de ambos modelos de clasifiación no superaba el 69%, por lo que el modelo de regresión logística es preferible frente a estos dos modelos.

# Código y Dataset

El código fuente y el dataset utilizado para esta práctica pueden encontrarse en el repositorio Git accesible a través de este enlace: [*Git*](https://github.com/rafaeloga/pra2_tipologia.git).

# Tabla de contribuciones

| Contribuciones | Firma |
|---|---|
| Investigación previa  | R.L.G y C.G.C |
| Redacción de las respuestas  | R.L.G y C.G.C |
| Desarrollo del código  | R.L.G y C.G.C |
