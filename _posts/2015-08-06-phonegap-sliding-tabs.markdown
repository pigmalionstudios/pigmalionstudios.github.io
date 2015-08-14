---
layout: post
title:  "Phonegap: Sliding Tabs - Parte 1"
date:   2015-08-06 12:31:45
categories: jekyll update
author: Por Gustavo Alejandro Gramajo
---

Porque un video vale mas que mil palabras, veamos primero la app en acción en `iOS`...

<iframe width="420" height="315" src="https://www.youtube.com/embed/oGDxMchEfnU" frameborder="0" allowfullscreen></iframe>

... y en `Android`

<iframe width="420" height="315" src="https://www.youtube.com/embed/j-gJP90QKcU" frameborder="0" allowfullscreen></iframe>

¿Te gusta lo que viste? ¡Entonces comencemos!

Esta app cuenta con las siguientes caracteristicas principales:

- Multiplataforma, con experiencia muy cercana a la nativa, gracias a <a href="http://phonegap.com/">Phonegap (Version 4.1.2)</a>. Optimizado para `iOS`, `Android` y `Blackberry 10`
- Swipe entre pantallas, a través de gestures o tabs.
- Tabs animados al mejor estilo `Play Store de Google`
- Scrolling pensado para Mobile (Usando <a href="https://www.filamentgroup.com/lab/overthrow.html">Overthrow.js</a>)
- Uso de <a href="http://dojotoolkit.org/">Dojo</a> (sin widgets)
- Arquitectura Master-Detail

Vale aclarar que no se pretende explicar el funcionamiento de los frameworks aquí utilizados (Dojo, Overthrow, Phonegap); para mas informaciòn, recomendamos ir directamente a las respectivas fuentes.

<b>¿Por que Dojo?</b>

Dojo esta construido de una forma tal que favorece la modularización del código y la encapsulación. Provee un muy buen manejo del DOM, carga asincrònica de modulos y es muy liviano.
Es relativamente simple, luego de familiarizarse con el framework, obtener un prototipo navegable gracias al uso de widgets. Y con un look & feel adecuado para cada plataforma! (Dojo aplica los temas de forma dinamica).

<b>¿Por que no Dojo?</b>

Lamentablemente, la pobre fluidez del scrolleo y algunos bugs de interacción entre sus propios Widgets (`ScrollViews` y `SwapViews`)especialmente en DOMS complejos, hacen que sea mejor optar por una implementación propia de dichos elementos, en pos de llegar a una mayor cantidad de dispositivos, y al mismo tiempo mantener una alta calidad en experiencia de usuario.

<b>Arquitectura y funcionamiento</b>
 
Comencemos primero por mostrar la estructura de archivos del proyecto:

<div style="clear: both; height: 350px;">
    <img src="{{ site.url }}/assets/project_structure.png" style="height: 100%;">
</div>

<br>

Tengan en mente que nos centraremos exclusivamente en el contenido de `www/js/App` y en `index.html`.

<b>Tabs</b>


<div style="clear: both;">
    <img src="{{ site.url }}/assets/tabs.png" style="height: 100%;">
</div>

<br>

La estructura html para los tabs es muy sencilla:

{% highlight ruby %}

<div id="selectableTabs" class="menu">

    <div id="1001" class="tab" style="opacity: 1">MAIN</div>
    <div class="tabSeparator"></div>
    <div id="1002" class="tab">NEWS</div>                   
    <div class="tabSeparator"></div>
    <div id="1003" class="tab">POPULAR</div>
    <div class="tabSeparator"></div>
    <div id="1004" class="tab">FAVORITES</div>
    <div class="tabSeparator"></div>
    <div id="1005" class="tab">CONFIGURATION</div>
    <div class="tabSeparator"></div>
    <div id="1006" class="tab">ABOUT...</div>

</div>

{% endhighlight %}

Es `MUY IMPORTANTE` usar ids numéricos y consecutivos en los divs, ya que la implementación se basa en ese supuesto.

<b>Swipe</b>

Toda la funcionalidad está en `SwipeManager.js`. La implementación es un tanto compleja, pero existen algunos settings que se pueden modificar rapidamente acorde a los gustos y/o necesidades:

{% highlight ruby %}

var MIN_TOLERANCE_X_FOR_SWIPE = 40; // pasados los 40 px, al levantar el dedo el swipe continuara hasta alcanzar la siguiente pantalla
var PAGE_TRANSITION_TIME = "0.3s"; // el tiempo que toma en hacer el swipe luego de levandar el dedo
var TAB_TRANSITION_TIME = PAGE_TRANSITION_TIME; // el tiempo que toma en hacer el swipe en los tabs luego de terminar el click en algun tab
var ENABLE_MANUAL_DRAG_ON_TABS = true; //este feature todavia está en beta, pero pueden ir probandolo; activa o desactiva el swipe dentro de los tabs

{% endhighlight %}

`SwipeManager.js` maneja los gestos de swipe, dispara callbacks cuando una transiciòn sucede (a través de `ViewTransitionListener.viewTransitioned()`), controla el estado  de los tabs y sus correspondientes animaciones. La entidad incluso generará un mensaje de error si la cantidad de páginas es distinta a la cantidad de tabs.

<b>ViewTransitionListener</b>

