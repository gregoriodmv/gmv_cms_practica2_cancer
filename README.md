---
title: "Práctica 2 de Tipología de Datos"
output:
  html_document:
    keep_md: yes
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)

if(!require(corrplot)){install.packages("corrplot")}
if(!require(ggplot2)){install.packages("ggplot2")}
if(!require(ggpubr)){install.packages("ggpubr")}
if(!require(rcompanion)){install.packages("rcompanion")}
if(!require(gridExtra)){install.packages("gridExtra")}

library (corrplot)
library (ggplot2)
library(ggpubr)
library(rcompanion)
library (gridExtra)
```

## Tratamiento y Análisis del Dataset de Cáncer de Mama      

**Gregorio de Miguel Vadillo**  
**Carlos Muñiz Solaz**

**Fecha:** Diciembre 2017 - Enero 2018

![](images/cancer-mama.jpg)

## 1. Descripción del dataset. ¿Por qué es importante y qué pregunta/problema pretende responder?

El dataset se corresponde a un conjunto de muestras de cancer de mama recibidas en el Dr. Wolberg, de la Universidad de Wisconsin, en el periodo comprendido entre Enero de 1989 y Noviembre de 1991.

El dataset consiste en 699 observaciones y 11 atributos que definen a cada observación. De los 11 atributos, 1 atributo se corresponde con el identificador de la muestra, 9 indican características de una observación y el undécimo indica la clasificación de la muestra como tumor benigno o tumor maligno. Vease a continuación información sobre los atributos:

1. Sample code number: id number
2. Clump Thickness: 1 - 10
3. Uniformity of Cell Size: 1 - 10
4. Uniformity of Cell Shape: 1 - 10
5. Marginal Adhesion: 1 - 10
6. Single Epithelial Cell Size: 1 - 10
7. Bare Nuclei: 1 - 10
8. Bland Chromatin: 1 - 10
9. Normal Nucleoli: 1 - 10
10. Mitoses: 1 - 10
11. Class: Variable categorica: 2 -> cancer es benigno, 4 -> cancer es maligno

Los valores 1 a 10 que toman los atributos 2 a 10 reflejan el grado de penetración del tumor en la célula.

Contar con una base de datos con muestras reales de cualquier emfermedad, cancer de mama en este caso, y su posterior estudio es de suma importancia, pues permitiría poder detectar la enfermedad y aplicar tratamientos en etapas de esta cada vez más precoces, lo que sin duda incrementaría la probabilidad de supervivencia del paciente. 

Muchos son los posibles objetivos que tiene este dataset, aunque nosotros vamos a utilizarlo para determinar las características que permiten diferenciar una muestra de tumor benigna de una maligna, por lo que nuestro objetido se centra en la clasificación de muestras.

### Carga del dataset en el entorno y tratamiento inicial de los datos

Como hemos comentado anteriormente, el dataset contiene 699 observaciones y 11 tributos. El archivo que lo contiene es el fichero "breast-cancer.txt" que tiene el siguiente formato.      
**NOTA.** Se muestra solamente un conjunto formado por las primeras observaciones:

![**_Formato del fichero que contiene los datos_**](images/formato_fichero.png)

Como puede observarse, el fichero no dispone de *header* y los campos están separados por comas. Por lo tanto, empleamos el siguiente comando para realizar la carga de la información del fichero dentro den entorno de desarrollo. La variable (data frame) que almacenará la información se llamará **breast_cancer_set**:

```{r chunk_carga_1}
breast_cancer_set<-read.table(file = "breast-cancer.txt", sep = ",")
```

Se comprueba que efectivamente en **breast_cancer_set** tenemos las 699 observaciones y los 11 atributos:

```{r chunk_carga_2}
dim(breast_cancer_set)
```

Visualizamos los 10 primeros registros del data frame para comprobar la integridad de los datos:

```{r chunk_carga_3}
head(breast_cancer_set)
```

Todos los atributos se han cargado como numéricos excepto V7, que se ha cargado como factor. Se comprueban sus niveles:

```{r chunk_carga_4}
levels(breast_cancer_set$V7)


