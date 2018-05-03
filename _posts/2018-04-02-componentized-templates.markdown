---
layout: post
title:  "Componentización de html y css"
date:   2018-04-02
categories: jekyll update
published: true
---

En grandes aplicaciones web, muchas veces es necesario mantener el look and feel
de los módulos presentes, contribuyendo así con la consistencia y armonía del proyecto. A su vez diferentes equipos están a cargo del desarrollo y mantenimiento de estos módulos por lo que se vuelve sumamente importante:
- lograr que los módulos se puedan distribuir facilmente
- que la integración de los mismos con el resto de las aplicaciones sea limpia

En esta oportunidad voy a comentar un poco como suelo llevar a cabo la "componentización" del html y css que componen nuestros módulos a fin de que puedan ser facilmente integrables al menos desde el punto de vista de la maqueta.

## ¿A qué nos referimos cuando decimos "componentización de html y css"?
Todos los componentes que se generan, se __abstraen del contexto__ o mejor 
dicho son __agnósticos del contexto__.


¿Qué significa esto?

Que el componente toma exclusivamente aquellas cosas que necesita para poder 
cumplir con su función y a cambio, este debe funcionar independientemente de 
donde y bajo que otras circunstancias se esté utilizando.

Los componentes que vamos a caracterizar son aquellos que constan basicamente
de html en alguna forma, como puede ser por ejemplo un mixin de pug y css en
otra forma como puede ser sass.

## ¿Y por donde empezamos?

Para lograr la abstracción requerida en general es bueno preguntarse que cosas son necesarias para que todo funcione:
- ¿se necesita definir un color de fondo?
- ¿se necesita agregar margen?
- ¿va a cambiar el layout del mismo dependiendo del dispositivo donde se vea?
- ¿a partir de que tamaño mínimo de viewport se dará soporte?

Son muchas las preguntas que uno puede formularse, pero vamos a tratar de dar una respuesta a aquellas que aplican a la mayoría de casos de uso.

# ¿Cómo definir un componente?
En general es común ver componentes definidos a partir de un `div` con una clase con un nombre corto y descriptivo.

```html
    mixin box(model)
        .box
            p.box__sub-box
                | "hello world!"
```

así como sus correspondientes estilos
```scss
    .box {
        ...
        &__sub-box {
            ...
        }
    }
```

# Los componentes no necesitan margen
Los componentes no necesitan margen __de ningún tipo__. Dado que no sabemos en que contexto se va a utilizar, no sabemos si el cliente va a querer o necesitar ese margen que estamos agregando. Si lo agregamos entonces él va a tener que aplicar
márgenes especiales para salvar nuestro caso rompiendo con su esquema de maqueta. Aplica de la misma manera para los paddings más externos.

Si es necesario un margen externo, es responsabilidad del cliente definirlo 
mediante un contenedor.

<div class="row">
    <div>
        <img src="../../../assets/images/facebookemojipicker.jpeg" alt="facebook emoji picker" />
        <p class="caption">
            facebook emoji picker con padding externo
        </p>
    </div>
</div>
Como se puede ver, el facebook emoji picker es un componente que asume que el cliente va a necesitar un padding externo, lo cual no siempre es cierto.

# Los componentes deberían tener explicito `position: relative`
Dado que no sabemos donde los clientes pueden querer posicionado el componente, no
se debe especificar que por ejemplo el mismo va a tener `position: absolute` o `position: fixed`. No obstante, debería definirse posicionamiento relativo con el fin de guardar el contexto para los submódulos que compongan nuestro componente. Por ejemplo:

```scss
    .box {
        /*
        Estaría bien ya que .box__sub-box se posiciona
        absolutamente sobre .box y no sobre el contexto general.
        No obstante definir
        position: absolute;
        No estaría del todo bien ya que no conocemos como es el ambiente
        donde se está utilizando nuestro componente y podría
        verse en un lugar donde no se espera.
        */
        position: relative;
        &__sub-box {
            position: absolute;
            top: 0;
            left: 0;
        }
    }
```

# Los componentes deberían utilizar unidades de medida relativas al tamaño de la fuente
Hay dos tipos de elementos html en base a sus estilos: los que definen `font-size` y los que no.

Si definen `font-size`, entonces el mismo debería estar en `rem`, es decir, tamaño relativo al `font-size`
del root (html o body). Si no definen `font-size`, entonces se debe asumir que es el heredado.

