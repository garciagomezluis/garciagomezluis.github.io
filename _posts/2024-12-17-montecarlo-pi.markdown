---
layout: post
title:  "Método de Montecarlo para la aproximación del valor de PI"
date:   2024-12-17
categories: opinión
published: true
---
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

## Problema

Estimar el valor de \\(\pi\\) utilizando el [Método de Montecarlo](https://es.wikipedia.org/wiki/M%C3%A9todo_de_Montecarlo).

¿Cuántas muestras se deben tomar si se quiere un error menor a 0.001 con probabilidad mayor que 0.9?

## Introducción

El Método de Montecarlo consiste en aproximar el valor de una incógnita utilizando la probabilidad.

Una forma no original de hallar el valor de \\(\pi\\) en un caso como el planteado es imaginar que queremos llenar de dardos toda la superficie de una diana. 

Dado que la diana es circular, conocemos su área: esto es importante porque nos permite introducir a \\(\pi\\) en el problema.

A medida que tiremos dardos vamos a ir "pintando" la diana con todas las posiciones en donde impactaron: esto es importante porque nos permite introducir a \\(n\\), la cantidad de dardos lanzados, o bien la cantidad de muestras tenidas en consideración.

Ambas partes relacionadas como se muestra a continuación nos permite estimar cuantos dardos (muestras) son necesarios para aproximar \\(\pi\\) según lo pedido.

## Solución

Consideremos \\( U_1, ..., U_n \\) como una muestra aleatoria con distribución \\( U[0,1] \\). Por la [Ley de los Grandes Números](https://es.wikipedia.org/wiki/Ley_de_los_grandes_n%C3%BAmeros), para una función continua \\( h \\), vale que:

$$
\frac{1}{n} \sum_{i=1}^n h(U_i) \to \int_0^1 h(x) dx \quad \text{cuando } n \to \infty.
$$

Pensemos en un arco de circunferencia de radio \\(1\\) en el primer cuadrante del plano cartesiano y llamemos a la región denotada como \\( A \\):

$$
\text{Área}(A) = \frac{\pi \cdot r^2}{4} \implies 4 \cdot \text{Área}(A) = \pi,
$$

donde \\( r = \text{radio de la circunferencia} = 1 \\).

Sabemos que entonces:

$$
\pi = 4 \cdot \text{Área}(A) = 4 \int_0^1 h(x) dx, \quad \text{donde } h(x) = \sqrt{1 - x^2}.
$$

\\(h\\) es continua, luego por la Ley de los Grandes Números, tenemos que:

$$
\pi \approx 4 \cdot \frac{1}{n} \sum_{i=1}^n \sqrt{1 - U_i^2}.
$$

Llamemos a esta expresión \\( X \\). Entonces, teniendo en cuenta que la muestra tiene distribución \\( U[0,1] \\):

$$
E(X) = E\left( 4 \cdot \frac{1}{n} \sum_{i=1}^n \sqrt{1 - U_i^2} \right) = 4 \cdot E(\sqrt{1 - U^2}) = \pi.
$$

$$
\text{Var}(X) = \text{Var}\left( 4 \cdot \frac{1}{n} \sum_{i=1}^n \sqrt{1 - U_i^2} \right) = \frac{16}{n} \cdot \text{Var}(\sqrt{1 - U^2}),
$$

donde:

$$
\text{Var}(\sqrt{1 - U^2}) = E(1 - U^2) - E(\sqrt{1 - U^2})^2 = \frac{2}{3} - \frac{\pi^2}{16}.
$$

Finalmente, tenemos:

$$
\text{Var}(X) = \frac{16}{n} \left( \frac{2}{3} - \frac{\pi^2}{16} \right) \approx \frac{0.797}{n}.
$$

Para un \\( n \\) tal que \\( E(X) \\) y \\( \text{Var}(X) \\) existan, por la [Desigualdad de Chebyshev](https://es.wikipedia.org/wiki/Desigualdad_de_Chebyshov):

$$
P(|X - E(X)| \leq \varepsilon) \geq 1 - \frac{\text{Var}(X)}{\varepsilon^2}.
$$

Si \\( \varepsilon = 0.001 \\), entonces:

$$
P(|X - \pi| \leq 0.001) \geq 1 - \frac{0.797}{n \cdot 0.001^2} \geq 0.9 \implies n \geq 7,970,000.
$$

## Conclusión

Se necesitan al menos \\(7,970,000\\) dardos lanzados a una diana de radio \\(1\\) para estimar el valor de \\(\pi\\) por un error menor a 0.001 con probabilidad mayor que 0.9.

Para el desarrollo representamos a una fracción de la diana como una función continua \\(h\\) y asumimos que cuando \\(n\\) (la cantidad de dardos) es suficientemente grande entonces podemos usar la Ley de los Grandes Números. Eso nos sirvió para calcular la esperanza y varianza de \\(X\\), necesaria para introducirlas en la Desigualdad de Chebyshev lo cual nos permitió encontrar la cota inferior para la cantidad de muestras.

## Implementación en R

El código propuesto para realizar este cálculo de forma vectorizada es:

```r
aproximarPi <- function(cantPuntos = 7970000) {
  x <- runif(cantPuntos, 0, 1)
  h <- function(x) {
    return(sqrt(1 - x^2))
  }
  promedio <- mean(h(x))
  return(4 * promedio)
}
```