```

Hay un nivel denotado como "?" que aparentemente se corresponde con datos incompletos. De acuerdo a la información del dataset, existen 16 datos incompletos. Comprobamos si se corresponden a los niveles denotados con "?":

```{r chunk_carga_5}
table(breast_cancer_set$V7)
```

Efectivamente el número de "?" coincide con el número de valores incompletos. Nos interesa que todos los atributos sean numéricos, con lo cual convertimos el atributo V7 a "character". Posteriormente convertimos los "?" a NA y finálmente se transforma el atributo a "numeric":

```{r chunk_carga_6}
breast_cancer_set$V7<-as.character(breast_cancer_set$V7)
breast_cancer_set$V7[breast_cancer_set$V7 == "?"]<-NA
breast_cancer_set$V7<-as.numeric(breast_cancer_set$V7)
```

El resultado es el siguiente:

```{r chunk_carga_7}
breast_cancer_set$V7
```

Los 16 valores incompletos representan el 2,3% de las muestras, lo cual no supone un gran impacto en la calidad del dataset, por lo que decidimos eliminarlas del dataset original. Se crea un nuevo data frame con todas las muestras excepto aquellas que contengan un valor del atributo V7 igual a NA. Llamaremos **breast_cancer_clean** a la nueva variable:

```{r chunk_carga_8}
breast_cancer_clean<-breast_cancer_set[complete.cases(breast_cancer_set),]
```

El resultado debe ser un dataset con 683 observaciones y 11 atributos:

```{r chunk_carga_9}
dim(breast_cancer_clean)
```

A continuación se renombran las columnas, de forma que los nombres de los atributos tengan sentido, y se visualizan los primeros registros para comprobar que los atributos han cambiado correctamente:

```{r chunk_carga_10}
colnames(breast_cancer_clean)<-c("id", "clump.thick", "uniform.size", "uniform.shape", "adhesion", "epithelial.cell.size", "bare.nuclei", "bland.chromatin", "nucleoli","mitoses","class")
head(breast_cancer_clean)
```


## 2. Limpieza de los datos
### A. Selección de los datos de interés a analizar. ¿Cuáles son los campos más relevantes para responder al problema?

Para realizar el estudio de las muestras, utilizaremos la información proporcionada por los atributos 2 al 10, es decir, todos los atributos excepto "id" y "class". El atributo "id" nos permite identificar de forma unívoca a una muestra dentro del conjunto de muestras, pero es simplemente un identificador, no proporciona ninguna información valiosa de las características de la muestra. El atributo "class" indica la clase a la que pertenece la muestra. Es decir, ha sido añadido a posteriori una vez que se han estudiado el resto de características de la prueba. Por tanto, lo descartamos para nuestro análisis. Este campo si que será importante más adelante, ya que nos servirá para comprobar si nuestro estudio clasifica las muestras correctamente. Por ello, creamos un vector para almacenar los valores del atributo "class" y poder ser utilizado posteriormente. La variable se llamará **clases_objetivo**:

```{r chunk_carga_11}
clases_objetivo<-breast_cancer_clean$class
```

Modificamos el dataset **breast_cancer_clean** eliminamos el atributo "id":

```{r chunk_carga_12}
breast_cancer_clean<-subset(breast_cancer_clean,,-c(id))
```

Cambiamos la clase a "B" y a "M", si el tumor es Benigno o Maligno:


```{r chunk_carga_13}
breast_cancer_clean$class <- factor (breast_cancer_clean$class, labels = c("B", "M"))
```

El nuevo dataset deberá tener 683 observaciones y 10 atributos:

```{r chunk_carga_14}
dim(breast_cancer_clean)
```


### B. ¿Los datos contienen ceros o elementos vacíos? ¿Y valores extremos? ¿Cómo gestionarías cada uno de estos casos?

Como se comprobó anterioremte, el *dataset original* contenía 16 valores incompletos. En este caso, una vez detectados pasamos a su eliminación, tras comprobar que no suponía un gran impacto en la calidad del dataset resultante.

Realizamos la comprobación de la no existencia de valores extremos:

```{r chunk_2_b_1}
summary(breast_cancer_clean)
```

Como se puede comprobar, los valores de los atributos están comprendidos en el rango [1-10], y se corresponde con los valores que se indicaban en la información del dataset original. Es decir, no hay valores extremos.

En el caso de haber detectado la presencia de algún valor fuera del rango de valores permitidos podríamos optar por alguna de las siguientes soluciones:

**Eliminación del registro que los contiene.** Dependería de si el número de registros afectados no tiene un gran impacto en la calidad del dataset resultante.

**Ajustar el valor al límite más cercano.** Tendríamos que hacer la asumpción de que un valor mayor que 10 es 10 y un valor menor que 1 es 1.

## 3. Análisis de los datos

A partir de ahora sólo trabajaremos sobre el dataset *breast_cancer_clean*.

### 3.1. Selección de los grupos de datos que se quieren analizar/comparar

Analizaremos y compararemos todos los atributos del dataset para identificar cuales de ellos son los importantes para clasificar si el cancer es *benigno* o *maligno*. 
En primer lugar, observamos que el dataset no está equilibrado:

```{r chunk_3_1_1}
barplot (table(breast_cancer_clean$class), xlab = "Tipo de cancer", ylab = "numero de casos", col=c("darkblue","red"), main = "Distribución de clases en el dataset breast_cancer_clean")