Si el elemento que se está estilando define alguna propiedad (no `font-size`) que haga uso de unidades entonces
la misma, debería estar en `em`, es decir, relativo al tamaño de la fuente del nodo en cuestión. Notar que es necesario tener
en cuenta si se esta heredando ese temaño o si se define en el mismo elemento.

Veamos un ejemplo:

```scss
    .box {
        font-family: Helvetica, Arial, sans-serif;
        font-size: 12px;
        min-width: 320px;
        &__sub-box {
            /* font-size heredado 12px */
            margin-bottom: 10px;
        }
    }
```
con un tamaño de fuente del root en 16px (por defecto en general), debería quedar
```scss
    .box {
        font-family: Helvetica, Arial, sans-serif;
        font-size: 0.75rem;
        min-width: 20em;
        &__sub-box {
            margin-bottom: 0.833em;
        }
    }
```
¿Qué se gana con esto?:
- se logra un componente más "responsive" en temas de accesibilidad.
Algunos browsers ofrecen la posibilidad de cambiar el tamaño de fuente por 
defecto con tal de lograr que los sitios se vean más grandes o más chicos 
(una especie de zoom basado en tamaño de fuente del root)
- por otra parte si el componente es utilizado en un documento cuyo root tiene un `font-size` diferente para el cual fue pensado, entonces basta con especificar este parámetro a la hora de compilar los estilos logrando que el mismo se "agrande" o "achique" de manera proporcional al resto del documento

<div class="row">
    <div>
        <img src="../../../assets/images/px.gif" alt="example-px">
        <p class="caption">
            componente con unidades en px
        </p>
    </div>
    <div>
        <img src="../../../assets/images/rem-em.gif" alt="example-rem-em">
        <p class="caption">
            componente con unidades en rem y em
        </p>
    </div>
</div>
Dejo a continuación un par de funciones de sass que utilizo generalmente para calcular en base al tamaño de la fuente correspondiente el valor de las unidades.

{% gist 2f013ea6b8e66da4e81ffff285ac7fc1 %}

¿Cómo saber cual es el tamaño de fuente del root?

Con css no se puede y obviamente tampoco con ningún metalenguage como sass.
Lo que se puede hacer es exportar el mixin o el componente para que sea 
compilado pidiendo entre sus parámetros el tamaño de fuente del root para
realizar los cálculos y generar las unidades correspondientes.

```scss
    @mixin box($context) {
        $browser-context: $context;
        .box {
            $node-font-size: 20px;
            font-family: Helvetica, Arial, sans-serif;
            font-size: rem($node-font-size);
            &__sub-box {
                $node-font-size: 16px;
                font-size: rem($node-font-size);
                margin-bottom: em(10px, $node-font-size);
            }
        }
    }
```

# Los componentes deberían definir atributos de fuentes
Definir `font-size` teniendo en cuenta lo mencionado anteriormente, asegura coherencia en las proporciones del mismo en distintos contextos. No cambiar de
fuente dependiendo del contexto contribuye a manetener esa coherencia y armonía que
buscábamos inicialmente. Por eso es necesario fijar al menos `font-family` también.

# Los componentes deberían definir `min-width` de soporte
Dado que los componentes son agnósticos de contexto, no se sabe a priori
el tipo de dispositivo donde va a ser renderizado, por lo que simplemente
puede no tener sentido definir el comportamiento del mismo por debajo de un ancho mínimo. Esta es una propidad que puede variar de componente en componente.

# Los componentes tienen una definición de bloque básica
Es necesario definirlos como elementos de bloque para que puedan tener un ancho 
y un alto independientemente de la etiqueta html con la que lo estemos definiendo.
También es importante definir el ancho y el alto al 100% con el fin de que 
se ajuste a cualquier tipo de contenedor, así como el tamaño de la fuente, 
el color de fondo de soporte entre otras propiedades menos relevantes.
Por ejemplo, con lo visto hasta ahora puede definirse `.component`:

```scss
    .component {
        // atributos comunes a todos los componentes
        // que los definen como elementos de bloque
        display: block;
        width: 100%;
        height: 100%;
        box-sizing: border-box;
        position: relative;
        margin: 0;
        padding: 0;
        list-style-type: none;
        border: none;
    }
    .box {
        @extend .component;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 1rem;
        min-width: 20em;
    }
```

La definición de estas propiedades sobre el componente constituye un "mini reset"
 y es muy importante para asegurar su comportamiento de bloque respecto a 
otros componentes. No olvidemos que desconocemos el contexto en el que está
siendo utilizado.

