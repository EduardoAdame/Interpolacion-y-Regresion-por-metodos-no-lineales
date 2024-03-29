# Interpolacion-y-Regresion-por-metodos-no-lineales
De manera general se aplican métodos de regresión e interpolación tanto lineal como no lineal para distintas bases de datos, en particular, se aplican  los siguientes métodos: Interpolación por polinomios de Lagrange, interpolación con splines cúbicos, modelos de regresión lineal, polinomial (cuadrático y cúbico), regresión por splines cúbicos. 
---
title: "Trabajo Final Análisis Numérico"
author: "Adame Serrano Eduardo"
output: 
html_document:
    theme: darkly
    toc: FALSE
    toc_depth: 3
    toc_float:
      collapsed: TRUE
      smooth_scroll : TRUE
      number_sections : TRUE
    df_print: paged
---
# {.tabset .tabset-fade .tabset-pills}
Se analizarán distintas bases de datos realizadas por el INEGI.

## Base 1 {.tabset .tabset-fade .tabset-pills}



```{r include=FALSE}
library(tidyverse)
library(lattice)
library(splines)
library(ggplot2)
library(tidyr)
library(Metrics)
theme_set(theme_classic())
```


### Introducción

Leemos la primer base de datos.

```{r echo=FALSE}
Base_1 <- read.csv("EmployedPercentageRate_Indicadores20210207143651.csv")
Base_1
```





Guardamos las variables de la base a utilizar, por una parte los porcentajes,

```{r echo=FALSE}
porcentajes <- Base_1$X[5:65]
porcentajes
```

y por otra parte debido a que tenemos 61 trimestres, comprendidos desde 2005 a 2020, entonces será más fácil gráficamente trabajar con un vector de 2005 a 2020 con saltos de 0.25 (3 meses).

```{r echo=FALSE}
trimestres <- seq(2005,2020,0.25)
trimestres
```



Hasta ahora estos son los datos con los que trabajaremos,

```{r echo=FALSE}
porcentajes <- c(as.numeric(paste(porcentajes)))
porcentajes
trimestres
```
guardamos ambos datos en un data frame al cual llamamos `base1_depurada`,
```{r echo=FALSE}
Base1_depurada<- data.frame(Trimestres = trimestres, Porcentajes = porcentajes)
```

`porcentajes`  es el porcentaje de datos de la base, `trimestres` es el tiempo en trimestres de 2005 a 2020, $n$ es la longitud de `porcentajes`.


Ahora, veamos si nuestros datos presentan una relación lineal graficando un diagrama de dispersión de los mismos,

```{r echo=FALSE, message=FALSE}
ggplot(Base1_depurada, aes(trimestres,porcentajes))+ geom_point() + ggtitle("Gráfico de Dispersión") + stat_smooth() + labs(x = "Trimestres" , y ="Porcentajes")
```


del diagrama anterior notamos que nuestros datos no siguen una tendencia lineal entre si.




Ahora, para ver que tanto varían nuestros datos entre si con los diferentes análisis utilizaremos las métricas **RMSE** y **$R^2$** para comparar los diferentes modelos. 

Recordemos que el **RMSE** representa el error de predicción del modelo, es decir, la diferencia promedio entre los valores de resultado observados y los valores de resultado predichos o mejor conocido como el error cuadrático medio. Por otro lado el **$R^2$** representará la correlación entre los valores de resultado observados y predichos.

El mejor modelo será aquel que tenga el **RMSE** más bajo y el **$R^2$** más alto, lo cual indicará un bajo índice en el error de predicción y una alta correlación entre los resultados.


Estas métricas las usaremos para las distintas bases de datos a evaluar. 

###  Interpolación con Polinomios de Lagrange

El polinomio de interpolación de Lagrange es una reformulación del polinomio de interpolación de Newton. El método tolera las diferencias entre las distancias $x$ entre puntos.

El polinomio se construye a partir de las fórmulas

$$ 
\begin{align*}
f_n(x) &= \sum_{i=0}^nL_i(x)f(x_i)\\
L_i(x) &= \prod_{j=0,j\neq i}^n \frac{x-x_j}{x_i-x_j}.
\end{align*}
$$
La primera de ellas se conoce como el polinomio interpolador en la forma de Lagrange, y es la combinación lineal dada por $f_n(x)$.

La segunda ecuación se conoce como las bases polinómicas de Lagrange.

Donde una vez que se han seleccionado los puntos a utilizar, se generan la misma cantidad de términos que puntos.


Mencionado lo anterior, creamos la función que nos dará la interpolación de Lagrange
```{r}
set.seed(1234)
lagrange <- function(x,y,x0){ #x:datos en el eje horizontal, y:datos en el eje vertical, x0= punto donde se desea interpolar para conocer su valor
  y0 = 0 #Lo inicializamos en 0 pero será el valor de x interpolado, es decir,fungirá como la imagen de x a través de f=lagrange
  for(i in 1:length(x)){ #Inicializamos el índice donde comenzarán a correr los valores de xi
    Li = 1 #Será la primer basepolinómica de Lagrange (caso cuando el numerador coincide con el denominador es decir, cuando i = j)
    for(j in 1:length(x)){ #Inicializamos el índice donde comenzarán a correr los valores de xj
      if(i!= j){ #Condición para empezar a calcular los productos de las bases polinómicas y que sean distintos de 1
        Li = Li*(x0-x[j])/(x[i]-x[j]) #Expresión iterativa para calcular lasbases polinómicas
      }
    }
    y0 = y0+ Li*y[i] #Valor del polinomio interpolador de Lagrange del punto x0
  }
  return(y0) #Solo pedimos que exprese el valor resultante
}
```
Una vez creada veamos si nuestra función interpola correctamente los puntos de nuestra base de datos

```{r}
#Prueba
#Veamos los valores para el primer trimestre de 2005 y para el primer trimestre de 2020
porcentajes[1]
porcentajes[61]
#Ahora veamos que nuestra función arroja los mismos valores
lagrange(trimestres,porcentajes,2005)
lagrange(trimestres,porcentajes,2020)
#Por otro lado, si quisieramos interpolar el valor de algun tiempo entre dos valores en nuestra base de datos, por ejemplo el primer semestre de 2005, el cual se representa por 2005.5, y el  primer semestre de 2019, el cual se representa por 2020.5, podemos usar nuestra función de la siguiente manera
lagrange(trimestres,porcentajes,2005.5)
lagrange(trimestres,porcentajes,2019.5)
#Finalmente si quisieramos cualquier valor en un tiempo dentro de nuestra base de datos por ejemplo en el mes de febrero de 2019, podemos hacer lo siguiente
lagrange(trimestres,porcentajes,2019.2)
```


Ahora pondremos a prueba nuestra función para verificar que en realidad puede interpolar todos los valores de nuestra base de datos depurada.
```{r}
interpolaciones1=0 #Inicializamos en cero el vector donde guardaremos las interpolaciones
  interpolacion_1i<-lagrange(trimestres,porcentajes,seq(2005,2020,.25))
  interpolaciones1 = interpolaciones1 + interpolacion_1i
  interpolaciones1
```



Obtengamos la visualización de las interpolaciones anteriores y de la base de datos.
```{r}
plot(trimestres, porcentajes, xlim=c(trimestres[1],tail(trimestres,1)), ylim=c(26.5,29.5), xlab = "", ylab = "")#Ploteamos los porcentajes de nuestra base de datos
par(new = 'TRUE') #Creamos un plot encima del otro
plot(trimestres, interpolaciones1, xlim=c(trimestres[1],tail(trimestres,1)), ylim=c(26.5,29.5), xlab = "Trimestres", ylab = "Porcentajes", col= 'steelblue', type = 'l') #Ploteamos las interpolaciones obtenidas 

```


Notamos que ambos gráficos se muestran encimados para asegurarnos que la función cumpla su objetivo por una parte los datos sin ningun tipo de linea que los una están representados por circulos, mientras que el polinomio de Lagrange que se adapta a los datos los une de color azul.


###  Interpolación con splines cúbicos

Un spline es una función definida a trozos sobre intervalos de $\mathbb{R}$ que se unen entre si obedeciendo a ciertas condiciones de regularidad. La terminología fue introducida por I. J. Schoenberg (1946).

El nombre de spline proviene del instrumento mecánico que consiste en un alambre flexible que puede ser utilizado para dibujar curvas suaves a través de puntos asignados. Generalmente empleados para dibujo técnico en las industrias aeronáuticas, automotriz, naval, etc. 


Un spline cúbico utiliza un polinomio de tercer gradopara cada intervalo entre nodos, 