prop.table(table(breast_cancer_clean$class))
```

Vemos que hay más caso de cancer *begnino* que *maligno*. Esto es lo esperado en el caso de dataset con enfermedades. 

También se puede observar que existe una alta correlación entre la variable *uniform.size* y *uniform.shape*

```{r chunk_3_1_2}
corr_mat <- cor(breast_cancer_clean[,1:9])
corrplot(corr_mat, order = "hclust", tl.cex = 1, addrect = 8)
```

Podemos mostrar las distribuciones de las variables según el cancer es "benigno" o "maligno"

```{r chunk_3_1_3}
benign <- breast_cancer_clean[breast_cancer_clean$class == "B", ]
malign <- breast_cancer_clean[breast_cancer_clean$class == "M", ]
par(mfrow=c(3,3))
for (i in 1:9) {
   hist(benign[[i]], col = c("darkblue"), main = colnames(benign)[i], xlab = "");
   hist (malign[[i]], col = c("red"), add = T)    
}
```

Se puede observar que para casi todas las variables, los valores para el caso de cancer "benigno" son muy pequeños. Sin embargo, para el cancer "maligno" los valores están mucho más distribuidos.


### 3.2. Comprobación de la normalidad y homogeneidad de la varianza. Si es necesario (y posible), aplicar transformaciones que normalicen los datos.

Para comprobar la normalidad de las distintas variables podemos utilizar gráficas Q-Q y comprobar si diferen de una distribución normal.

```{r chunk_3_2_1}
par(mfrow=c(3,3))
for (i in 1:9) {
     q1 <- qqnorm(benign[[i]], plot.it = FALSE)
     q2 <- qqnorm(malign[[i]], plot.it = FALSE)
     plot(range(q1$x, q2$x), range(q1$y, q2$y), type = "n", xlab ="", ylab = "", main = colnames(benign)[i])
     points(q1, col = "blue")
     points(q2, col = "red", pch = 3)
     abline(a=mean(benign[[i]]),b=sd(benign[[i]]))
     abline(a=mean(malign[[i]]),b=sd(malign[[i]]))
}
```

Visualmente se puede comprobar que ninguna de las variables sigue una distribución normal ya que los puntos están fuera de la linea recta.

Para comprobar analiticamente que ninguna de las variables sigue una distribución normal podemos usar el **test de Shapiro-Wilks*:

**Test de Shapiro-Wilks**

```{r chunk_3_2_2}
shapiro.test (benign$clump.thick)
shapiro.test (malign$clump.thick)
shapiro.test (benign$uniform.size)
shapiro.test (malign$uniform.size)
```
El test lo he aplicado para las variables **clump.thick** y **uniform.size**. Se puede ver como en ambos casos el *p-valor* es muy inferior a 0.05, con lo cual podemos rechazar la hipotésis nula y asumir que las variables no siguen una distribución normal.
Podemos mostrar la función de distribución para comprobar sigue o no forma de campana:

```{r chunk_3_2_3}
ggdensity(breast_cancer_clean$clump.thick, 
         main = "Grafico de Densidad del clump thick",
         xlab = "Tamaño del clump thick")
