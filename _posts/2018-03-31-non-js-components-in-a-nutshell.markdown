---
layout: post
title:  "Componentes HTML + CSS"
date:   2018-03-31 01:37:53 -0300
categories: jekyll update
---

Los componentes "HTML + CSS" (desde ahora simplemente llamados componentes) son módulos que constan
basicamente de html y css en alguna forma, como puede ser un mixin de pug o instrucciones de sass.

Los componentes deben ser agnosticos del contexto y por lo tanto ser funcionales en cualquier circunstancia,
como por ejemplo podria ser cualquier resolución de pantalla en la que sea mostrado.

Para lograr buenos componentes es necesario abstraerse del contexto en el momento en que van a ser definidos:
- Se necesita agregar margen?
- Se necesita definir un color de fondo?
- A partir de que tamaño mínimo de resolución se dará soporte?
- Va a cambiar el layout del mismo para algún tamaño de resolución?

Son muchas las preguntas que uno puede formularse, pero nos vamos a centrar en algunas que son básicas y aplican a la
mayoria de casos de uso.

# Los componentes no necesitan margen
Los componentes no necesitan margen __de ningún tipo__. Dado que no sabemos en que contexto se va a utilizar, no sabemos
si el cliente va a querer o necesitar ese margen que estariamos agregando. Si lo agregáramos entonces el tendria que aplicar
márgenes especiales para salvar nuestro caso rompiendo con su esquema de maqueta. Aplica de la misma manera para los paddings
más externos.

# Los componentes no necesitan reglas de posicionamiento
Dado que no sabemos donde los clientes pueden querer posicionado el componente, no se debe especificar que por ejemplo
el mismo va a tener posicionamiento absoluto. No obstante si puede definirse posicionamiento relativo con el fin de guardar
contexto para los submódulos que compongan nuestro componente.

Por ejemplo, si `box` es el nombre de nuestro componente entonces
```css
    .box {
        /*
        Está bien, ya que .box__sub-box se posicionará
        absolutamente sobre .box y no sobre el contexto general.
        No obstante definir
        position: absolute;
        Está mal ya que no conocemos como es el ambiente donde se está
        utilizando nuestro componente y podría verse en un lugar donde
        no se espera.
        */
        position: relative;
    }
    .box__sub-box {
        position: absolute;
        top: 0;
        left: 0;
    }
```

# Los componentes tienen una definición de bloque básica
Es necesario definirlos como elementos de bloque para que puedan tener un ancho y un alto más allá de que lo
estemos maquetando sobre un elemento que de por sí sea de bloque. También es importante definir el ancho y el alto
al 100% con el fin de que se ajuste a cualquier tipo de contenedor, así como el tamaño de la fuente, el color de fondo de soporte
entre otras propiedades menos relevantes. Por ejemplo, con lo visto hasta ahora:

```css
    .box {
        display: block;
        width: 100%;
        height: 100%;
        box-sizing: border-box;
        position: relative;
        margin: 0;
        padding: 0;
        font-size: 1rem;
        background-color: #fff;
        color: #444;
        min-width: 20em;
    }
```

La definición de estas propiedades sobre el componente constituye un "mini reset" y es muy importante para asegurar
su comportamiento de bloque respecto a otros componentes.

# Los componentes deben utilizar unidades de medida relativas al tamaño de la fuente
Hay dos tipos de elementos html en base a sus estilos: los que definen font-size y los que no.

Si definen `font-size`, entonces el mismo debe estar en `rem`, es decir, tamaño relativo al `font-size`
del root (html o body). Si no definen `font-size`, entonces se debe asumir que es el heredado.

Si el elemento que se esta estilando define alguna propiedad (no `font-size`) que haga uso de unidades entonces
esta, debería estar en `em`, es decir, relativo al tamaño de la fuente del nodo en cuestión. Notar que es necesario tener
en cuenta si se esta heredando ese temaño o si se define en el mismo elemento.

Veamos un ejemplo:

```css
    .box {
        font-size: 12px;
        min-width: 320px;
    }
    .box__sub-box {
        /* font-size heredado 12px */
        margin-bottom: 10px;
    }
```
con un tamaño de fuente del root en 16px (por defecto en general), debería quedar
``` css
    .box {
        font-size: 0.75rem;
        min-width: 20em;
    }
    .box__sub-box {
        margin-bottom: 0.833em;
    }
```

Con esto se logra un componente más "responsive" en temas de accesibilidad. Algunos browsers
ofrecen la posibilidad de cambiar el tamaño de fuente por defecto con tal de lograr que los sitios se vean
más grandes o más chicos (una especie de zoom basado en tamaño de fuente del root).

¿Cómo saber cual es el tamaño de fuente del root?

Con css no se puede y obviamente tampoco con ningún metalenguage como sass. Lo que se puede hacer es exportar
los mixins o el componente para que sea compilado pidiendo entre sus parámetros el tamaño de fuente del root para
realizar los cálculos y generar las unidades correspondientes.