$$ f_i(x) = a_ix^3 + b_ix^2 + c_ix + d_i$$

 
Para $n+1$ datos (nodos $i =( 0 ,1, 2, \ldots,n)$ existen $n$ intervalos y por lo tanto $4n$ incógnitas a evaluar.

Una manera sencilla de obtener los coeficientes de los splines cúbicos es utilizando el concepto de la segunda derivada. El resultado es una ecuación cúbica para cada intervalo: 


$$
\begin{align*}
f_i (x) &= \frac{f''(x_{i-1})}{6(x_i - x_{i-1})}(x_i-x)^3 + \frac{f''(x_{i})}{6(x_i - x_{i-1})}(x-x_{i-1})^3 \\
&+\left[   \frac{f(x_{i-1})}{(x_i - x_{i-1})} -  \frac{f''(x_{i-1})(x_i -x_{i-1})}{6}  \right](x_i-x) \\
&+\left[   \frac{f(x_{i})}{(x_i - x_{i-1})} -  \frac{f''(x_{i-1})(x_i -x_{i-1})}{6}  \right](x-x_{i-1}) \\
\end{align*}
$$
Esta ecuación contiene solo dos incógnitas, las segundas derivadas en los extremos de cada intervalo $f_i''(x_{i-1})$ y $f_i''(x_i)$


Las segundas derivadas se evaluán tomando la condición de que las primeras derivadas deben ser continuas en los nodos: $f_i'(x_i) = f_{i+1}'=(x_i)$. Derivando la expresión para $f_i(x)$ y $f_{i+1}(x)$ e igualando en la posición $x_i$ del nodo interio, se obteien una expresión algebraíca que permite hallar las segundas derivadas en cada nodo interior:

$$ \begin{align*}
&(x_i-x_{i-1})f''(x_{i-1})+ 2(x_{i+1} - x_{i-1})f''(x_i) + (x_{i+1} -x_i)f''(x_{i+1}) \\
&= \frac{6}{x_{i+1} -x_i}[f(x_{i+1})-f(x_i)] + \frac{6}{x_i - x_{i-1}}[f(x_{i-1})-f(x_i)]
\end{align*}
$$

Además, en los nodos extremos (primero y último) se tendrá que la segunda derivada es cero, la ecuación anterior solo involucra $n-1$ incógnitas y el sistema es tridiagonal.


Ahora mencionado lo anterior, encontramos que en R existe una función que realiza lo mencionado y haremos uso de la función `splinefun` para generar el spline cubico; `splinefun` devuelve una función con argumentos formales `x` y `deriv`, este último por defecto a cero. Esta función se puede utilizar para evaluar el spline cúbico de interpolación (deriv = 0), o sus derivados (deriv = 1, 2, 3) en los puntos $x$, donde la función spline interpola los puntos de datos especificados originalmente.

Las entradas pueden contener valores perdidos que se eliminan, por lo que se requiere al menos un par completo $(x, y)$. Si tenemos `método = "fmm"`, el spline utilizado es el de Forsythe, Malcolm y Moler (se ajusta a un cúbico exacto a través de los cuatro puntos en cada extremo de los datos, y esto se utiliza para determinar las condiciones finales).


Hacemos uso de la función mencinada anteriormente para los datos de la base de datos depurada (`base1_depurada`)
```{r}
func_spline= splinefun(x=trimestres,y=porcentajes,method="fmm",ties=mean)
porcentajes_spline_bim <- func_spline(seq(2005,2020,0.2))
porcentajes_spline_bim #Se generan los splines obtenidos para bimestres
```

Guardamos los porcentajes de la interpolación obtenida por spline cúbico para bimestres, primero generando los bimestres
```{r}
bimestres <- seq(2005,2020,.2)
```
Guardando los bimestres y las observaciones
```{r}
base_splines_bimestres <- data.frame(Bimestres = bimestres, Porcentajes = porcentajes_spline_bim)
base_splines_bimestres
```

Otra forma de interpolar los datos es con la función `spline` la cual devuelve una lista que contiene los componentes $x$ e $y$ que dan las ordenadas donde tuvo lugar la interpolación y los valores interpolados. Esta interpolación suele contener un mayor numero de observaciones.

```{r}
interp_spline <- spline(trimestres, y = porcentajes, n = 3*length(trimestres), method = "fmm",
       xmin = min(trimestres), xmax = max(trimestres), ties = mean)
interp_spline
```
Guardamos en un data frame la interpolación que generó la función anterior.
```{r}
interp_spline_df <- data.frame(interp_spline) #Este data frame contendrá dos tipos de observaciones, los puntos x donde se llevó a cabo la interpolación y el valor y de la misma.
```

Veamos gráficamente la interpolación por el método de splines cúbicos

```{r}
#Generamos los nudos para la gráfica
nudos_spline <- quantile(interp_spline_df$x, p = c(0.25, 0.5 ,0.75))
```

```{r echo=TRUE}
#Generamos la gráfica
ggplot(interp_spline_df, aes(x, y))+ 
  geom_point() + ggtitle("Interpolación con spline cúbico") +labs(x = "Bimestres" , y ="Porcentajes")+
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL,  , knots = nudos_spline, degree = 3))
```



Ahora para hacer proyecciones futuras utilizando la interpolación por splines generamos el modelo, para ello generamos el vector de tiempo donde queremos realizar las proyecciones,


```{r}
trimestres_proyecciones <- seq(2020,2035,0.25)
```
creamos los nudos para los segmentos polinomiales

```{r}
nudos_proyecciones <- quantile(trimestres_proyecciones, p = c(0.25, 0.5 ,0.75))
```
Seguido del modelo
```{r}
modelo_spline_cub_interp <- lm (porcentajes ~ bs(trimestres_proyecciones, knots = nudos_proyecciones), data = Base1_depurada)  #La función bs genera la matriz base B-spline para una spline polinomial.
```
Ahora las proyecciones
```{r}
proyeccion_splinescub <- modelo_spline_cub_interp %>% predict(data.frame(x = seq(2020,2035,.25))) #predict es una función genérica para predicciones a partir de los resultados de varias funciones de ajuste de modelos. La función invoca métodos particulares que dependende la clase del primer argumento.
proyeccion_splinescub
```

Guardamos en un data frame los trimestres y la proyección que generó la función anterior.

```{r}
proyeccion_spline_df <- data.frame(Trimestres = trimestres_proyecciones, Porcentajes = proyeccion_splinescub)
proyeccion_spline_df
```


Veamos gráficamente la proyección por el método de splines cúbicos


```{r echo=FALSE}
ggplot(proyeccion_spline_df, aes(trimestres_proyecciones  ,  proyeccion_splinescub ))+ 
  geom_point() + ggtitle("Proyección con spline cúbico para trimestres") + labs(x = "Trimestres",y = "Proyecciones") +
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL,  , knots = nudos_proyecciones, degree = 3))
```

Notamos que el spline cúbico modela perfectamente nuestros datos interpolados y proyectados previamente.

Ahora, si quisieramos interpolar las proyecciones por bimestres, lo haríamos de la siguiente forma
Generamos los periodos de tiempo
```{r}
bimestres_proyecciones <- seq(2020,2035,0.2)
```
Ahora creamos los nudos para los segmentos polinomiales.
```{r}
nudos_proyecciones_bim<- quantile(bimestres_proyecciones, p = c(0.25, 0.5 ,0.75))
```
Seguido del modelo
```{r}
modelo_spline_cub_interp_bim<- lm (porcentajes_spline_bim ~ bs(bimestres_proyecciones, knots = nudos_proyecciones_bim), data = Base1_depurada)
modelo_spline_cub_interp_bim
```
Ahora las proyecciones
```{r}
proyeccion_splinescub_bim <- modelo_spline_cub_interp_bim %>% predict(data.frame(x = seq(2020,2035,.2)))
proyeccion_splinescub_bim
```

Guardamos en un data frame los bimestres y la proyección que generó la función anterior.

```{r}
proyeccion_spline_df_bim <- data.frame(Bimestres = bimestres_proyecciones, Porcentajes = proyeccion_splinescub_bim)
proyeccion_spline_df_bim
```
Veamos gráficamente la proyección por el método de splines cúbicos para bimestres
```{r echo=FALSE}
ggplot(proyeccion_spline_df_bim, aes(bimestres_proyecciones  ,  proyeccion_splinescub_bim ))+ 
  geom_point() + ggtitle("Proyección con spline cúbico para bimestres") + labs(x = "Bimestres",y = "Proyecciones") +
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL,  , knots = nudos_proyecciones, degree = 3))
```

Notamos como la cantidad de observaciones aumenta (puntos en el spline cúbico) y claramente nuestro spline se adapta de mejor manera a nuestros datos, es decir, tenemos una gráfica más completa.


### Regresión lineal simple
La Regresión Lineal Simple es un modelo de regresión en donde una función lineal representa la relación existente entre una variable dependiente y su respectiva variable independiente.

En algunos casos,la verdadera relación entre el resultado y una variable predictora puede no ser lineal. 


La ecuación del modelo de regresión lineal estándar se puede escribir como:

$$Y_i = \beta_0 + \beta_1 X_i + \epsilon_i$$

En donde $Y$ es la variable dependiente, $\epsilon_i$ un error (residual) observacional aleatorio cuya esperanza es cero  y varianza es $\sigma^2$, es decir, son variables aleatorias con las cuales se puede suponer una distribucion Normal con parámetros mencionados anteriormente , además $\beta_0$ y $\beta_1$ son los coeficientes de la recta (ordenada al orígen y pendiente, respectivamente) que, bajo algún criterio de minimización como el de mínimos cuadrados, ofrece el mejor ajuste a los datos de entrada. A efectos de entender más claramente el modelo, apliquemos la Regresión Lineal Simple a nuestros datos.

Para calcular el modelo de regresión lineal, construimos el modelo,y recordemos que el aplicar una regresión sobre estos datos implica obtener la línea recta que mejor ajuste la relación existente entre la variable independiente y la variable dependiente. Para ello, vamos a crear un modelo lineal haciendo uso de la función `lm` del paquete `stats`.

```{r}
# Construyendo el modelo
modelo_lineal <- lm(porcentajes ~ trimestres, data = Base1_depurada)
```
Una vez creado el modelo lineal, podemos explorar los resultados y calidad del ajuste haciendo uso de la función `summary`

```{r}
# Nos brinda un resumen de la regresión
summary(modelo_lineal)
```
Entre los datos de interés que ofrece la función `summary` están la estadística de los residuales (es decir, las distancias que existen entre los valores de las variables del conjunto de datos y sus proyecciones con la curva que los ajusta), los valores de los coeficientes obtenidos, el p-value (relevancia estadística de la variable independiente como elemento predictivo, y que aparece representado en el reporte como $Pr(>|t|))$,  los grados de libertad `DF` así como los valores de calidad del ajuste $R^2$, que en este caso es de 0.03846.

Revisamos el modelo
```{r}
anova(modelo_lineal)
```
`anova` nos proporciona los grados de libertad `Df` en este caso de 59, el cálculo de la suma de cuadrados `Sum sq`, los cuadrados medios `Mean sq`cuyo valor es de 0.8741 , el estadístico `F` de 2.3604 y el `pi-value  = Pr(>F)` de 0.1298, de esta forma se observa que se rechaza la hipótesis nula, con un nivel de confianza del 87%, por lo que se tiene evidencia de que $\beta_1 \neq 0$, es decir, existe una asociación lineal entre la variable de respuesta `trimestres` y la variable explicativa `porcentajes`.  


Verificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_lineal)
```


`Residual vs Fitted`: Deberían estar distribuidos aleatoriamente alrededor de una linea horizontal que representa un error residual de cero ($\epsilon_i =0$)

`Normal Q-Q`: Debería sugerir que los errores residuales se distribuyen normalmente.

`Scale-Location` Muestra la raiz cuadrada de los residuos estandarizados, como una funcion de los valores ajustados. No deberia existir una tendencia clara en ese trama.

`Residual vs Leverage` Las distancias mas grandes que 1 son sospechosos y sugieren la presencia de un valor atipico posible. y su eliminacion podria tener efectos sobre la regresion.


Por otro lado, podemos determinar explícitamente la ecuación de predicción obtenida por la regresión, de manera muy sencilla podemos utilizar la funcion  `summary`, la cual nos dará explícitamente los valores de $\beta_0$ y $\beta_1$,de modo que dicha ecuación estará dada por


$$Porcentajes = 82.41541 + (-0.02719)*Trimestre$$
La interpretacion de los resultados es:

En promedio, cada incremento en una unidad de los trimestres , corresponde a un decremento del porcentaje de 0.02719, asi mismo que un trimestre cuyo valor sea de cero (origen de y), tenga una porcentaje de 82.41541, esto obviamente no parece posible. Dado que la regresion estima como logramos ver estimaciones mas allá de los valores observados.

Generamos la ecuación de la recta
```{r}
y_gorro1 = 82.41541 + (-0.02719)*trimestres
y_gorro1
```

Gráficamente se ve de la siguiente manera

```{r}
plot(trimestres,porcentajes)
lines(trimestres,y_gorro1, type = "l", col = "darkred")
```

La recta pendiente que expresamos anteriormente, también se puede obtener manualmente de la siguiente manera

```{r}
#Calculamos la media de nuestros datos
media_p1 = mean(porcentajes) 
media_t1 = mean(trimestres)
```

Recordemos que 
$$\begin{align*} \hat{\beta_1} &= \frac{S_{XY}}{S_{XX}} \\
\\
\hat{\beta_0} &= \overline{Y} - \hat{\beta_1}\overline{X}
\end{align*} $$

```{r}
#Calculamos las ecuaciones normales
Sxy1 = sum( (porcentajes-media_p1)*(trimestres-media_t1) )
Sxy1