Reacciona ante las transiciones (en la implementación actual, cambia el color del header del detail para que sea consistente con el del tab correspondiente) y guarda el id de la vista/tab actual. Esto último permite saber en todo momento cual de todas las vistas principales está siendo mostrada en pantalla. 

<b>Paginas y Detalle</b>

Los nodos `master` y `detail` son hermanos dentro de `body`. 
Para ver el detalle de un item en particular se hace un toggle entre ambas (show `detail`, hide `master`).
Para volver a `master`, se hace la inversa.
Esta lógica es manejada por `DetailView.js`.

{% highlight ruby %}

<body>

    <div id="master" class="master">
        
        ...
        ...

        <div id="pages" class="pages">

            <div id="page_main" class="page overthrow">
            </div>

            <div id="page_news" class="page overthrow">
            </div>

            ...
            ...
                
        </div>

    </div> 

    <div id="detail" style="display: none;">

        ...
        ...
        <div id="detail_body" class="detail">
            Here goes your detailed data
        </div>

    </div>

</body>

{% endhighlight %}

Por ejemplo, en estas capturas `master` esta visible (mostrando `page_about` y `page_favorites`)

<div style="clear: both; height: 350px;">

    <img src="{{ site.url }}/assets/master_about.png" style="float: left; height: 100%;">
    <img src="{{ site.url }}/assets/master_favorites.png" style="float: left; height: 100%;">

</div>

<br>

Y en esta, es `detail` quien está visible (luego de hacer tap en `GO TO DETAIL VIEW`, desde `page_news`)

<div style="clear: both; height: 350px;">
    <img src="{{ site.url }}/assets/detail_news.png" style="height: 100%;">
</div>


<br>

<b>UIListener</b>

Finalmente, tenemos al encargado de los eventos de la interfaz. Tal vez un nombre más adecuado podria ser StaticUIListener, dado que solo se encarga de los elementos estáticos de toda la app (no hay elementos dinámicos en esta demo).

<b>Personalizar la cantidad de páginas/tabs</b>

Los pasos a seguir si, por ejemplo, se quiere agregar un tab y una página más son:

- Dentro de `selectableTabs`, y despues del ultimo tab existente, se debe agregar

{% highlight ruby %}

<div class="tabSeparator"></div>
<div id="1007" class="tab">SOCIAL</div>

{% endhighlight %}

- Dentro de `pages`, y después de la última página existente, se debe agregar

{% highlight ruby %}

<div id="page_social" class="page overthrow">
</div>

{% endhighlight %}

- En `ViewTransitionListener`, modificar el array `HEADER_STYLES`, agregando un nuevo key/value.
La clave debe ser "1007"; el valor, el nombre de una clase css que se desee:

{% highlight ruby %}

var HEADER_STYLES = {"1001" : "color_main", "1002" : "color_news", "1003" : "color_popular", 
    "1004" : "color_favorites", "1005" : "color_configuration", "1006" : "color_about" ,"1007" : "color_social"};

{% endhighlight %}

<b>Corolario</b>

El propósito de este framework es proveer un punto de partida veloz, configurable y personalizable para quienes tengan en mente crear una app Master-Detail multiplataforma. Como desarrollador, sólo deberás encargarte de definir la cantidad de páginas/tabs, el contenido de cada una de ellas y de darles estilo (ya sea inline o agregando las clases en styles.css).<br>
Respecto a esto último, cabe mencionar que intencionalmente se ha dejado a `page_news`, `page_popular` y `page_favorites` sin un estilo aplicado (chequeenlo en la app, no tienen siquiera un margin-left o margin-top), y por el contrario, `detail`, `page_configuration`y `page_about` si los tienen (`page_main`seria algo intermedio; no se ve tan mal, pero sigue luciendo intencionalmente descuidado).
Los que si lo tienen, demuestran cómo puede verse una página con tan sólo un poco de estilo. Los que no, reflejan nuestro deseo de que creen sus propias páginas, con los estilos que más les gusten. Nosotros ofrecemos la arquitectura, pero en sus manos está darles su toque personal y distintivo...  

<b>Finalizando...</b>

A esta altura solo resta dejarles el acceso al <a href="https://github.com/pigmalionstudios/sliding-tabs">repo git</a>, instarlos a adentrarse en el còdigo, modificarlo y jugar con él.
¡Ah! Y no dejen de chequear `Educ.ar Noticias`, una app basada totalmente en la solución recien presentada, publicada en los stores de
<a href="https://play.google.com/store/apps/details?id=ar.gob.educarnoticias&hl=es_419">Android</a>, <a href="https://itunes.apple.com/us/app/educ.ar-noticias/id896186536?l=es&mt=8">IOS</a> y Blackberry 10.
No olviden dejar sus comentarios al final del post. ¡Espero que les haya sido útil!

Por `Gustavo Alejandro Gramajo`, para Pigmalion Studios.

<b>Próximos posts</b>

Sliding Tabs - Parte 2: Manejo de contenido dinámico. Llamadas ajax, obtención de datos, creación, populado y captura de eventos de listas.<br>
Sliding Tabs - Parte 3: Pull to refresh / Pull to page. Extensión de `SwipeManager.js` para soportar ambos métodos de actualización de contenidos.