# Atributos laxos
Los componentes pueden definir ciertos atributos relacionados a los colores por
ejemplo o bien delegar esa resposabilidad al cliente (haciendo uso del valor `inherit` para cada propiedad).

# Sobre cambios de look and feel (exportando estados)
Muchas veces hacemos lo siguiente
```scss
    .box {
        width: 20em;
        &--big {
            // definición de un modificador
            width: 30em;
        }
    }
```

Lo cierto es que si queremos dar la posibilidad de que el cliente cambie alguna característica visual
de nuestro componente, no es necesario enviar todas las reglas en la hoja que define la visualización del
componente base (aquel primario que se verá bien en todos los dispositivos, pensando en "mobile first"). A veces podemos estar duplicando
el tamaño de la hoja de estilos productiva con elementos que no van a ser utilizados. En cambio lo que se puede hacer
es lo siguiente:

```scss
    .box {
        width: 20em;
    }
    @mixin box--big {
        .box {
            width: 30em;
        }
    }
```

y dejar que el cliente decida bajo que circunstancia utilizar el mixin que da el nuevo estilado.

# No pienses en 320, 768, 1024,..., pensá en estados
Si: se puede pensar en "mobile first" en el sentido de maquetar el componente para dispositivos de baja resolución e ir avanzando
a partir de ahí. Lo cierto es que no sabemos en que contexto se va a utilizar el mismo, por el que no tiene mucho
sentido de hablar de 320, 768, 1024 (como media queries),... El cliente puede querer tener la vista del componente en 320 estándo en un dispositivo de 1024. Lo que si se puede hacer es dar un estilado base que se corresponde con
el que se va a mostrar en dispositivos de la más baja resolución para la que demos soporte y a partir de ahi ir avanzando.
¿Pero cómo? Una manera de hacerlo utilizando la idea mencionada antes es dejar de mirar por media queries y exportar los estados:

```scss
    .box {
        // estilos de soporte para la resolución más baja
        color: red;
    }
    @mixin box--wide-1 {
        .box {
            // estilos de soporte para la siguiente resolución más grande
            color: blue;
        }
    }
    @mixin box--wide-2 {
        .box {
            // estilos de soporte para la siguiente resolución más grande
            color: green;
        }
    }
```
y dejar la implementación de las media queries del lado del cliente, que es quien conoce el contexto y puede compilar los estados que exportamos.

# Exportando clases
Hay veces que nos gustaria dejar que el cliente pueda definir ciertas carácterísticas de nuestro componente sin
exponer como esta formado internamente. Exportar estados como vimos anteriormente puede servir, pero otorga un rango acotado de posibilidades para elegir. Es entonces que recurrimos a exportar clases. Por ejemplo imaginemos que queremos dejar que `.box__sub-box` tome cualquier color
que el usuario del componente prefiera, en ese caso podriamos definir:

```html
    //- componente box
        clases exportadas:
            box-sub-box-background-color: se puede especificar unicamente la propiedad background-color
    @mixin box(model)
        .box
            p.box__sub-box.box-sub-box-background-color
                | "hello world!"
```

De esta manera el cliente tiene la potestad de dejar a `.box__sub-box` el color de fondo que prefiera sin exponer la estructura interna.