Sxx1 = sum( (trimestres-media_t1)^2 )
Sxx1
```
Finalmente
```{r}
Bp1 = Sxy1 / Sxx1
Bp1

Bp0 = media_p1 - Bp1*media_t1
Bp0

#porcentaje = 82.4154102179 - 0.0271939136393*trimestre
```

Generamos la ecuación
```{r}
y_gorrom1 = Bp0 + Bp1*trimestres
y_gorrom1
```
Hacemos el plot
```{r}
plot(trimestres,porcentajes)
lines(trimestres,y_gorrom1, type = "l", col = "darkred")
```



Una vez que tenemos el modelo construido, vamos a crear un vector de predicciones basado en el propio conjunto de observaciones, y con este vector podremos visualizar la curva de ajuste de los datos. Para obtener las predicciones, hacemos uso de la función `predict`


```{r}
# Predicciones
predicciones_lineales <- modelo_lineal %>% predict(Base1_depurada)
predicciones_lineales
```

Otra forma de hacer proyecciones sería si deseáramos tener una predicción particular para valores cualesquiera de $x$, podemos hacerlo de la siguiente manera:

```{r}
predicciones_lineal <- predict(modelo_lineal, data.frame(x = seq(2020,2035,.25)))
predicciones_lineal
```

Guardamos en un data frame los datos obtenidos con sus respectivos periodos.
```{r}
proyecciones_df <- data.frame(Trimestres = seq(2020,2035,.25) , Porcentajes = predicciones_lineal)
proyecciones_df
```

En este caso se obtiene una predicción para los trimestres comprendidos entre 2005 a 2035.


En este punto es importante mencionar que la función `predict` espera como argumento de la variable independiente un dataframe, por lo que es necesario convertir el valor seleccionado a tal tipo de dato como se observa en el código anterior.


La forma gráfica de las predicciones anteriores se presenta a continuación.

```{r}
ggplot() + geom_point(data =proyecciones_df , aes(x = Trimestres, y = Porcentajes), size = 0.9) + 
  geom_line(aes( x = proyecciones_df$Trimestres, y =predicciones_lineal), color = "red") +
  xlab("Trimestres") + 
  ylab("Porcentajes") + 
  ggtitle("Curva de Ajuste sobre Conjunto de Observaciones")
```

Lo cual claramente representa la ecuación de predicción obtenida anteriormente, cuya pendiente es negativa.




Ahora construímos la gráfica adecuada para el modelo

```{r}
ggplot() + geom_point(data = Base1_depurada, aes(x = trimestres, y = porcentajes), size = 0.9) + 
  geom_line(aes( x = Base1_depurada$Trimestres, y =predicciones_lineales), color = "red") +
  xlab("Trimestres") + 
  ylab("Porcentajes") + 
  ggtitle("Curva de Ajuste sobre Conjunto de Observaciones")
```


Otra forma sería


```{r echo=FALSE}
ggplot(Base1_depurada, aes(trimestres, porcentajes))+ 
  geom_point() + ggtitle("Regresión Lineal") +
  stat_smooth(method = lm,formula = y~x)
```

Finalmente veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste $R^2$


```{r}
modelo_lineal_df <- data.frame(RMSE = rmse(predicciones_lineales,Base1_depurada$Porcentajes), 
           R2 = round(cor(predicciones_lineales,Base1_depurada$Porcentajes)^2, 10000))
modelo_lineal_df
```
 
El valor de calidad de ajuste 0.03846746 indica una baja correlación y el error cuadrático medio 0.5984499 es bastante alto, por lo tanto, podemos decir que es un ajuste no indicado para los datos por parte del modelo obtenido.




### Regresión polinomial cuadrática
Este es el enfoque simple para modelar relaciones no lineales
Sabemos que nuestros datos no presentan una relación lineal pero al inspeccionar la gráfica vemos que los datos parecen seguir un comportamiento polinomial

Cuando se habla de regresión polinomial, se busca producir una ecuación de ajuste de las variables dependiente - independiente que tenga la forma $$y=a+bx+cx^2+...+Nx^n,$$ en donde $n$ es el grado del polinomio. Es decir, se introducen al modelo términos polinomiales  (cuadrados, cubos, etc) de la variable independiente hasta lograr el mejor ajuste a los datos.



Ahora bien, ya que los conjuntos de datos están creados, para realizar la regresión polinomial vamos a introducir en primer lugar un término de segundo grado a nuestros datos a fin de que el regresor pueda ser polinomial.

En R para crear un predictor $x^2$ es recomendable utilizar la funcion $I()$. De la siguiente manera: $I(x^2)$. Esto eleva $x$ a la potencia 2.


Ahora, podemos construir nuestro modelo de regresión polinomial:


```{r}
set.seed(1234)
modelo_cuadratico <- lm(porcentajes ~ trimestres + I(trimestres^2), data = Base1_depurada)
```
Otra forma es 
```{r}
lm(porcentajes ~ poly(trimestres,2,raw = TRUE), data =  Base1_depurada)
```

Como puede verse, en este caso la fórmula del primer argumento `porcentajes ~ trimestres + I(trimestres^2)` incluye la nueva columna.

Al aplicar la función `summary` a nuestro nuevo regresor tenemos:

```{r}
summary(modelo_cuadratico)
```
Como puede verse, tanto la variable `trimestres` como la variable `trimestres^2` son relevantes a efectos de la predicción (valores pequeños del p-value) y se obtiene un $R^2$ de 0.1454.


Revisamos el modelo
```{r}
anova(modelo_cuadratico)
```

Verificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_cuadratico)
```
En la gráfica superior izquierda claramente nuestros datos se alejan de la linea horizontal que representa un error residual cero, por otro lado, en el gráfico superior izquierdo podemos ver como las colas de nuestros datos son pesadas, pues se alejan de la linea  diagonal punteada que supondría una distribución normal de los residuales y finalmente la gráfica inferior derecha indicaría la presencia de valores atípicos en el modelo.



Ahora, obtengamos las predicciones para el conjunto de observaciones y elaboremos la gráfica del modelo obtenido

```{r}
#Predicciones
predicciones_cuadradas <- predict(modelo_cuadratico,data.frame(x = seq(2020,2035,.25)))
predicciones_cuadradas
```


```{r echo=FALSE}
ggplot(Base1_depurada, aes(trimestres,porcentajes))+ 
  geom_point() + ggtitle("Regresión cuadrática")+ 
stat_smooth(method = lm, formula = y~poly(x,2,raw = TRUE))
```

En efecto, vemos que la curva de ajuste producida por la regresión sigue la forma no lineal de los datos observados aunque no logra representar una buena regresión entre ellos, para esto veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste.

```{r}
modelo_cuadratico_df <- data.frame(RMSE = rmse(predicciones_cuadradas,Base1_depurada$Porcentajes), 
           R2 = round(cor(predicciones_cuadradas,Base1_depurada$Porcentajes)^2, 10000))
modelo_cuadratico_df
```

El valor de calidad de ajuste 0.1453784 indica una baja correlación entre los datos, aunque en comparación con el modelo de regresión lineal simple (0.03846746), es más alto y, por otro lado, el error cuadrático medio 0.5641996	 es un poco más bajo que el del modelo de regresión lineal (0.5984499) por tanto podemos decir que el modelo de regresión polinomial cuadrado es un poco más indicado para los datos por parte del modelo obtenido.


Ahora, ¿qué ocurre si incorporamos más órdenes al polinomio? ¿Nuestro modelo de regresión mejorará? Hagamos la prueba.

### Regresion polinomial cúbica

Con la misma teoría estudiada para la regresión polinomial cuadrada creamos el modelo cúbico.

Tenemos que tomar en cuenta que para la interpretación de los trimestres lo haremos de 1 a 61 entendiendo como 1 el primer trimestre de 2005 y como 61 el primer trimestre de 2020, esto debido a que al momento de hacer el modelo cubico si tomáramos las fechas como "2005" (numérico), tendríamos que al momento de realizar el cúbo de las fechas la memoria no alcanza a guardar el resultado y los introducirá como NA. 

Creamos dichos trimestres.
```{r}
trim1 <- seq(1,61)
trim2 <- trim1^2
trim3 <- trim1^3
```

Generamos el modelo cúbico
```{r}
modelo_cubico <- lm(porcentajes ~ trim1 + trim2 +trim3, data =  Base1_depurada)
```

Analizamos sus propiedades 
```{r}
summary(modelo_cubico)
```
Como puede verse, tanto la variable `trimestres` ,la variable `trimestres^2`  y la variable `trimestres^3`son relevantes a efectos de la predicción (valores pequeños del p-value) y se obtiene un $R^2$ de 0.1744.

Revisamos el modelo
```{r}
anova(modelo_cubico)
```

Veriificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_cubico)
```

Notamos de manera general que nuestros residuales cada vez se hacen más pequeños de acuerdo al modelo empleado, por otro lado, podemos ver que se asemejan cada vez más a una distribución normal con una distribución asimétrica positiva, pues la cola de la distribución apunt hacia la derecha en el gráfico superior derecho, lo cual se comprueba de igual forma en el gráfico inferior derecho.

Obtengamos ahora las predicciones para el conjunto de observaciones y elaboremos la gráfica del modelo obtenido
```{r}
modelo_cubico_predict <- predict(modelo_cubico, data.frame(x = seq(2020,2035,.25)))
modelo_cubico_predict
```


```{r}
ggplot() + geom_point(data = Base1_depurada, aes(x = trimestres, y = porcentajes), size = 0.9) + 
  geom_line(aes( x = Base1_depurada$Trimestres, y = modelo_cubico_predict), color = "red") +
  xlab("Trimestres") + 
  ylab("Porcentajes") + 
  ggtitle("Curva de Ajuste de Regresión Cúbica")
```


En efecto, vemos que la curva de ajuste producida por la regresión sigue la forma no lineal de los datos observados y cada vez se adapta mejor el polinomio a los mismos.


Finalmente veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste $R^2$

```{r}
modelo_cubico_df <- data.frame(RMSE = rmse(modelo_cubico_predict,Base1_depurada$Porcentajes), 
           R2 = round(cor(modelo_cubico_predict,Base1_depurada$Porcentajes)^2, 10000))
