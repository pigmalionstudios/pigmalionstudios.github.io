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

- Multiplataforma, con experiencia muy cercana a la nativa, gracias a <a href="http://phonegap.com/">Phonegap</a>. Optimizado para `iOS`, `Android` y `Blackberry 10`
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

<div id="swipeableTabs" class="menu">

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

    <!-- UNCOMMENT THIS TO GET MORE TABS; CHECK ALSO MIN_TAB_ID, MAX_TAB_ID AND HEADER_STYLES IN CONFIG.JS
         ALWAYS USE CONSECUTIVE ID NUMBERS!!
    <div id="1007" class="tab">OTHER APPS</div>
    <div class="tabSeparator"></div>
    <div id="1008" class="tab">SOCIAL</div>
    <div class="tabSeparator"></div>
    <div id="1009" class="tab">FRIENDS</div>
    -->
</div>
<div id="currTabSelected" class="tab_underline"></div>

{% endhighlight %}

El id del primer (`MIN_TAB_ID`) y del último (`MAX_TAB_ID`) tab de la app siempre debe ser igual a los especificados en `Config.js`:

{% highlight ruby %}

...
MIN_TAB_ID: 1001, // "MAIN" TAB
MAX_TAB_ID: 1006, // "ABOUT" TAB
HEADER_STYLES: {"1001" : "color_main", "1002" : "color_news", "1003" : "color_popular", 
        "1004" : "color_favorites", "1005" : "color_configuration", "1006" : "color_about"}
...

{% endhighlight %}

El array `HEADER_STYLES` debe ser modificado también si se aumenta o disminuye la cantidad de tabs. Este vector contiene los estilos que se aplican al header cada vez que se transiciona a un nuevo tab (o se accede al detalle de un tab).

<b>Swipe</b>

Toda la funcionalidad está en `SwipeManager.js`. La implementación es un tanto compleja, pero existen algunos settings que se pueden modificar rapidamente acorde a los gustos y/o necesidades:

{% highlight ruby %}

var MIN_TOLERANCE_X_FOR_SWIPE = 40; // pasados los 40 px, al levantar el dedo el swipe continuara hasta alcanzar la siguiente pantalla
var TRANSITION_TIME = 300; // el tiempo que toma en hacer el swipe

{% endhighlight %}

`SwipeManager.js` maneja los gestos de swipe, dispara callbacks cuando una transiciòn sucede (a través de `ViewTransitionListener.viewTransitioned()`), controla el estado  de los tabs y sus correspondientes animaciones.

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

<b>Finalizando...</b>

A esta altura solo resta dejarles el acceso al <a href="https://github.com/pigmalionstudios/sliding-tabs">repo git</a>, instarlos a adentrarse en el còdigo, modificarlo, jugar con él y dejar sus comentarios al final del post. ¡Espero que les haya sido útil!

Por `Gustavo Alejandro Gramajo`, para Pigmalion Studios.

<b>Próximos posts</b>

Sliding Tabs - Parte 2: Manejo de contenido dinámico. Llamadas ajax, obtención de datos, creación, populado y captura de eventos de listas.<br>
Sliding Tabs - Parte 3: Pull to refresh / Pull to page. Extensión de `SwipeManager.js` para soportar ambos métodos de actualización de contenidos.