# Elección de nombres
Hay ciertas metodologías que determinan una manera de nombrar componentes, como por ejemplo [BEM](//bem.info/).
Sea cual fuere la metodología es importante que los nombres no sean largos ni complicados de entender y  que todas las clases que se vean implicadas estén definidas especificamente para el componente en cuestión.

A veces se encuentra conveniente tener la opción de prefijar las clases que se ven implicadas. La razón principal
se encuentra en evitar la colisión de nombres de clases así como evitar el conflicto a la hora de hacer un release
de una nueva versión del componente.

A veces también se encuentra conveniente prefijar internamente las clases para podes distinguir aquellas que
serán utilizadas por css, de aquellas que serán utilizadas por JS (de ser necesario). Por ejemplo entre
`.box-ui__sub--box` y `.box-js__sub--box`

# Definiendo comentarios o documentación
Como en todos los componentes, debería haber una documentación asociada en la que se tendría que especificar entre otras cosas:
- nombre y versión del componente
- datos de contacto por soporte
- fecha de último release
- browsers soportados
- clases exportadas
- estados exportados
- cosas que se asumen del contexto
- esquema del modelo que aplica al template en caso de haber uno
- otros detalles

La documentación es la "puesta en papel" de los derechos y obligaciones del cliente y del componente a la hora de utilizarlos.

## Un ejemplo de juguete
Escribamos `box` con las cosas mencionadas hasta acá para que quede definido
como un componente que pueda ser fácil y limpiamente integrable.

```html
    //- box - v0.0.1 (04-05-2018)
        support@ux.com
        chrome +50, firefox +39, edge 11, otros
        clases exportadas:
            box-sub-box-background-color: se puede especificar
            unicamente la propiedad background-color
        estados exportados:
            box--wide-1: cambio de look and feel en espacios mayores
            (http://via.placeholder.com/350x150)
            box--wide-2: cambio de look and feel en espacios mayores
            (http://via.placeholder.com/700x150)
        atributos laxos:
            background-color y color
        - no tiene bordes redondeados
        - asume $browser-context = 16px
        - min-width: 320px
        - la fuente especificada es Helvetica, Arial, sans-serif
        model(object, no undefined) = {
            "message"(string, no undefined) = "hello world!"
        }
    @mixin box(model)
        .box
            p.box__sub-box.box-sub-box-background-color
                | #{model.message}
```

```scss
    .component {
        display: block;
        width: 100%;
        height: 100%;
        box-sizing: border-box;
        position: relative;
        margin: 0;
        padding: 0;
        list-style-type: none;
        border: none;
    }
    @mixin box($context:16px) {
        $browser-context: $context;
        .box {
            $node-font-size: 16px;
            @extend .component;
            font-family: Helvetica, Arial, sans-serif;
            font-size: rem($node-font-size);
            min-width: em(320px, $node-font-size);
            &__sub-box {
                $node-font-size: 12px;
                font-size: rem($node-font-size);
                margin-bottom: em(10px, $node-font-size);
            }
        }
    }
    @mixin .box--wide-1 {
        // estado 1: estilos para resoluciones más grandes
        .box {
            color: red;
        }
    }
    @mixin .box--wide-2 {
        // estado 2: estilos para resoluciones aún más grandes
        .box {
            color: blue;
        }
    }
    @mixin .box--rounded {
        // estado 3: especifica bordes redondeados
        .box {
            border-radius: 100%;
        }
    }
```

## ¿Y si somos el "cliente"?
Si somos el cliente las responsabilidades que nos tocan (entre otras) son:
- compilar unicamente los estilos necesarios para nuestros casos de uso
- definir las media queries
- definir comportamiento de clases exportadas si aplica
- documentar y reportar errores

Veamos un ejemplito de integración en base a la definición de `box` dada previamente

```html
    .box-container-top
        +box({ "message": "hello" })
    .box-container-bottom
        +box({ "message": "world" })
```

```scss
    $browser-context: 20px;
    // instanciamos los estilos del componente con un font-size del root en 20px
    include box($browser-context);

    .box-container-top {
        $node-font-size: 20px; // heredada
        width: em(400px, $node-font-size);
        height: em(300px, $node-font-size);
        .box-sub-box-background-color {
            // defino el valor para la clase exportada
            background-color: red;
        }
    }

    .box-container-bottom {
        $node-font-size: 20px; // heredada
        width: em(800px, $node-font-size);
        height: em(200px, $node-font-size);
        // en este caso no necesitamos cambiar el valor de la clase exportada
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
Por lo tanto luego de pasar la primer media query (y la única), el componente .box de la parte superior tendra un look and feel correspondiente al definido en `box--wide-1`, mientras que el de la parte inferior tendra un look and feel correspondiente al definido en `box--rounded`.

## Algunas ideas finales
Hasta acá tratamos de caracterizar un poco como estos componentes html + css deberían estar definidos para que puedan ser facilmente integrables en proyectos de gran envergadura mantenidos por diferentes equipos.

Nuestro principal objetivo era asegurar que estos componentes fueran componentes efectivamente asumiendo cosas del contexto pero luego abstrayéndose de él, delegando responsabilidades en el usuario o cliente del mismo.

Ya sea dejando a propiedades tomar el valor heradado en el componente, así como exportando estados o clases logramos dejar un margen de acción para que nuestro usuario pueda terminar de definir algunos aspectos básicos del look and feel del componente.

Muchas veces vamos a tener que tomar decisiones basadas en la lógica del negocio pero lo importante es dejar el código entendible para que pueda ser mantenido. Tanto para esto último así como para comunicar datos relevantes del componente los comentarios tienen un rol fundamental.

Si bien la llegada de Web Components es prometedora, aún falta iniciar el periodo de transición entre el draft y la estandarización entre los diferentes browsers, por lo cual, mantener nuestros componentes html + css es importante aún.