modelo_cubico_df
```

El valor 0.17445 indica una baja correlación aunque en comparación con el modelo de regresión lineal simple (0.03846746) y el modelo polinomial cuadrático (0.1453784) es más alto, por otro lado, el error cuadrático medio 0.5545199 sigue mejorando (decreciendo) en comparación con el modelo de regresión lineal simple (0.5984499) y el modelo polinomial cuadrado (0.5641996) por lo tanto, es un ajuste aún más indicado para los datos por parte del modelo obtenido.

### Regresion spline cúbico
Ya analizamos el modelo de interpolación por splines cúbicos y observamos que era un buen modelo para interpolar y proyectar nuestros datos, ahora analicemos el modelo de regresión por splines cúbicos.

Este modelo se ajusta a una curva suave con una serie de segmentos polinomiales. Los valores que delimitan los segmentos spline se denominan nudos. 

Creamos los nudos para los segmentos polinomiales
```{r}
nudos <- quantile(Base1_depurada$Trimestres, p = c(0.25, 0.5 ,0.75))
```

Ahora creamos el modelo
```{r}
set.seed(1234)
modelo_spline_cub <- lm (porcentajes ~ bs(trimestres, knots = nudos), data = Base1_depurada)
```

Analizamos sus propiedades
```{r}
summary(modelo_spline_cub)
```

Revisamos el modelo
```{r}
anova(modelo_spline_cub)
```

Verificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_spline_cub)
```

Notamos que a pesar de ser una distribución aleatoria de nuestros datos, cada vez se acercan más a la linea horizontal que representa los residuales iguales a cero, es decir, de inicio podemos decir que aparentemente eso haría menos variable nuestro modelo (gráfica superior izquierda), por otro lado observamos que cada vez los residuales reflejan una distribución normal (gráfico superior derecho) y por último, notamos que cada vez tenemos menos valores atípicos (gráfico inferior derecho). 

Ahora obtengamos las predicciones para el conjunto de observaciones y elaboremos la gráfica del modelo obtenido
```{r}
#Predicciones
predicciones_splinescub <- modelo_spline_cub %>% predict(data.frame(x = seq(2020,2035,.25)))
predicciones_splinescub
```

Visualización del spline cúbico

```{r echo=FALSE}
ggplot(Base1_depurada, aes(trimestres, porcentajes))+ 
  geom_point() + ggtitle("Regresión con Spline") + 
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL, knots = nudos, degree = 3))
```

En efecto, vemos que la curva de ajuste producida por la regresión sigue la forma no lineal de los datos observados y cada vez simula de mejor manera el comportamiento de los mismos.

Finalmente veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste.
```{r}
# Rendimiento del modelo
modelo_splinecub_df <- data.frame(RMSE = rmse(predicciones_splinescub,Base1_depurada$Porcentajes), 
           R2 = round(cor(predicciones_splinescub,Base1_depurada$Porcentajes)^2, 100))
modelo_splinecub_df
```


El valor 0.6679225 indica una alta correlación y en comparación con el modelo de regresión lineal simple (0.03846746), el modelo polinomial cuadrático (0.1453784) y el modelo polinomial cúbico (0.1744514) es considerablemente más alto, por otro lado el error cuadrático medio de este metodo es de 0.3516945, significativamente menor al obtenido por los otros modelos de regresión, por lo tanto, podemos decir que el spline cúbico es un ajuste indicado para los datos por parte del modelo obtenido.


## Base 2 {.tabset .tabset-fade .tabset-pills}

### Introducción

Leemos la segunda base de datos

```{r echo=FALSE}
Base_2 <- read.csv("PercentageInflationRate_Indicadores20210207152622.csv")
Base_2
```



Guardamos las variables de la base a utilizar, por una parte los porcentajes,

```{r echo=FALSE}
porcentajes2 <- Base_2$X[5:616]
porcentajes2
```

y por otra parte debido a que tenemos 612 meses, comprendidos desde enero de 1970 a diciembre de 2020, entonces será más fácil gráficamente trabajar con un vector de enero de 1970 a diciembre de 2020 con saltos de $\frac{1}{12}$ (1 mes).



```{r}
meses2 <- seq(1970,2020.917,1/12)
meses2
```

Hasta ahora estos son los datos con los que trabajaremos,
```{r echo=FALSE}
porcentajes2 <- c(as.numeric(paste(porcentajes2)))
porcentajes2
meses2
```
guardamos ambos datos en un data frame al cual llamamos `base2_depurada`,
```{r echo=FALSE}
Base2_depurada<- data.frame(Meses = meses2,Porcentajes2 = porcentajes2)
```

`porcentajes2`  es el porcentaje de datos de la base, `meses` es el tiempo en meses de enero de 1970 a diciembre de 2020, $n$ la longitud de `porcentajes2`.


Ahora, veamos si nuestros datos presentan una relación lineal graficando un diagrama de dispersión de los mismos,

```{r echo=FALSE, message=FALSE}
ggplot(Base2_depurada, aes(meses2,porcentajes2))+ geom_point() + ggtitle("Gráfico de Dispersión") + stat_smooth() + labs(x = "Meses" , y ="Porcentajes") 
```


del diagrama anterior notamos que nuestros datos  presentan dos tipos de comportamientos, de 1970 al año 2000 no siguen una tendencia lineal entre si, sin embargo del año 2000 a 2020 aparentemente presentan una relación lineal.


Para ver que tanto varían nuestros datos entre si con los diferentes análisis utilizaremos las métricas **RMSE** y **$R^2$** para comparar los diferentes modelos. 

Recordemos que el **RMSE** representa el error de predicción del modelo, es decir, la diferencia promedio entre los valores de resultado observados y los valores de resultado predichos o mejor conocido como el error cuadrático medio. Por otro lado el **$R^2$** representará la correlación entre los valores de resultado observados y predichos.

El mejor modelo será aquel que tenga el **RMSE** más bajo y el **$R^2$** más alto, lo cual indicará un bajo índice en el error de predicción y una alta correlación entre los resultados.


### Interpolación con polinomios de Lagrange

Creamos la función que nos dará la interpolación de lagrange
```{r}
lagrange <- function(x,y,x0){ #x:datos en el eje horizontal, y:datos en el eje vertical, x0= punto donde se desea interpolar para conocer su valor
  y0 = 0 #Lo inicializamos en 0 pero será el valor de x interpolado, es decir,fungirá como la imagen de x a través de f=lagrange
  for(i in 1:length(x)){ #Inicializamos el índice donde comenzarán a correr los valores de xi
    Li = 1 #Será la primer basepolinómica de Lagrange (caso cuando el numerador coincide con el denominador es decir, cuando i = j)
    for(j in 1:length(x)){ #Inicializamos el índice donde comenzarán a correr los valores de xj
      if(i!= j){ #Condición para empezar a calcular los productos de las bases polinómicas y que sean distintos de 1
        Li = Li*(x0-x[j])/(x[i]-x[j]) #Expresión iterativa para calcular lasbases polinómicas
      }
    }
    y0 = y0+ Li*y[i] #Valor del polinomio interpolador de Lagrange del punto x0
  }
  return(y0) #Solo pedimos que exprese el valor resultante
}
```
Ahora veamos si nuestra función interpola correctamente los puntos de nuestra base de datos

```{r}
#Prueba
#Veamos los valores para enero de 1970 y junio de 2020
porcentajes2[1]
porcentajes2[607]
#Ahora veamos que nuestra función arroja los mismos valores
lagrange(meses2,porcentajes2,1970)
lagrange(meses2,porcentajes2,2020.5)
#Finalmente si quisieramos interpolar cualquier valor 
lagrange(meses2,porcentajes2,1987.7)
#Tener cuidado porque arroja valores en notación científica y exponencial
```

Ahora pondremos a prueba nuestra función para verificar que en realidad puede interpolar todos los valores de nuestra base de datos depurada.
```{r}
interpolaciones=0 #Inicializamos en cero el vector donde guardaremos las interpolaciones
  interpolacion_i<-lagrange(meses2,porcentajes2,seq(1970,2020.917,1/12))
  interpolaciones = interpolaciones + interpolacion_i
  interpolaciones
```



Obtengamos la visualización de las interpolaciones anteriores y de la base de datos.
```{r}
plot(meses2, porcentajes2, xlim=c(meses2[1],tail(meses2,1)), ylim=c(0,200), xlab = "", ylab = "")#Ploteamos los porcentajes de nuestra base de datos
par(new = 'TRUE') #Creamos un plot encima del otro
plot(meses2, interpolaciones, xlim=c(meses2[1],tail(meses2,1)), ylim=c(0,200), xlab = "Meses", ylab = "Porcentajes", col= 'steelblue', type = 'l') #Ploteamos las interpolaciones obtenidas 

```


Notamos que ambos gráficos se muestran encimados para asegurarnos que la función cumpla su objetivo por una parte los datos sin ningun tipo de linea que los una están representados por circulos, mientras que el polinomio de Lagrange que se adapta a los datos los une de color azul.

### Interpolación con splines cúbicos

Haremos una interpolación con splines cúbicos para quincenas, entonces generamos los splines cúbicos necesarios.
```{r}
func_spline2= splinefun(x=meses2,y=porcentajes2,method="fmm",ties=mean)
porcentajes_spline_quin2 <- func_spline2(seq(1970,2020.917,1/24)) #Aqui suponemos que hay 24 quincenas por cada año.
porcentajes_spline_quin2
```

Guardamos los porcentajes de la interpolación obtenida por spline cúbico para quincenas, primero generando las quincenas
```{r}
quincenas2 <- seq(1970,2020.917,1/24)
```
Guardando las quincenas y las observaciones
```{r}
base_splines_quin2 <- data.frame(Quincenas=quincenas2, Porcentajes = porcentajes_spline_quin2)
base_splines_quin2
```

Otra forma de interpolar los datos es con la función `spline` la cual devuelve una lista que contiene los componentes $x$ e $y$ que dan las ordenadas donde tuvo lugar la interpolación y los valores interpolados.
Esta interpolación suele contener un mayor número de observaciones. 

```{r}
interp_spline_m2 <- spline(meses2, y = porcentajes2, n = 3*length(meses2), method = "fmm",
       xmin = min(meses2), xmax = max(meses2), ties = mean)
interp_spline_m2
```
Guardamos en un data frame la interpolación que generó la función anterior.
```{r}
interp_spline_df_m2 <- data.frame(interp_spline_m2)
```


Veamos gráficamente la interpolación por la función de splines cúbicos

```{r}
#Generamos los nudos para la gráfica
nudos_spline_m2 <- quantile(interp_spline_df_m2$x, p = c(0.25, 0.5 ,0.75))
```

```{r echo=TRUE}
#Generamos la gráfica
ggplot(interp_spline_df_m2, aes(x, y))+ 
  geom_point() + ggtitle("Interpolación con spline cúbico") +labs(x = "Meses" , y ="Porcentajes")+
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL,  , knots = nudos_spline_m2, degree = 3))
```

Ahora para hacer proyecciones futuras utilizando la interpolación por splines para meses generamos el modelo, primero creamos el vector de tiempo donde queremos realizar las proyecciones,


```{r}
meses2_proyecciones <- seq(2021,2071.975,1/12)
```
creamos los nudos para los segmentos polinomiales.

```{r}
nudos2_proyecciones <- quantile(meses2_proyecciones, p = c(0.25, 0.5 ,0.75))
```
Seguido del modelo
```{r}
modelo_spline_cub_interp2 <- lm (porcentajes2 ~ bs(meses2_proyecciones, knots = nudos2_proyecciones), data = Base2_depurada)
```
Ahora las proyecciones
```{r}
proyeccion_splinescub2 <- modelo_spline_cub_interp2 %>% predict(data.frame(x = seq(2021,2071.975,1/12)))
proyeccion_splinescub2
```