```

Como se puede observar una vez más la distribución no sigue una forma de campana, con lo que concluimos que las variables de nuestro dataset no siguen distribuciones normales.

Vamos a intentar transformar las variables a distribuciones normales:

```{r chunk_3_2_4}
par(mfrow=c(3,1))
plotNormalHistogram(benign$clump.thick, xlab = "clump.thick", main = "Distribucion sin transformar")
plotNormalHistogram(log10(benign$clump.thick + 1), main = "Usando una transformación logaritmica")
plotNormalHistogram(sqrt(benign$clump.thick), main = "Usando una transformación de raíz cuadrada")
```

- Hemos usado dos transformaciones para intentar normalizar la distribución. Una **transformación logaritmica** y una **transformación de raiz cuadrada**. 
- De las tres distribución la que más se acerca a una normal es la **transformación logaritmica**.

Aún así, esta distribución también falla el test de normalidad de Shapiro.
Es por ello, que si queremos realizar pruebas estadisticas tendremos que realizarlas con **tests no parámetricos**.

**Análisis de varianza**

Ya sabemos que las distribuciones de las variables para cada una de las clases no son normales y que sólo tenemos dos clases. Normalmente, el análisis de varianza se aplica cuando tenemos más de dos clases para ver si la varianza entre las dos distribuciones es la misma o difieren. Y en caso de que difieran, entre cuales de las clases.
Aunque sabemos que no podemos aplicar **ANOVA** en este caso ya que las distribuciones no son normales vamos a aplicarlo para ver que nos dice sobre la varianza entre las clases:

```{r chunk_3_2_5}
anova(lm (breast_cancer_clean$clump.thick ~ breast_cancer_clean$class))
```

Nos sale un *p-valor* muy inferior a 0.05, con lo cual podemos rechazar la hipotésis nula y asumir que la varianza de las dos clases son distintas.

En este caso una manera muy sencilla de ver que las distribuciones son muy distintas es usando boxplots:

```{r chunk_3_2_6}
par(mfrow=c(3,3))
for (i in 1:9) {
  boxplot(breast_cancer_clean[[i]] ~ breast_cancer_clean$class, ylab = colnames(breast_cancer_clean)[i], xlab = "Tipo de Cancer", col = c("darkblue", "red"))
}
```

Para finalizar, vamos a realizar el **análisis de varianza** utilizando un **método no parámetrico** como el de Kruskal:


```{r chunk_3_2_7}
cancer.kruskal = kruskal.test(clump.thick~class,data=breast_cancer_clean)
```

```{r chunk_3_2_8}
print(cancer.kruskal)
```

El *p-valor* es inferior a 0.05, con lo que la varianza de las dos clases son distintas.


### 3.3. Aplicación de pruebas estadísticas (tantas como sea posible) para comparar los grupos de datos.

* **Test de dos muestras**

Podemos usar el **test de Wilcoxon** que utiliza métodos no parámetricos para probar que las distribuciones de las dos clases de cancer son distintas:

```{r chunk_3_3_1}
wilcox.test(breast_cancer_clean$clump.thick ~ breast_cancer_clean$class)
```

El *p-valor* es inferior a 0.05, con lo que rechazamos la hipotesis nula.
 
* **Test de una muestra**

Podemos intentar probar que la media del clump.thick de las muestras de *cancer maligno* son distintas a la media de las muestras de *cancer benigno*.

```{r chunk_3_3_2}
 wilcox.test(benign$clump.thick, mu = mean (malign$clump.thick))