En [github](//www.linkedin.com/in/garciagomezluis/) dejo un par de funciones de sass que utilizo generalmente para calcular en base al tamaño de la fuente
correspondiente el valor de las unidades.

# Sobre cambios de look&feel
Muchas veces hacemos lo siguiente
```css
    .box {
        color: red;
    }
    .box.box--blue {
        color: blue;
    }
```

Lo cierto es que si queremos dar la posibilidad de que el cliente cambie alguna característica visual
de nuestro componente, no es necesario enviar todas las reglas en la hoja que define la visualización del
componente base (aquel primario que se verá bien en todas las resoluciones). A veces podemos estar duplicando
el tamaño de la hoja de estilos productiva con elementos que no van a ser utilizados. En cambio lo que se puede hacer
es lo siguiente:

```css
    .box {
        color: red;
    }
    @mixin box--color-blue {
        .box {
            color: blue;
        }
    }
```

y dejar que el cliente decida bajo que circunstancia utilizar el mixin que da el nuevo estilado.

# No pienses en 320, 768, 1024, ...
Si: pensá en "mobile first" en el sentido de maquetar el componente para dispositivos de baja resolución e ir avanzando
a partir de ahí. Lo cierto es que no sabemos en que contexto se va a utilizar el mismo, por el que no tiene mucho
sentido de hablar de 320, 768, 1024, ... Lo que si se puede hacer es dar un estilado base que se corresponde con
el que se va a mostrar en dispositivos de la más baja resolución para la que demos soporte y a partir de ahi ir avanzando.
¿Pero cómo? Seguro no mediante media queries. Una manera de hacerlo es la siguiente:

```css
    .box {
        /* estilos de soporte para la resolución más baja */
        color: red;
    }
    @mixin box--wide-1 {
        .box {
            /* estilos de soporte para la siguiente resolución más grande */
            color: blue;
        }
    }
    @mixin box--wide-2 {
        .box {
            /* estilos de soporte para la siguiente resolución más grande */
            color: green;
        }
    }
```
y dejar la implementación de las media queries del lado del cliente, que es quien conoce el contexto.
Fijate que esto es muy parecido a lo que mencionábamos anteriormente.

# Exportando clases
Hay veces que nos gustaria dejar que el cliente pueda definir ciertas carácterísticas de nuestro componente sin
exponer como esta formado internamente. Por ejemplo imaginemos que queremos dejar que `.box__sub-box` tome cualquier color
que el usuario del componente prefiera, en ese caso un workaround posible es el de exportar clases:

```html
    <!-- componente box -->
    <!-- clases exportadas:
    -   box-sub-box-background-color: se puede especificar
        unicamente la propiedad background-color
    -->
    <div class="box">
        <div class="box__sub-box box-sub-box-background-color">
        </div>
    </div>
```

De esta manera el cliente tiene la potestad de dejar a `.box__sub-box` el color de fondo que prefiera sin exponer
la estructura interna. Notar que en este punto la documentación del componente es muy importante y supone un contrato,
como toda documentación entro desarrollador y usuario.

# Elección de nombres
Hay ciertas metodologías que determinan una manera de nombrar componentes, como por ejemplo [BEM](//bem.info/).
Sea cual fuere la metodología es importante que los nombres no sean largos ni complicados de entender.

A veces se encuentra conveniente tener la opción de prefijar las clases que se ven implicadas. La razón principal
se encuentra en evitar la colisión de nombres de clases así como evitar el conflicto a la hora de hacer un release
de una nueva versión del componente.

A veces también se encuentra conveniente prefijar internamente las clases para podes distinguir aquellas que
serán utilizadas por css, de aquellas que serán utilizadas por js (de ser necesario). Por ejemplo entre
`.box-ui__sub--box` y `.box-js__sub--box`

# ¿Y si soy el "cliente"?
Si sos el cliente las responsabilidades que te tocan (entre otras) son:
- compilar unicamente los estilos necesarios para tus casos de uso
- definir las media queries
- definir comportamiento de clases exportadas si aplica
- documentar y reportar errores

Veamos un ejemplo: Supongamos que tenemos el siguiente componente a disposición

```html
    <div class="box">
        <div class="box__sub-box box-sub-box-background-color">
            "hola mundo!"
        </div>
    </div>
```

``` css
    .box {
        /* estilos base de mi componente */
    }
    @media .box--wide-1 {
        .box {
            color: red;
        }
    }
    @media .box--wide-2 {
        .box {
            color: blue;
        }
    }
    @media .box--rounded {
        .box {
            border-radius: 100%;
        }
    }
```

Notemos que se está exportando la clase `.box-sub-box-background-color` asi como también definiendo dos mixins de sass
para utilizarlos en caso de querer ajustar el componente a una nueva resolución de pantalla y un mixin para ajustar los bordes
en caso de ser necesario.

Mientras tanto el cliente:

```html
    <div class="box-container-top">
        <div class="box">
            <div class="box__sub-box box-sub-box-background-color">
                "hola mundo!"
            </div>
        </div>
    </div>
    <div class="box-container-bottom">
        <div class="box">
            <div class="box__sub-box box-sub-box-background-color">
                "hola mundo!"
            </div>
        </div>
    </div>
```

```css
    .box-container-top {
        width: 25em; /* 400px */
        height: 25em;
        font-family: Arial;
        .box-sub-box-background-color {
            /* defino el valor para la clase exportada */
            background-color: red;
        }
    }
    .box-container-bottom {
        width: 50em; /* 800px */
        height: 50em;
        font-family: Helvetica;
    }
    @media (min-width: 1024px) {
        .box-continer-top {
            @include box--wide-1;
        }
        .box-container-bottom {
            @include box--rounded;
        }
    }
```
Por lo tanto luego de pasar la primer media query (y la única), el componente .box de la parte
superior tendra un look&feel correspondiente al definido en `box--wide-1`, mientras que el de
la parte inferior tendra los bordes redondeados.