Guardamos en un data frame los meses y la proyección que generó la función anterior.


```{r}
proyeccion_spline_df2 <- data.frame(Meses = meses2_proyecciones, Porcentajes = proyeccion_splinescub2)
proyeccion_spline_df2
```


Veamos gráficamente la proyección por el método de splines cúbicos


```{r echo=FALSE}
ggplot(proyeccion_spline_df2, aes(meses2_proyecciones  ,  proyeccion_splinescub2 ))+ 
  geom_point() + ggtitle("Proyección con spline cúbico para meses") + labs(x = "Meses",y = "Proyecciones") +
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL,  , knots = nudos2_proyecciones, degree = 3))
```
Notamos que el spline cúbico modela perfectamente nuestros datos interpolados y proyectados previamente.

Ahora, si quisieramos interpolar las proyecciones por quincenas suponiendo que cada año tiene 24 quincenas, lo haríamos de la siguiente forma.

Generamos los periodos de tiempo,
```{r}
quincenas2_proyecciones <- seq(2021,2071.917,1/24)
```
ahora creamos los nudos para los segmentos polinomiales.

```{r}
nudos_proyecciones_quinc2<- quantile(quincenas2_proyecciones, p = c(0.25, 0.5 ,0.75))
```
Seguido del modelo.

```{r}
modelo_spline_cub_interp_quin2<- lm (porcentajes_spline_quin2 ~ bs(quincenas2_proyecciones, knots = nudos_proyecciones_quinc2), data = Base2_depurada)
modelo_spline_cub_interp_quin2
```
Ahora las proyecciones
```{r}
proyeccion_splinescub_quin2 <- modelo_spline_cub_interp_quin2 %>% predict(data.frame(x = seq(2021,2071.917,1/24)))
proyeccion_splinescub_quin2
```

Guardamos en un data frame las quincenas y la proyección que generó la función anterior.


```{r}
proyeccion_spline_df_quin2 <- data.frame(Quincenas = quincenas2_proyecciones, Porcentajes = proyeccion_splinescub_quin2)
proyeccion_spline_df_quin2
```


Veamos gráficamente la proyección por el método de splines cúbicos para quincenas


```{r echo=FALSE}
ggplot(proyeccion_spline_df_quin2, aes(quincenas2_proyecciones  ,  proyeccion_splinescub_quin2 ))+ 
  geom_point() + ggtitle("Proyección con spline cúbico para quincenas") + labs(x = "Quincenas",y = "Proyecciones") +
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL,  , knots = nudos_proyecciones_quinc2, degree = 3))
```

Notamos como la cantidad de observaciones aumenta (puntos en el spline cúbico) y claramente nuestro spline se adapta de mejor manera a nuestros datos, apreciamos una gráfica mucho más completa.

### Regresión lineal simple

Para calcular el modelo de regresión lineal, construimos el modelo, y recordemos que el aplicar una regresión sobre estos datos implica obtener la línea recta que mejor ajuste la relación existente entre la variable independiente y la variable dependiente. Para ello, vamos a crear un modelo lineal haciendo uso de la función `lm` del paquete `stats`.


```{r}
# Construyendo el modelo
modelo_lineal2 <- lm(porcentajes2 ~ meses2, data = Base2_depurada)

```
Una vez creado el modelo lineal, podemos explorar los resultados y calidad del ajuste haciendo uso de la función summary
```{r}
# Nos brinda un resumen de la regresion
summary(modelo_lineal2)
```
Entre los datos de interés que ofrece la función `summary` están las estadísticas de los residuales,los valores de los coeficientes obtenidos, el p-value, los grados de libertad así como los valores de calidad de ajuste que este caso es de 0.133565273.




Revisamos el modelo
```{r}
anova(modelo_lineal2)
```
anova nos proporciona los grados de libertad Df en este caso de 610, el cálculo de la suma de cuadrados `Sum sq`, los cuadrados medios `Mean sq`, el estadístico F y el pi-value  = Pr(>F). 

Verificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_lineal2)

```

Aqui observamos que nuestros datos presentan errores residuales distintos de cero pues en una parte de estos se observa como se separan de la linea horizonal punteada que representa errores residuales iguales a cero (gráfico superior izquierdo), por otro lado vemos que los residuales no siguen una distribución normal (gráfico superior derecho) y finalmente, la presenciade datos atípicos en el modelo (gráfico inferior derecho).

Por otro lado, podemos determinar explícitamente la ecuación de predicción obtenida por la regresión gracias a la funcion `summary`, dicha ecuación estará dada por

$$Porcentajes2=1557.64122727  -0.769331591708*meses2$$
La interpretacion de los resultados es:

En promedio, cada incremento en una unidad de los meses , corresponde a un decremento del porcentaje de 21.91260087, asi mismo que un mes cuyo valor sea de cero (origen de y), tenga una porcentaje de 7.234107851, esto obviamente no parece posible. Dado que la regresion estima como logramos ver estimaciones mas allá de los valores observados.

Generamos la ecuación de la recta
```{r}
y_gorro2 = 1557.64122727  -0.769331591708*meses2
y_gorro2
```

Gráficamente se ve de la siguiente manera

```{r}
plot(meses2,porcentajes2)
lines(meses2,y_gorro2, type = "l", col = "darkred")
```

La recta pendiente que expresamos anteriormente, también se puede obtener manualmente de la siguiente manera

```{r}
#Calculamos la media de nuestros datos
media_p2 = mean(porcentajes2) 
media_t2 = mean(meses2)
```

Recordemos que 
$$\begin{align*} \hat{\beta_1} &= \frac{S_{XY}}{S_{XX}} \\
\\
\hat{\beta_0} &= \overline{Y} - \hat{\beta_1}\overline{X}
\end{align*} $$

```{r}
#Calculamos las ecuaciones normales
Sxy2 = sum( (porcentajes2-media_p2)*(meses2-media_t2) )
Sxy2

Sxx2 = sum( (meses2-media_t2)^2 )
Sxx2
```
Finalmente
```{r}
B1p2 = Sxy2 / Sxx2
B1p2

B0p2 = media_p2 - B1p2*media_t2
B0p2

#porcentaje = 1557.64122727  -0.769331591708*meses2
```

Generamos la ecuación
```{r}
y_gorrom2 = B0p2 + B1p2*meses2
y_gorrom2
```
Hacemos el plot
```{r}
plot(meses2,porcentajes2)
lines(meses2,y_gorrom2, type = "l", col = "darkred")
```


Una vez que tenemos el modelo construido, vamos a crear un vector de predicciones basado en el propio conjunto de observaciones, y con este vector podremos visualizar la curva de ajuste de los datos. Para obtener las predicciones, hacemos uso de la función `predict`


```{r}
# Predicciones
predicciones_lineales2 <- modelo_lineal2 %>% predict(Base2_depurada)
predicciones_lineales2
```
Otra forma de hacer proyecciones sería si deseáramos tener una predicción particular para valores cualesquiera de x, podemos hacerlo de la siguiente manera:
```{r}
predicciones_lineal2 <- predict(modelo_lineal2, data.frame(x = seq(2021,2071.975,1/12)))
predicciones_lineal2
```
Guardamos en un data frame los datos obtenidos con sus respectivos periodos.
```{r}
proyecciones2_df <- data.frame(Meses = seq(2021,2071.975,1/12) , Porcentajes = predicciones_lineal2)
proyecciones2_df
```


En este caso se obtiene una predicción para los meses comprendidos entre enero de 2021 a diciembre de 2071.

Es importante mencionar que la función `predict` espera como argumento de la variable independiente un dataframe, por lo que es necesario convertir el valor seleccionado a tal tipo de dato como se observa en el código anterior.

La forma gráfica de las predicciones anteriores se presenta a continuación.

```{r}
ggplot() + geom_point(data =proyecciones2_df , aes(x = meses2_proyecciones, y = predicciones_lineal2), size = 0.9) + 
  geom_line(aes( x = proyecciones2_df$Meses, y =predicciones_lineal2), color = "red") +
  xlab("Meses") + 
  ylab("Porcentajes") + 
  ggtitle("Curva de Ajuste sobre Conjunto de Observaciones")
```

Lo cual claramente representa la ecuación de predicción obtenida anteriormente, cuya pendiente es negativa.

Ahora construímos la gráfica adecuada para el modelo
```{r}
ggplot() + geom_point(data = Base2_depurada, aes(x = meses2, y = porcentajes2), size = 0.9) + 
  geom_line(aes( x = Base2_depurada$Meses, y =predicciones_lineales2), color = "red") +
  xlab("Meses") + 
  ylab("Porcentajes") + 
  ggtitle("Curva de Ajuste sobre Conjunto de Observaciones")
```

Otra forma sería


```{r echo=FALSE}
ggplot(Base2_depurada, aes(meses2, porcentajes2))+ 
  geom_point() + ggtitle("Regresión Lineal") +
  stat_smooth(method = lm,formula = y~x)
```

Finalmente veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste $R^2$


```{r}
modelo_lineal2_df <- data.frame(RMSE = rmse(predicciones_lineales2,Base2_depurada$Porcentajes2), 
           R2 = round(cor(predicciones_lineales2,Base2_depurada$Porcentajes2)^2, 10000))
modelo_lineal2_df
```

El valor 0.1335653 indica una baja correlación y el error cuadrático medio 28.8478 es elevado, por lo tanto, es un ajuste no indicado para los datos por parte del modelo obtenido.

### Regresión polinomial cuadrática 

Construimos nuestro modelo de regresión polinomial:

```{r}
set.seed(1234)
modelo_cuadratico2 <- lm(porcentajes2~ meses2 + I(meses2^2), data = Base2_depurada)
```


Otra forma es
```{r}
lm(porcentajes2 ~ poly(meses2,2,raw = TRUE), data =  Base2_depurada)
```
Como puede verse, en este caso la fórmula del primer argumento `porcentajes2 ~ meses2 + I(meses2^2)` incluye la nueva columna.

Al aplicar la función `summary` a nuestro nuevo regresor tenemos:
```{r}
summary(modelo_cuadratico2)
```

Como puede verse, tanto la variable `meses2` como la variable `meses2^2` son relevantes a efectos de la predicción (valores pequeños del p-value) y se obtiene un $R^2$ de 0.232297308.

Revisamos el modelo
```{r}
anova(modelo_cuadratico2)
```

Verificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_cuadratico2)
```
En la gráfica superior izquierda notamos como nuestros datos se alejan de la linea horizontal punteada que representa un error residual cero, por otro lado, en el gráfico superior derecho podemos ver como los residuales no siguen una distribución normal. Finalmente en el gráfico inferior derecho encontramos gran cantidad de valores atípicos. 