```

El *p-valor* es inferior a 0.05, con lo que rechazamos la hipotesis nula.


* **Test de Barlett**

Podemos usar el test de Barlett para comprobar para probar si k muestras provienen de poblaciones con la misma varianza. A las varianzas iguales a través de las muestras se llama *homogeneidad de varianzas*. El análisis de la varianza ANOVA, suponen que las varianzas son iguales en todos los grupos o muestras. La prueba de Bartlett se puede utilizar para verificar esa suposición.

```{r chunk_3_3_3}
bartlett.test(breast_cancer_clean$clump.thick ~ breast_cancer_clean$class)
```

* **Regresión lineal múltiple**

Generando un modelo de regresión multiple podremos analizar el nivel de influencia de un conjunto de variables respecto a otra. Se estudiará que variable ejerce mayor influencia positiva y mayor influencia negativa sobre la variable dependiente.

El estudio se divide en los grupos de muestras benignas y malignas.

Por tanto, vamos generando modelos modificando la variable dependiente en cada caso.

**Regresión multiple en muestras benignas**

*clump.thick*
```{r chunk_rlm_clump.thick2}
sort(glm(clump.thick ~ uniform.size + uniform.shape + adhesion + epithelial.cell.size + bare.nuclei + bland.chromatin + nucleoli + mitoses, data = benign)$coefficients)
```

Las variables **mitoses**, **bare.nuclei** y **epithelial.cell.size** ejercen una influencia negativa, es decir, cuando aumentan **clump.thick** disminuye. La variable que meyor influencia positiva ejerce es **adhesion**

*uniform.size*
```{r chunk_rlm_uniform.size2}
sort(glm(uniform.size ~ clump.thick + uniform.shape + adhesion + epithelial.cell.size + bare.nuclei + bland.chromatin + nucleoli + mitoses, data = benign)$coefficients)
```

Todas las variables ejercen influencia positiva en **benign**, pero la que ejerce mayor influencia es **uniform.shape**.

*adhesion*
```{r chunk_rlm_adhesion}
sort(glm(adhesion ~ clump.thick + uniform.shape + uniform.size + epithelial.cell.size + bare.nuclei + bland.chromatin + nucleoli + mitoses, data = benign)$coefficients)
```

**bland.chromatin** ejerce influencia negativa sobre **adhesion**, pero en un coeficiente muy bajo. La que mayor influencia positiva ejerce es **bare.nuclei**.

*uniform.shape*
```{r chunk_rlm_uniform.shape2}
sort(glm(uniform.shape ~ clump.thick + adhesion + uniform.size + epithelial.cell.size + bare.nuclei + bland.chromatin + nucleoli + mitoses, data = benign)$coefficients)
```

Tanto **mitoses** como **bland.chromatin** ejercen influencia negativa, pero los coeficientes son muy bajos. La varaible que mayor influencia positiva ejerce es **uniform.size**.


*epithelial.cell.size*
```{r chunk_rlm_epithelial.cell.size}
sort(glm(epithelial.cell.size ~ clump.thick + adhesion + uniform.size + uniform.shape + bare.nuclei + bland.chromatin + nucleoli + mitoses, data = benign)$coefficients)
```


**mitoses**, **bland.chromatin** y **clump.thick** influyen de forma negativa. **nucleoli** es la que más influye positivamente.


*bare.nuclei*
```{r chunk_rlm_bare.nuclei2}
sort(glm(bare.nuclei ~ clump.thick + adhesion + uniform.size + uniform.shape + epithelial.cell.size + bland.chromatin + nucleoli + mitoses, data = benign)$coefficients)
```

La variable **clump.thick** influye negativamente, mientras que **uniform.size** es que la influye de forma positiva con más fuerza.


*bland.chromatin*
```{r chunk_rlm_bland.chromatin}
sort(glm(bland.chromatin ~ clump.thick + adhesion + uniform.size + uniform.shape + epithelial.cell.size + bare.nuclei + nucleoli + mitoses, data = benign)$coefficients)
```

**mitoses**, **epithelial.cell.size**, **uniform.shape** y **adhesion** influyen de forma negativa sobre **bland.chromatin** mientras que **nucleoli** es la que influye de forma más positiva.

*nucleoli*
```{r chunk_rlm_nucleoli2}
sort(glm(nucleoli ~ clump.thick + adhesion + uniform.size + uniform.shape + epithelial.cell.size + bare.nuclei + bland.chromatin + mitoses, data = benign)$coefficients)
```

Todas influyen de forma positiva en **nucleoli**, pero la que lo hace con mayor fuerza es **uniform.size**

*mitoses*
```{r chunk_rlm_mitoses2}
sort(glm(mitoses ~ clump.thick + adhesion + uniform.size + uniform.shape + epithelial.cell.size + bare.nuclei + bland.chromatin + nucleoli, data = benign)$coefficients)
```

Por último, **mitoses** se ve influenciada negativamente por **epithelial.cell.size**, **bland.chromatin**, **uniform.shape** y **clump.thick**. **bare.nuclei** es la que influye de forma más positiva.


En el caso de muestras de tumor benigno, observamos que la mayoría de las variables tienen una influencia positiva en el resto. Aquellas que influyen de forma negativa lo hacen en coeficientes tan bajos que el cambio apenas es apreciable. La mayor influencia entre variables la encontramos entre **uniform.size** y **uniform.shape** de forma bidireccional. Por tanto a medida que aumente una debería aumentar la otra. 



**Regresión multiple en muestras malignas**

Se repite el proceso para las muestras de tumor malignas. En este caso se incluirá un comentario final, ya que los resultados de los coeficientes de cada modelo de regresión hablan por si solos en cada caso. 

*clump.thick*
```{r chunk_rlm_clump.thick}
sort(glm(clump.thick ~ uniform.size + uniform.shape + adhesion + epithelial.cell.size + bare.nuclei + bland.chromatin + nucleoli + mitoses, data = malign)$coefficients)
```

*uniform.size*
```{r chunk_rlm_uniform.size}
sort(glm(uniform.size ~ clump.thick + uniform.shape + adhesion + epithelial.cell.size + bare.nuclei + bland.chromatin + nucleoli + mitoses, data = malign)$coefficients)
```

*adhesion*
```{r chunk_rlm_adhesion2}
sort(glm(adhesion ~ clump.thick + uniform.shape + uniform.size + epithelial.cell.size + bare.nuclei + bland.chromatin + nucleoli + mitoses, data = malign)$coefficients)
```

*uniform.shape*
```{r chunk_rlm_uniform.shape}
sort(glm(uniform.shape ~ clump.thick + adhesion + uniform.size + epithelial.cell.size + bare.nuclei + bland.chromatin + nucleoli + mitoses, data = malign)$coefficients)
```

*epithelial.cell.size*
```{r chunk_rlm_epithelial.cell.size2}
sort(glm(epithelial.cell.size ~ clump.thick + adhesion + uniform.size + uniform.shape + bare.nuclei + bland.chromatin + nucleoli + mitoses, data = malign)$coefficients)
```

*bare.nuclei*
```{r chunk_rlm_bare.nuclei}
sort(glm(bare.nuclei ~ clump.thick + adhesion + uniform.size + uniform.shape + epithelial.cell.size + bland.chromatin + nucleoli + mitoses, data = malign)$coefficients)
```

*bland.chromatin*
```{r chunk_rlm_bland.chromatin2}
sort(glm(bland.chromatin ~ clump.thick + adhesion + uniform.size + uniform.shape + epithelial.cell.size + bare.nuclei + nucleoli + mitoses, data = malign)$coefficients)
```

*nucleoli*
```{r chunk_rlm_nucleoli}
sort(glm(nucleoli ~ clump.thick + adhesion + uniform.size + uniform.shape + epithelial.cell.size + bare.nuclei + bland.chromatin + mitoses, data = malign)$coefficients)
```

*mitoses*
```{r chunk_rlm_mitoses}
sort(glm(mitoses ~ clump.thick + adhesion + uniform.size + uniform.shape + epithelial.cell.size + bare.nuclei + bland.chromatin + nucleoli, data = malign)$coefficients)
```

Los resultados obtenidos en el caso de muestras tumorales malignas son similares a las obtenidas en muestras benignas. Las variables que ejercen influencia negativa lo hacen a un nivel muy bajo. De nuevo encontramos que **uniform.shape** y **uniform.size** ejercen una gran influencia la una respecto a la otra de forma bidireccional.


* **Estudio de las medias de las poblaciones**

Al dividir el conjunto de datos originales en 2 subconjuntos claramente diferenciados, hemos aplicado clustering de forma manual. Al igual que lo haría un algoritmo de clustering para crear los centros de los grupos, podemos realizar una comparación de las medias de las variables de ambos grupos para comprobar si podemos establecer alguna diferencia entre los grupos:

```{r}
benign_mean<-sapply(benign[,1:9], mean)
malign_mean<-sapply(malign[,1:9], mean)

