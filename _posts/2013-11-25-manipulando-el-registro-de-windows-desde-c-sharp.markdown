---
layout: post
title: "Manipulando el registro de Windows desde aplicaciones .NET"
date: 2013-11-26 11:00
comments: true
categories: []
published: false
---

El registro de windows que tantos dolores de cabeza nos ha dado no es más que una manera de almacenar información en forma de árbol. Se emplea para almacenar preferencias por parte de aplicaciones y por el sistema operativo. En este artículo veremos cómo acceder, almacenar y borrar claves del mismo desde nuestra aplicación.

# Introducción.

Lo primero que hemos de tener en cuenta es que el registro es algo "delicado", ya que se comparte por todas las aplicaciones del sistema (de hecho, las aplicaciones Windows Store para Windows 8 no tienen acceso al mismo), así que cuando lo manipulemos, lo recomendable es limitarnos a la carpeta que almacena la información de nuestra aplicación.

Existen múltiples rutas, las 2 más conocidas son

Lo primero que tenemos que entender es que el registro de Windows posee 4 claves fundamentales a las que se nombra de la siguiente manera:

* **HKEY\_LOCAL\_MACHINE** (o HKLM), que contiene información a nivel global del sistema.
* **HKEY\_CURRENT\_USER** (o HKCU), que contiene información relativa a la sesión del usuario actual.

Para usar el registro en nuestras aplicaciones necesitamos la siguiente directiva:

{% highlight csharp %}
using Microsoft.Win32;
{% endhighlight %}

# Accediendo a una clave y obteniendo valores

Para acceder a la información contenida en una clave, en primer lugar accederemos al campo **CurrentUser**, equivalente a HKCU definido anteriormente, luego a la carpeta "Software" y finalmente a la carpeta de nuestra aplicación.

{% highlight csharp %}
string name = "Foo";
RegistryKey key = Registry.CurrentUser.OpenSubKey(@"Software\MyApp\", true);
object selectedValue = key.GetValue(name);
{% endhighlight %}

El true es para modo escritura, si no lo ponemos se obtendrá la clave en modo lectura, y cualquier operación de escritura que se intente sobre la misma devolverá como resultado una **excepción**. En caso de que la clave buscada no se encuentre, obtendremos un nulo como respuesta. Una vez hayamos accedido a la clave, podremos obtener el valor, que será de tipo objeto.

# Creando una clave e instertando valores

Para crear una clave, el proceso es similar al anterior, sin embargo, en este caso los permisos por defecto incluyen la escritura:

{% highlight csharp %}
string name = "Foo";
string value = "Bar";
key = Registry.CurrentUser.CreateSubKey(@"Software\MyApp\");
key.SetValue(name, value);
{% endhighlight %}

Una vez creada, se pueden insertar los valores de manera sencilla.

# Borrando valores en claves

Finalmente, para eliminar un valor de una clave necesitamos acceder a ellos de la misma manera que hemos hecho antes:

{% highlight csharp %}
RegistryKey key = Registry.CurrentUser.OpenSubKey(@"Software\MyApp\", true);
key.DeleteValue(name);
{% endhighlight %}

El borrado de valores se realiza de manera similar a la consulta, devolviendo una excepción si no se encuentra el valor a borrar.

# Resumen

En este breve artículo hemos visto cómo crear subclaves de registro, cómo acceder a las mismas y a sus valores, y finalmente, cómo borrar los valores de las claves. El artículo de la documentación de MSDN contiene, además, sobrecargas de los métodos que hemos visto aquí.

# Enlaces adicionales

* [Registry Hive](http://pcsupport.about.com/od/termsr/g/registryhive.htm)
* En MSDN [Registry Key](http://msdn.microsoft.com/en-us/library/microsoft.win32.registrykey.aspx)