Ahora, obtengamos las predicciones para el conjunto de observaciones y elaboremos la gráfica del modelo obtenido
```{r}
#Predicciones
predicciones_cuadradas2 <- predict(modelo_cuadratico2,data.frame(x = seq(2021,2071.917,1/12)))
predicciones_cuadradas2
```
Gráficamente se ven de la siguiente forma
```{r echo=FALSE}
ggplot(Base2_depurada, aes(meses2,porcentajes2))+ 
  geom_point() + ggtitle("Regresión cuadrática")+ 
stat_smooth(method = lm, formula = y~poly(x,2,raw = TRUE))
```

En efecto, vemos que la curva de ajuste producida por la regresión sigue la forma no lineal de los datos observados y aun no logra representar una buena regresión entre ellos, para ello veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste $R^2$


```{r}
modelo_cuadratico2_df <- data.frame(RMSE = rmse(predicciones_cuadradas2,Base2_depurada$Porcentajes2), 
           R2 = round(cor(predicciones_cuadradas2,Base2_depurada$Porcentajes2)^2, 10000))
modelo_cuadratico2_df
```

El valor de calidad de ajuste 0.2322973 indica una baja correlación aunque en comparación con el modelo de regresión lineal simple (0.1335653) es más alto, por otro lado el error cuadrático medio 27.15453 es un poco mas bajo que el del modeloderegresión lineal simple (28.84787) , por lo tanto, podemos decirque el modelo de regresión polinomial cuadrado es un ajuste un poco más indicado para los datos por parte del modelo obtenido.



Ahora, ¿qué ocurre si incorporamos más órdenes al polinomio? ¿Nuestro modelo de regresión mejorará? Hagamos la prueba.

### Regresión polinomial cúbica 

Con la misma teoría estudiada para la regresión polinomial cuadrada creamos el modelo cúbico

Tomemos en cuenta que para la interpretación de los meses lo haremos de 1 a 612 entendiendo como 1 el mes de enero de 1970 y como 612 el ultimo mes de 2020, esto debido a que al momento de hacer el modelo cubico si tomáramos las fechas como "1970" al momento de realizar el cúbo la memoria no alcanza a guardar el resultado y los introducirá como NA. 

Creamos dichos meses.
```{r}
mesesb1 <- seq(1,612)
mesesb2 <- mesesb1^2
mesesb3 <- mesesb1^3
```
Generamos el modelo cúbico

```{r}
modelo_cubico2 <- lm(porcentajes2 ~ mesesb1 +mesesb2 + mesesb3, data =  Base2_depurada)
```

Analizamos sus propiedades 
```{r}
summary(modelo_cubico2)
```
Como puede verse, tanto la variable `mesesb1` ,la variable `mesesb2`  y la variable `mesesb3`son relevantes a efectos de la predicción (valores pequeños del p-value) y se obtiene un $R^2$ de 0.44468621.


Revisamos el modelo
```{r}
anova(modelo_cubico2)
```

Veriificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_cubico2)
```
En la gráfica superior izquierda notamos como nuestros datos se siguen alejando de la linea horizontal punteada que representa un error residual cero, por otro lado, en el gráfico superior derecho podemos ver como los residuales no siguen una distribución normal sin embargo se van acercando cada vez más. Finalmente en el gráfico inferior derecho encontramos gran cantidad de valores atípicos. 

Obtengamos ahora las predicciones para el conjunto de observaciones y elaboremos la gráfica del modelo obtenido
```{r}
modelo_cubico_predict2 <- predict(modelo_cubico2, data.frame(x = seq(2021,2071.917,1/12)))
modelo_cubico_predict2
```


```{r}
ggplot() + geom_point(data = Base2_depurada, aes(x = meses2, y = porcentajes2), size = 0.9) + 
  geom_line(aes( x = Base2_depurada$Meses, y = modelo_cubico_predict2), color = "red") +
  xlab("Meses") + 
  ylab("Porcentajes") + 
  ggtitle("Curva de Ajuste de Regresión Cúbica")
```


En efecto, vemos que la curva de ajuste producida por la regresión sigue la forma no lineal de los datos observados y cada vez se adapta mejor el polinomio a los mismos.

Finalmente veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste $R^2$


```{r}
modelo_cubico_df2 <- data.frame(RMSE = rmse(modelo_cubico_predict2,Base2_depurada$Porcentajes2), 
           R2 = round(cor(modelo_cubico_predict2,Base2_depurada$Porcentajes2)^2, 10000))
modelo_cubico_df2
```

El valor 0.4446862 indica una buena correlación y en comparación con el modelo de regresión lineal simple (0.1335653) y el modelo polinomial cuadrático (0.2322973) es más alto, por otro lado el error cuadrático medio 23.09484 sigue mejorando (decreciendo) en comparación con el modelo de regresión lineal simple (28.84787) y el modelo polinomial cuadrado (27.15453	), por lo tanto, es un ajuste más indicado para los datos por parte del modelo obtenido.



### Regresión spline cúbico
Hemos analizado el modelo de interpolación por splines cúbicos y observamos que era un buen modelo para interpolar y proyectar nuestros datos, ahora analizaremos el modelo de regresión por splines cúbicos.

Este modelo se ajusta a una curva suave con una serie de segmentos polinomiales. Los valores que delimitan los segmentos spline se denominan nudos. 

Creamos los nudos para los segmentos polinomiales
```{r}
nudos2 <- quantile(Base2_depurada$Meses, p = c(0.25, 0.5 ,0.75))
```

Ahora creamos el modelo
```{r}
set.seed(1234)
modelo_spline_cub2 <- lm (porcentajes2 ~ bs(meses2, knots = nudos2), data = Base2_depurada)
```

Analizamos sus propiedades

```{r}
summary(modelo_spline_cub2)
```


Revisamos el modelo
```{r}
anova(modelo_spline_cub2)
```

Verificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_spline_cub2)
```

Ahora obtengamos las predicciones para el conjunto de observaciones y elaboremos la gráfica del modelo obtenido
```{r}
#Predicciones
predicciones_splinescub2 <- modelo_spline_cub2 %>% predict(data.frame(x = seq(2021,2071.917,1/12)))
predicciones_splinescub2
```

Visualización del spline cúbico

```{r echo=FALSE}
ggplot(Base2_depurada, aes(meses2, porcentajes2))+ 
  geom_point() + ggtitle("Regresión con Spline") + 
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL, knots = nudos2, degree = 3))
```


En efecto, vemos que la curva de ajuste producida por la regresión sigue la forma no lineal de los datos observados.

Finalmente veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste
```{r}
# Rendimiento del modelo
modelo_splinecub_df2 <- data.frame(RMSE = rmse(predicciones_splinescub2,Base2_depurada$Porcentajes2), 
           R2 = round(cor(predicciones_splinescub2,Base2_depurada$Porcentajes2)^2, 100))
modelo_splinecub_df2
```


El valor 0.5823296 indica una alta correlación y en comparación con el modelo de regresión lineal simple (0.1335653), el modelo polinomial cuadrático (0.2322973) y el modelo polinomial cúbico (0.4446862) es considerablemente más alto, por otro lado el error cuadrático medio de este método es de 20.02915, significativamente menor al obtenido por los otros modelos de regresión, por lo tanto, podemos decir que el spline cúbico es un ajuste indicado para los datos por parte del modelo obtenido.





## Base 3 {.tabset .tabset-fade .tabset-pills}

### Introducción

Leemos la segunda base de datos

```{r echo=FALSE}
Base_3 <- read.csv("ProduccionConstruccion_Indicadores20210207152959.csv")
Base_3
```



Guardamos las variables de la base a utilizar, por una parte los porcentajes,

```{r echo=FALSE}
porcentajes3 <- Base_3$X[5:183]
porcentajes3
```

y por otra parte debido a que tenemos 179 meses, comprendidos desde enero de 2006 a noviembre de 2020, entonces será más fácil gráficamente trabajar con un vector de enero de 2006 a noviembre de 2020 con saltos de $\frac{1}{12}$ (1 mes).



```{r}
meses3 <- seq(2006,2020 + 10/12,1/12)
meses3
```

Hasta ahora estos son los datos con los que trabajaremos,
```{r echo=FALSE}
options(digits = 12)
porcentajes3 <- c(as.numeric(paste(porcentajes3)))
porcentajes3
meses3
```
guardamos ambos datos en un data frame al cual llamamos `base3_depurada`,
```{r echo=FALSE}
Base3_depurada<- data.frame(Meses = meses3,Porcentajes3 = porcentajes3)
```

`porcentajes3`  es el porcentaje de datos de la base, `meses3` es el tiempo en meses de enero de 2006 a finles de noviembre de 2020, $n$ la longitud de `porcentajes3`.


Ahora, veamos si nuestros datos presentan una relación lineal graficando un diagrama de dispersión de los mismos,

```{r echo=FALSE, message=FALSE}
ggplot(Base3_depurada, aes(meses3,porcentajes3))+ geom_point() + ggtitle("Gráfico de Dispersión") + stat_smooth() + labs(x = "Meses" , y ="Miles de pesos") 
```


del diagrama anterior notamos que nuestros datos no siguen una tendencia lineal entre si.


Para ver que tanto varían nuestros datos entre si con los diferentes análisis utilizaremos las métricas **RMSE** y **$R^2$** para comparar los diferentes modelos. 

Recordemos que el **RMSE** representa el error de predicción del modelo, es decir, la diferencia promedio entre los valores de resultado observados y los valores de resultado predichos o mejor conocido como el error cuadrático medio. Por otro lado el **$R^2$** representará la correlación entre los valores de resultado observados y predichos.

El mejor modelo será aquel que tenga el **RMSE** más bajo y el **$R^2$** más alto, lo cual indicará un bajo índice en el error de predicción y una alta correlación entre los resultados.


### Interpolación con polinomios de Lagrange

Creamos la función que nos dará la interpolación de lagrange
```{r}
lagrange <- function(x,y,x0){ #x:datos en el eje horizontal, y:datos en el eje vertical, x0= punto donde se desea interpolar para conocer su valor
  y0 = 0 #Lo inicializamos en 0 pero será el valor de x interpolado, es decir,fungirá como la imagen de x a través de f=lagrange
  for(i in 1:length(x)){ #Inicializamos el índice donde comenzarán a correr los valores de xi
    Li = 1 #Será la primer basepolinómica de Lagrange (caso cuando el numerador coincide con el denominador es decir, cuando i = j)
    for(j in 1:length(x)){ #Inicializamos el índice donde comenzarán a correr los valores de xj
      if(i!= j){ #Condición para empezar a calcular los productos de las bases polinómicas y que sean distintos de 1
        Li = Li*(x0-x[j])/(x[i]-x[j]) #Expresión iterativa para calcular lasbases polinómicas
      }
    }
    y0 = y0+ Li*y[i] #Valor del polinomio interpolador de Lagrange del punto x0
  }
  return(y0) #Solo pedimos que exprese el valor resultante
}
```
Veamos si nuestra función interpola correctamente los puntos de nuestra base de datos