(df<-as.data.frame(t(cbind(benign_mean, malign_mean))))
```

A continuación mostramos de menor a mayor la diferencia entre las variables de ambas poblaciones:

```{r}
sort(sapply(df, diff))
```

Las muestras de tumores benignos y malignos muestran las mayores diferencias respecto a las variables **bare.nuclei**, **uniform.size** y **uniform.shape**, por lo que se puede usar esta información, por lo que podríamos concluir que estos son los 3 atributos más determinantes para distinguir una muestra. Por otro lado y siguiente este razonamiento, la variable **mitoses** sería la menos determinante a la hora de distinguir una muestra de otra.

## 4. Conclusiones del Análisis

Después de analizar el dataset podemos concluir que:

- Las observaciones del dataset no están *equilibradas*. Tenemos más observaciones de cáncer benigno que de cáncer maligno
- Las 9 variables numéricas que componen el dataset están poco correlacionadas entre ellas, salvo el uniform.size y uniform.shape
- Las distribuciones de las observaciones de cáncer benigno y cáncer maligno son distintas. La distribuciones de cáncer benigno tienen números bajos en todas las variables, mientras que las distribuciones de cáncer maligno están más distribuidas en el rango de las variables y en algunos casos toma valores altos como es el *bare.nuclei*
- Se observan grandes diferencias entre los centros de las poblaciones, lo que nos da una idea de los atributos que pueden ser más significativos a la hora de distinguir muestras.
- Las distribuciones de las variables no siguen una distribución normal y son distintas entre ellas para todas las variables. Tanto la media como la varianza.
- Al no ser distribuciones normales hemos necesitado de tests no paramétricos para comprobarlo

