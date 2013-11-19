---
layout: post
title: "Angular js para aplicaciones Windows Store: Primeras impresiones"
date: 2013-11-20
comments: true
---

Existe una gran variedad de frameworks javascript, Angular, Ember, knockout, etc. Todos ellos nos proporcionan algo que aquellos que hemos hecho wpf estábamos esperando: Data binding, y la posibilidad de usar patrones como MVVM.

En este artículo doy un vistazo a Angular.js, un framework desarrollado por Google que está dando mucho de que hablar, y, para hacerlo más interesante, he lo empleo dentro del contexto de una aplicación Windows 8, donde las reglas del juego cambian ligeramente.
<h2>Instalación y configuración</h2>
Para este proyecto, el primer paso es crear una aplicación vacía de Windows 8 con Javascript, podemos usar Visual Studio Express para ello.

Para incluir Angular.js, una cosa que podemos pensar es enlazar directamente a la CDN de angular desde nuestro fichero default.html:

{% highlight html %}<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.1/angular.min.js">
</script> {% endhighlight %}

Sin embargo nos encontramos con esto:

<a href="http://robertoluis.files.wordpress.com/2013/11/usingcdn.png"><img class="aligncenter size-full wp-image-2499" alt="usingcdn" src="http://robertoluis.files.wordpress.com/2013/11/usingcdn.png" width="625" height="147" /></a>

Es decir, no podemos usar una referencia no local para ficheros **javascript** con lo cual tendremos que descargarlo e incluirlo en nuestro proyecto manualmente. Podríamos además, estar tentados a instalarlo usando nuget, pero en mi opinión agrega demasiados archivos al proyecto.

Como anotación, recomiendo descargar la versión **angular.js**, no la **angular.min.js**, ya que tendremos que hacer algunos cambios más adelante.

Una vez tengamos el fichero agregado a nuestro proyecto, si intentamos compilar tendremos una segunda sorpresa:

<a href="http://robertoluis.files.wordpress.com/2013/11/epicexception.png"><img class="aligncenter size-full wp-image-2501" alt="epicException" src="http://robertoluis.files.wordpress.com/2013/11/epicexception.png" width="625" height="233" /></a>Hay muchas maneras de resolver este error, pero el que a mí me ha funcionado pasa por editar la función problemática, y sustituir la llamada a la función insertBefore por:

{% highlight ruby %}MSApp.execUnsafeLocalFunction(function (){element.insertBefore(child, index);}); {% endhighlight %}

Con este ligero cambio, ya podremos ejecutar nuestra aplicación:
<h2>Las vistas</h2>
Para mostrar información por pantalla, en vez de construir el HTML manualmente y editar el DOM desde código javascript, simplemente usamos ciertas convenciones para incrustar datos dentro de una plantilla HTML.

{% highlight ruby %}
<div><label>Name:</label>
 <input type="text" placeholder="Enter a name here" />

<hr />

<h1>Hello {{yourName}}!</h1>
</div>
</div>
 {% endhighlight %}

En este caso, el input genera una variable de nombre yourName, que se actualiza en el campo <strong>h1</strong>, en la región habilitada para ello. Estas variables con corchetes se pueden usar también dentro de los elementos html, por ejemplo para fijar la propiedad "src" de una imagen:.
{% highlight javascript %}<img src="{{item}}" /> {% endhighlight %}
No tenemos que preocuparnos por la actualización, ya que al cambiar el contenido del dato, sea cual sea, el valor será actualizado automáticamente. Esto representa una diferencia bastante interesante respecto a otros frameworks como Knockout, donde para que se actualizara el objeto teníamos que definirlo con un tipo muy específico.
<h2>Controladores</h2>
El controlador se define como una clase javascript, con propiedades y funciones, las cuales luego suelen ser llamadas desde el código fuente, por ejemplo:

{% highlight javascript %}

function TodoCtrl($scope) {
$scope.todos = [];

$scope.addTodo = function () {
$scope.todos.push({ text: $scope.todoText, done: false });
        $scope.todoText = '';
};

};

{% endhighlight %}

En este ejemplo se define un controlador, un campo llamado <strong>todos</strong>, y un método para almacenar en la lista el valor definido en la variable <strong>todoText</strong>. El $context es similar al this del standard, aunque con algunas variantes. La vista de este código se parece a algo así:

{% highlight javascript %}/pre>
<div>
<h2>Todo</h2>
<div>
{{todos.length}}
<form><input type="text" placeholder="add new todo here" size="30" />
 <input class="btn-primary" type="submit" value="add" /></form></div>
</div>
<pre>

{% endhighlight %}

A diferencia de lo visto anteriormente, en este código definimos un controlador, accedemos al campo length del objeto <strong>todos</strong> definido anteriormente, y finalmente llamamos al método como resultado del envío de formularios.
<h2>Bucles (y bucles dentro de bucles)</h2>
Otra de las cosas interesantes que tiene, es el uso de los bucles para recorrer información:

{% highlight javascript %}</pre>
<ul>
	<li>{{todo.name}}

 <img alt="" src="{{item}}" /></li>
</ul>
{% endhighlight %}

En este caso representamos la estructura de <strong>todos</strong>, y asumiendo que <strong>todo</strong> es una estructura compuesta, Estos bucles además se pueden anidar, consiguiendo una mayor libertad a la hora de representar datos complejos.
<h2>Llamadas HTTP</h2>
Para poder hacer llamadas a un servidor externo mediante http solamente necesitamos la url y el método por el cual realizaremos la petición, para efectuar la llamada de la siguiente manera:

{% highlight javascript %}

$http({ method: 'GET', url: 'http://url' }).
 success(function (data, status, headers, config) {

 }).
 error(function (data, status, headers, config) {

 });

{% endhighlight %}

Este código lo incluiremos en nuestro controlador sin llamadas adicionales, de tal manera que sea ejecutado al cargar el controlador. Si lo hacemos de esta manera nos encontraremos con un bonito error como el que se muestra a continuación: Para evitar ese error, hemos de modificar la definición de nuestro controlador:
<pre>function TodoCtrl($scope)</pre>
De tal manera que ahora además de $scope, recibe $http:
<pre>function TodoCtrl($scope. $http)</pre>
<h2>Conclusiones</h2>
Aunque solamente he rascado en la superficie, es suficiente para ver que va a dar mucho de que hablar en un futuro.