```{r}
#Prueba
#Veamos los valores para enero de 2006 y noviembre de 2020
porcentajes3[1]
porcentajes3[179]
#Ahora veamos que nuestra función arroja los mismos valores
lagrange(meses3,porcentajes3,2006)
lagrange(meses3,porcentajes3,2020 + 10/12)
#Finalmente si quisieramos interpolar cualquier valor 
lagrange(meses3,porcentajes3,2020.1)
#Tener cuidado porque arroja valores en notación científica y exponencial
```

Ahora pondremos a prueba nuestra función para verificar que en realidad puede interpolar todos los valores de nuestra base de datos depurada.
```{r}
interpolaciones3=0 #Inicializamos en cero el vector donde guardaremos las interpolaciones
  interpolacion_3i<-lagrange(meses3,porcentajes3,seq(2006,2020 + 10/12,1/12))
  interpolaciones3 = interpolaciones3 + interpolacion_3i
  interpolaciones3
```



Obtengamos la visualización de las interpolaciones anteriores y de la base de datos.
```{r}
plot(meses3, porcentajes3, xlim=c(meses3[1],tail(meses3,1)), ylim=c(19775429.91,37835701.78), xlab = "", ylab = "")#Ploteamos los porcentajes de nuestra base de datos
par(new = 'TRUE') #Creamos un plot encima del otro
plot(meses3, interpolaciones3, xlim=c(meses3[1],tail(meses3,1)), ylim=c(19775429.91,37835701.78), xlab = "Meses", ylab = "Miles de pesos", col= 'steelblue', type = 'l') #Ploteamos las interpolaciones obtenidas 

```


Notamos que ambos gráficos se muestran encimados para asegurarnos que la función cumpla su objetivo por una parte los datos sin ningun tipo de linea que los una están representados por circulos, mientras que el polinomio de Lagrange que se adapta a los datos los une de color azul.

### Interpolación con splines cúbicos
Haremos una interpolación con splines cúbicos para quincenas, entonces generamos los splines cúbicos necesarios.
```{r}
func_spline3 = splinefun(x=meses3,y=porcentajes3,method="fmm",ties=mean)
porcentajes_spline_quin3 <- func_spline3(seq(2006,2020 + 10/12,1/24))
porcentajes_spline_quin3
```

Guardamos los porcentajes de la interpolación obtenida por spline cúbico para quincenas, primero generando las quincenas
```{r}
quincenas3 <- seq(2006,2020 + 10/12,1/24)
```
Guardando las quincenas y las observaciones
```{r}
base_splines_quin3 <- data.frame(Quincenas=quincenas3, Porcentajes = porcentajes_spline_quin3)
base_splines_quin3
```

Otra forma de interpolar los datos es con la función `spline` la cual devuelve una lista que contiene los componentes $x$ e $y$ que dan las ordenadas donde tuvo lugar la interpolación y los valores interpolados.

```{r}
interp_spline_m3 <- spline(meses3, y = porcentajes3, n = 3*length(meses3), method = "fmm",
       xmin = min(meses3), xmax = max(meses3), ties = mean)
interp_spline_m3
```
Guardamos en un data frame la interpolación que generó la función anterior.
```{r}
interp_spline_df_m3 <- data.frame(interp_spline_m3)
```


Veamos gráficamente la interpolación por la función de splines cúbicos

```{r}
#Generamos los nudos para la gráfica
nudos_spline_m3 <- quantile(interp_spline_df_m3$x, p = c(0.25, 0.5 ,0.75))
```

```{r echo=TRUE}
#Generamos la gráfica
ggplot(interp_spline_df_m3, aes(x, y))+ 
  geom_point() + ggtitle("Interpolación con spline cúbico") +labs(x = "Meses" , y ="Miles de pesos")+
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL,  , knots = nudos_spline_m3, degree = 3))
```

Ahora para hacer proyecciones futuras utilizando la interpolación por splines para meses generamos el modelo, primero creamos el vector de tiempo donde queremos realizar las proyecciones,


```{r}
meses3_proyecciones <- seq(2020,2034 + 10/12 ,1/12)
```
creamos los nudos para los segmentos polinomiales.

```{r}
nudos3_proyecciones <- quantile(meses3_proyecciones, p = c(0.25, 0.5 ,0.75))
```
Seguido del modelo
```{r}
modelo_spline_cub_interp3 <- lm (porcentajes3 ~ bs(meses3_proyecciones, knots = nudos3_proyecciones), data = Base3_depurada)
```
Ahora las proyecciones
```{r}
#Predicciones

proyeccion_splinescub3 <- modelo_spline_cub_interp3 %>% predict(data.frame(x = seq(2020,2034 + 10/12 ,1/12)))
proyeccion_splinescub3
```

Guardamos en un data frame los meses y la proyección que generó la función anterior.


```{r}
proyeccion_spline_df3 <- data.frame(Meses = meses3_proyecciones, Porcentajes= proyeccion_splinescub3)
proyeccion_spline_df3
```


Veamos gráficamente la proyección por el método de splines cúbicos


```{r echo=FALSE}
ggplot(proyeccion_spline_df3, aes(meses3_proyecciones  ,  proyeccion_splinescub3 ))+ 
  geom_point() + ggtitle("Proyección con spline cúbico para meses") + labs(x = "Meses",y = "Proyecciones") +
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL,  , knots = nudos3_proyecciones, degree = 3))
```
Notamos que el spline cúbico modela perfectamente nuestros datos interpolados y proyectados previamente.

Ahora, si quisieramos interpolar las proyecciones por quincenas suponiendo que cada año tiene 24 quincenas, lo haríamos de la siguiente forma.



Generamos los periodos de tiempo,
```{r}
quincenas3_proyecciones <- seq(2020,2034 + 10/12 ,1/24)
```
ahora creamos los nudos para los segmentos polinomiales.

```{r}
nudos_proyecciones_quinc3<- quantile(quincenas3_proyecciones, p = c(0.25, 0.5 ,0.75))
```
Seguido del modelo

```{r}
modelo_spline_cub_interp_quin3<- lm (porcentajes_spline_quin3 ~ bs(quincenas3_proyecciones, knots = nudos_proyecciones_quinc3), data = Base3_depurada)
modelo_spline_cub_interp_quin3
```
Ahora las proyecciones

```{r}
proyeccion_splinescub_quin3 <- modelo_spline_cub_interp_quin3 %>% predict(data.frame(x = seq(2020,2034 + 10/12 ,1/24)))
proyeccion_splinescub_quin3
```

Guardamos en un data frame las quincenas y la proyección que generó la función anterior.


```{r}
proyeccion_spline_df_quin3 <- data.frame(Quincenas = quincenas3_proyecciones, Porcentajes = proyeccion_splinescub_quin3)
proyeccion_spline_df_quin3
```


Veamos gráficamente la proyección por el método de splines cúbicos


```{r echo=FALSE}
ggplot(proyeccion_spline_df_quin3, aes(quincenas3_proyecciones  ,  proyeccion_splinescub_quin3 ))+ 
  geom_point() + ggtitle("Proyección con spline cúbico para quincenas") + labs(x = "Quincenas",y = "Proyecciones") +
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL,  , knots = nudos_proyecciones_quinc3, degree = 3))
```
Notamos como la cantidad de observaciones aumenta (puntos en el spline cúbico) y claramente nuestro spline se adapta de mejor manera a nuestros datos, apreciamos una gráfica mucho más completa.


### Regresión lineal simple

Para calcular el modelo de regresión lineal, construimos el modelo,y recordemos que el aplicar una regresión sobre estos datos implica obtener la línea recta que mejor ajuste la relación existente entre la variable independiente y la variable dependiente. Para ello, vamos a crear un modelo lineal haciendo uso de la función lm del paquete stats.


```{r}
# Construyendo el modelo
modelo_lineal3 <- lm(porcentajes3 ~ meses3, data = Base3_depurada)

```
Una vez creado el modelo lineal, podemos explorar los resultados y calidad del ajuste haciendo uso de la función summary
```{r}
# Nos brinda un resumen de la regresion
summary(modelo_lineal3)
```
Entre los datos de interés que ofrece la función `summary` están las estadísticas de los residuales,los valores de los coeficientes obtenidos, el p-value, los grados de libertad así como los valores de calidad de ajuste que este caso es de 0.533963504.

Revisamos el modelo
```{r}
anova(modelo_lineal3)
```
anova nos proporciona los grados de libertad Df en este caso de 177, el cálculode la suma de cuadrados `Sum sq`, los cuadrados medios  `Mean sq`, el estadístico F y el `pi-value  = Pr(>F)`.

Verificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_lineal3)

```

De la gráfica superior izquierda notamos que nuestros datos distribuidos aleatoriamente están cercanos a la linea horizontal que representa errores residuales cero, por otro lado, en el gráfico superior derecho observamos que los residuales siguen casi una distribución normal, pues en general la mayoría de ellos están muy cercanos a la linea diagonal punteada, finalmente, del gráfico inferior derecho observamos que hay muy pocos valores atípicos.

Por otro lado, podemos determinar explícitamente la ecuación de predicción obtenida por la regresión gracias a la funcion summary, dicha ecuación estará dada por

$$Porcentajes3 = 1260971203.93 -609659.164184*meses3$$
La interpretacion de los resultados es:

En promedio, cada incremento en una unidad de los meses , corresponde a un decremento del porcentaje de 11.57225427, asi mismo que un mes cuyo valor sea de cero (origen de y), tenga una porcentaje de 12.4276751, esto obviamente no parece posible. Dado que la regresion estima como logramos ver estimaciones mas allá de los valores observados.

Generamos la ecuación de la recta
```{r}
y_gorro3 = 1260971203.93 -609659.164184*meses3
y_gorro3
```

Gráficamente se ve de la siguiente manera

```{r}
plot(meses3,porcentajes3)
lines(meses3,y_gorro3, type = "l", col = "darkred")
```

La recta pendiente que expresamos anteriormente, también se puede obtener manualmente de la siguiente manera

```{r}
#Calculamos la media de nuestros datos
media_p3 = mean(porcentajes3) 
media_t3 = mean(meses3)
```

Recordemos que 
$$\begin{align*} \hat{\beta_1} &= \frac{S_{XY}}{S_{XX}} \\
\\
\hat{\beta_0} &= \overline{Y} - \hat{\beta_1}\overline{X}
\end{align*} $$

```{r}
#Calculamos las ecuaciones normales
Sxy3 = sum( (porcentajes3-media_p3)*(meses3-media_t3) )
Sxy3

Sxx3 = sum( (meses3-media_t3)^2 )
Sxx3
```
Finalmente
```{r}
B1p3 = Sxy3 / Sxx3
B1p3

B0p3 = media_p3 - B1p3*media_t3
B0p3

#porcentaje3 = 1260971203.93 -609659.164184*meses3
```

Generamos la ecuación
```{r}
y_gorrom3 = B0p3 + B1p3*meses3
y_gorrom3
```
Hacemos el plot
```{r}
plot(meses3,porcentajes3)
lines(meses3,y_gorrom3, type = "l", col = "darkred")
```


Una vez que tenemos el modelo construido, vamos a crear un vector de predicciones basado en el propio conjunto de observaciones, y con este vector podremos visualizar la curva de ajuste de los datos. Para obtener las predicciones, hacemos uso de la función `predict`


```{r}
# Predicciones
predicciones_lineales3 <- modelo_lineal3 %>% predict(Base3_depurada)
predicciones_lineales3
```
Otra forma de hacer proyecciones sería si deseáramos tener una predicción particular para valores cualesquiera de $x$, podemos hacerlo de la siguiente manera:
```{r}
predicciones_lineal3 <- predict(modelo_lineal3, data.frame(x = seq(2020,2034 + 10/12,1/12)))
predicciones_lineal3
```
```{r}
proyecciones3_df <- data.frame(Meses = seq(2020,2034 + 10/12,1/12) , Porcentajes = predicciones_lineal3)
proyecciones3_df
```


En este caso se obtiene una predicción para los meses comprendidos entre enero de 2020 a noviembre de 2034.

En este punto es importante mencionar que la función `predict` espera como argumento de la variable independiente un dataframe, por lo que es necesario convertir el valor seleccionado a tal tipo de dato como se observa en el código anterior.

La forma gráfica de las predicciones anteriores se presenta a continuación.


```{r}
ggplot() + geom_point(data = proyecciones3_df , aes(x = meses3_proyecciones, y = predicciones_lineal3), size = 0.9) + 
  geom_line(aes( x = proyecciones3_df$Meses, y =predicciones_lineal3), color = "red") +
  xlab("Meses") + 
  ylab("Porcentajes") + 
  ggtitle("Curva de Ajuste sobre Conjunto de Observaciones")
```

Lo cual claramente representa la ecuación de predicción obtenida anteriormente, cuya pendiente es negativa.

Ahora construímos la gráfica adecuada para el modelo
```{r}
ggplot() + geom_point(data = Base3_depurada, aes(x = meses3, y = porcentajes3), size = 0.9) + 
  geom_line(aes( x = Base3_depurada$Meses, y =predicciones_lineales3), color = "red") +
  xlab("Meses") + 
  ylab("Porcentajes") + 
  ggtitle("Curva de Ajuste sobre Conjunto de Observaciones")
```

Otra forma sería


```{r echo=FALSE}
ggplot(Base3_depurada, aes(meses3, porcentajes3))+ 
  geom_point() + ggtitle("Regresión Lineal") +
  stat_smooth(method = lm,formula = y~x)
```

Finalmente veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste $R^2$


```{r}
modelo_lineal3_df <- data.frame(RMSE = rmse(predicciones_lineales3,Base3_depurada$Porcentajes3), 
           R2 = round(cor(predicciones_lineales3,Base3_depurada$Porcentajes3)^2, 10000))
modelo_lineal3_df
```

El valor 0.533963504434 indica una baja correlación y el error cuadrático medio  es elevado, por lo tanto, es un ajuste no indicado para los datos por parte del modelo obtenido.

### Regresión polinomial cuadrática 

Construimos nuestro modelo de regresión polinomial:

```{r}
set.seed(1234)
modelo_cuadratico3 <- lm(porcentajes3~ meses3 + I(meses3^2), data = Base3_depurada)
```


Otra forma es
```{r}
lm(porcentajes3 ~ poly(meses3,2,raw = TRUE), data =  Base3_depurada)
```
Como puede verse, en este caso la fórmula del primer argumento `porcentajes3 ~ meses3 + I(meses3^2)` incluye la nueva columna.

Al aplicar la función `summary` a nuestro nuevo regresor tenemos:


```{r}
summary(modelo_cuadratico3)
```

Como puede verse, tanto la variable `meses3` como la variable `meses3^2` son relevantes a efectos de la predicción (valores pequeños del p-value) y se obtiene un $R^2$ de 0.725171181.

Revisamos el modelo
```{r}
anova(modelo_cuadratico3)
```

Verificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_cuadratico3)
```
Análogo al análisis para la regresión lineal simple en general se siguen conservando los supuestos mencionados anteriormente.

Obtengamos las predicciones para el conjunto de observaciones y elaboremos la gráfica del modelo obtenido
```{r}
#Predicciones
predicciones_cuadradas3 <- predict(modelo_cuadratico3,data.frame(x = seq(2020,2034 + 10/12,1/12)))
predicciones_cuadradas3
```
Gráficamente se ven de la siguiente forma 
```{r echo=FALSE}
ggplot(Base3_depurada, aes(meses3,porcentajes3))+ 
  geom_point() + ggtitle("Regresión cuadrática")+ 
stat_smooth(method = lm, formula = y~poly(x,2,raw = TRUE))
```

En efecto, vemos que la curva de ajuste producida por la regresión sigue la forma no lineal de los datos observados y comienza a representar una buena regresión entre ellos.

Ahora veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste $R^2$


```{r}
modelo_cuadratico3_df <- data.frame(RMSE = rmse(predicciones_cuadradas3,Base3_depurada$Porcentajes3), 
           R2 = round(cor(predicciones_cuadradas3,Base3_depurada$Porcentajes3)^2, 10000))
modelo_cuadratico3_df
```



El valor 0.72517118103 indica una alta correlación y en comparación con el modelo de regresión lineal simple es más alto  (mejor) , de la misma manera el error cuadrático medio sigue decreciendo, por lo tanto, podemos decir que es un ajuste más indicado para los datos por parte del modelo obtenido.



Ahora, ¿qué ocurre si incorporamos más órdenes al polinomio? Hagamos la prueba

### Regresión polinomial cúbica 

Creamos el modelo cúbico

Para la interpretación de los meses lo haremos de 1 a 179 entendiendo como 1 el mes de enero de 2006 y como 179 el mes de noviembre de 2020, esto debido a que al momento de hacer el modelo cubico si tomáramos las fechas como "2006" al momento de realizar el cúbo la memoria no alcanza a guardar el resultado y los introducirá como NA. 

Con la misma teoría estudiada para la regresión polinomial cuadrada creamos el modelo cúbico


Creamos dichos meses.

```{r}
mesesc1 <- seq(1,179)
mesesc2 <- mesesc1^2
mesesc3 <- mesesc1^3
```


```{r}
modelo_cubico3 <- lm(porcentajes3 ~ mesesc1 +mesesc2 + mesesc3, data =  Base3_depurada)
```


Analizamos sus propiedades 
```{r}
summary(modelo_cubico3)
```
Como puede verse, tanto la variable `mesesc1` ,la variable `mesesc2`  y la variable `mesesc3`son relevantes a efectos de la predicción (valores pequeños del p-value) y se obtiene un $R^2$ de 0.868797921.


Revisamos el modelo
```{r}
anova(modelo_cubico3)
```

Veriificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_cubico3)
```

Lo mas notorio en este modelo es que los erroes residuales ya se ajustaron a una distribución normal con media 0 y varianza $\sigma^2$, donde claramente lo podemos ver en el gráfico superior derecho.

Obtengamos las predicciones para el conjunto de observaciones y elaboremos la gráfica del modelo obtenido
```{r}
modelo_cubico_predict3 <- predict(modelo_cubico3, data.frame(x = seq(2020,2034 + 10/12,1/12)))
modelo_cubico_predict3
```
Graficamos dichas predicciones

```{r}
ggplot() + geom_point(data = Base3_depurada, aes(x = meses3, y = porcentajes3), size = 0.9) + 
  geom_line(aes( x = Base3_depurada$Meses, y = modelo_cubico_predict3), color = "red") +
  xlab("Meses") + 
  ylab("Porcentajes") + 
  ggtitle("Curva de Ajuste de Regresión Cúbica")
```


En efecto, vemos que la curva de ajuste producida por la regresión sigue la forma no lineal de los datos observados y cada vez simula un poco más el comportamiento de nuestros datos.

Finalmente veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste $R^2$


```{r}
modelo_cubico_df3 <- data.frame(RMSE = rmse(modelo_cubico_predict3,Base3_depurada$Porcentajes3), 
           R2 = round(cor(modelo_cubico_predict3,Base3_depurada$Porcentajes3)^2, 10000))
modelo_cubico_df3
```

El valor 0.868797921018 indica una alta correlación en comparación con el modelo de regresión lineal simple (0.533963504434) y el modelo polinomial cuadrático (0.72517118103) es más alto y, por lo tanto, es un ajuste aún más indicado para los datos por parte del modelo obtenido.



### Regresión spline cúbico
Hemos analizado el modelo de interpolación por splines cúbicos y observamos que era un buen modelo para interpolar y proyectar nuestros datos, ahora analizaremos el modelo de regresión por splines cúbicos.

Este modelo se ajusta a una curva suave con una serie de segmentos polinomiales. Los valores que delimitan los segmentos spline se denominan nudos. 

Creamos los nudos para los segmentos polinomiales

```{r}
nudos3 <- quantile(Base3_depurada$Meses, p = c(0.25, 0.5 ,0.75))
```

Ahora creamos el modelo
```{r}
set.seed(1234)
modelo_spline_cub3 <- lm (porcentajes3 ~ bs(meses3, knots = nudos3), data = Base3_depurada)
```

Analizamos sus propiedades

```{r}
summary(modelo_spline_cub3)
```

Notamos que obtenemos un $R^2$ de 0.940178532 lo cual indicaría indudablemente que es un muy buen modelo para nuestros datos, pues la correlación es demasiado alta.

Revisamos el modelo
```{r}
anova(modelo_spline_cub3)
```

Verificamos los supuestos
```{r}
par(mfrow = c(2, 2))
plot(modelo_spline_cub3)
```

Obtengamos las predicciones para el conjunto de observaciones y elaboremos la gráfica del modelo obtenido
```{r}
#Predicciones
predicciones_splinescub3 <- modelo_spline_cub3 %>% predict(data.frame(x = seq(2020,2034 +10/12,1/12)))
predicciones_splinescub3
```

Visualización del spline cúbico

```{r echo=FALSE}
ggplot(Base3_depurada, aes(meses3, porcentajes3))+ 
  geom_point() + ggtitle("Regresión con Spline") + 
  stat_smooth(method = lm, formula = y ~ splines::bs(x,df = NULL, knots = nudos3, degree = 3))
```


En efecto, vemos que la curva de ajuste producida por la regresión sigue la forma no lineal de los datos observados pero claramente el spline cúbico se adapta muy bien al comportamiento de nuestros datos

Finalmente veamos el rendimiento del modelo de acuerdo a su error cuadrático medio y los valores de calidad del ajuste 
```{r}
# Rendimiento del modelo
modelo_splinecub_df3 <- data.frame(RMSE = rmse(predicciones_splinescub3,Base3_depurada$Porcentajes3), 
           R2 = round(cor(predicciones_splinescub3,Base3_depurada$Porcentajes3)^2, 100))
modelo_splinecub_df3
```


El valor 0.940178531553 indica una  muy alta correlación en comparación con el modelo de regresión lineal simple, el modelo polinomial cuadrático  y el modelo polinomial cúbico, por lo tanto, podemos decir que el spline cúbico es un ajuste ampliamente indicado para los datos por parte del modelo obtenido.



