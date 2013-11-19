---
layout: post
title: "Moviendo ventanas con la API de Windows, la historia de WinResize"
date:   2013-11-19 09:37:48
---

Para hacer pruebas de VS Anywhere en un entorno de producción, una de las cosas que solemos hacer es abrir 2 sesiones de Visual Studio a cada lado de la pantalla, de esta manera podemos hacer pruebas rápidas de características específicas.

<img class="aligncenter size-full wp-image-2478" alt="goal" src="http://robertoluis.files.wordpress.com/2013/11/goal.png" />

El problema es que esto requiere abrir 2 instancias de visual studio de manera manual, y fijarlas cada una a un lado de la pantalla, algo tedioso, aburrido, y, sobre todo, <strong>automatizable</strong>. Así que me lancé a la búsqueda de la verdad, qué me puede solucionar?

<h2>El camino recorrido</h2>

Si tenemos Windows 7 o superior podemos hacer uso de <strong>Aero Snap</strong>, que usando Win+Izquierda o Win+Derecha, fija una ventana a la izquierda o a la derecha de la pantalla. Parecía sencillo, solamente tendríamos que emular esta combinación de teclas, lo cual, en un principio era algo fácil de hacer:

En teoría, usando Powershell, podríamos hacerlo, de hecho, en la siguiente entrada de TechNet <a href="http://blogs.technet.com/b/heyscriptingguy/archive/2011/01/10/provide-input-to-applications-with-powershell.aspx">Provide Input to Applications with PowerShell</a> se explica cómo enviar comandos a una aplicación empleando SendKeys, siendo este código el resultado:
{% highlight powershell %}
add-type -AssemblyName microsoft.VisualBasic
add-type -AssemblyName System.Windows.Forms
Calc
start-sleep -Milliseconds 500
[Microsoft.VisualBasic.Interaction]::AppActivate("Calc")
[System.Windows.Forms.SendKeys]::SendWait("1{ADD}1=")
{% endhighlight %}
De acuerdo con la última línea, deberíamos poder enviar al programa cualquier combinación de teclas, así que el siguiente paso era buscar la tecla <strong>Windows</strong> en la referencia de SendKeys: <a href="http://msdn.microsoft.com/en-us/library/system.windows.forms.sendkeys.send(v=vs.110).aspx">http://msdn.microsoft.com/en-us/library/system.windows.forms.sendkeys.send(v=vs.110).aspx</a>

Sorpresa: <strong>no está</strong><strong>. </strong>Por otra parte, buscando un poco por internet, encontré que se podría simular la pulsación de la tecla <strong>Windows</strong> usando la  Ctrl + Esc.

Segunda sorpresa: <strong>SIMULA LA TECLA WINDOWS, y nada más</strong>. En el momento que se pulsa eso, se salta al escritorio, o al menú inicio, no siendo posible usarlo para simular una combinación de teclas que involucre Win + cualquier cosa, así que por este camino poco más podemos hacer.

Por otro lado, si lo que estamos haciendo es un atajo de teclado, seguramente corresponda con alguna API de windows que haga la acción de Aero Snap, pero no, no hay API, no hay documentación y no hay ninguna referencia salvo el nombre comercial. con lo cual este es otro callejón sin salida,

Aún quedaba una opción, que era mover directamente la ventana usando código, tras algo de búsqueda encontré este código, que permitía hacer lo que quería usando powershell:

https://gist.github.com/coldnebo/1148334

No era demasiado complejo, pero opté por hacer una pequeña aplicación C# para que me resolviera el problema, usando las llamadas nativas a la API de Windows a través de P/Invoke:
<h2>Importando las funciones y moviendo la ventana</h2>
Para usar funciones de la API de Windows hemos de copiar su cabecera en nuestra función, definirla como externa (el framework ya sabrá que hacer) y definir la DLL de la que realizaremos la importación, en este caso la función <strong>MoveWindow</strong> situada en <strong>user32.dll</strong>:

{% highlight c# %}
[DllImport("user32.dll")]
public static extern bool MoveWindow(IntPtr hWnd, int X, 
int Y, int nWidth, int nHeight, bool bRepaint);
{% endhighlight %}

Deberemos repetir este proceso para cada función que queramos importar, y además, para todo esto, necesitaremos agregar el siguiente <strong>using</strong><strong>:</strong>
<pre>using System.Runtime.InteropServices;</pre>
Para poder mover la ventana, necesitamos su <strong>hWnd</strong>, que es un identificador de ventana único. Hay muchas maneras de localizarlo, y una de ellas es a partir de la lista de procesos, seleccionando aquellos cuya ventana principal coincidiera con lo que estaba buscando, en este caso Visual Studio.

{% highlight c# %}
foreach (Process proc in Process.GetProcesses())
{
    if (proc.MainWindowTitle.Contains("Visual Studio"))
    {
        IntPtr handle = proc.MainWindowHandle;
        ...
        MoveWindow(handle ...);
        ...
    }
}
{% endhighlight %}
Para poder mover la pantalla necesitamos establecer un punto origen y un tamaño, y para ello podríamos o bien fijarlos manualmente, o basarnos en la resolución de la pantalla, usando para ello las siguientes funciones:
<pre>Screen.PrimaryScreen.WorkingArea.Width;
Screen.PrimaryScreen.WorkingArea.Height;</pre>
que se encuentran dentro del namespace<strong> System.Windows.Forms</strong>, que requiere además una referencia as<strong> System.Drawing</strong> desde nuestro proyecto.

Lo realmente interesante de esto, es que me permitía mover la ventana a la posición que quisiera con cualquier tamaño, así que lo aproveché para poder establecer cualquier número de ventanas, estando estas igualmente distribuidas.

Finalmente necesitamos un par de llamadas más a funciones de la API de Windows para maximizar la ventana antes de moverla, y para establecer el foco, que se importan de manera similar:
<pre>[DllImport("user32.dll")]
public static extern bool ShowWindow(IntPtr hWnd, int X);
[DllImport("user32.dll")]
public static extern bool SetFocus(IntPtr hWnd);</pre>
<h2>Lanzando las aplicaciones y parámetros de entrada</h2>
Además de la funcionalidad de mover ventanas, podría ser interesante lanzar las aplicaciones, así que un poco de código para generar un proceso, establecer el nombre del fichero, y, tras lanzarlo, una leve espera mientras carga:
<pre>Process p = new Process();
p.StartInfo.FileName = "notepad.exe";
p.Start();
System.Threading.Thread.Sleep(2000);</pre>
Para hacerlo más sencillo, el último paso es convertir todos los parámetros usados a parámetros de entrada, siendo finalmente el proyecto una aplicación de consola, que recibe, en este orden;
<ul>
	<li>Ruta del ejecutable</li>
	<li>Título de la ventana a buscar</li>
	<li>Número de procesos a lanzar</li>
	<li>Espera para cada proceso</li>
</ul>
<span style="line-height: 1.714285714; font-size: 1rem;">La salida que muestra la aplicación de consola es ésta, para el caso de 3 ventanas de notepad.exe:</span>

<a href="http://robertoluis.files.wordpress.com/2013/11/winresize.png"><img class="aligncenter size-full wp-image-2483" alt="WinResize" src="http://robertoluis.files.wordpress.com/2013/11/winresize.png" /></a>

Por otra parte tenemos las 3 ventanas, igualmente distribuidas.

<a href="http://robertoluis.files.wordpress.com/2013/11/result.png"><img class="aligncenter size-full wp-image-2484" alt="Result" src="http://robertoluis.files.wordpress.com/2013/11/result.png" /></a>

Como último detalle, el título de la ventana está fijado usando el siguiente bloque de código:
<pre>Console.Title = "WinResize";</pre>
<h2>Conclusiones</h2>
Esta es una pequeña app que en mi caso soluciona un escenario muy concreto, se puede expandir hasta límites insospechados, y posiblemente lo haga, aunque hay cosas que se me han quedado en el tintero seguro, y que dejo como idea para el futuro:
<ul>
	<li>Hacer todo el script en powershell, es posible, y podemos hacer P/Invoke sin problemas</li>
	<li>Emular la pulsación de la tecla (tiene que haber un código asociado a la tecla windows, y una manera de pulsarlo!) simulando el teclado.</li>
	<li>En vez de recorrer todos los procesos, usar el identificador de los que acabo de crear.</li>
</ul>
El código está disponible de manera gratuita y bajo licencia open-source. Si quieres contribuir envía tu pull request :)

Enjoy!
<h2>Enlaces adicionales</h2>
<ul>
	<li>En Github:<a href="https://github.com/rlbisbe/WinResize">Código fuente de la aplicación</a></li>
	<li>En Stack Overflow: <a href="http://stackoverflow.com/questions/18313673/programmatically-maximize-window-on-half-of-screen">Programmatically Maximize Window On Half Of Screen</a></li>
	<li>En Stack Overflow: <a href="http://stackoverflow.com/questions/637652/get-the-handle-of-a-window-with-not-fully-known-title-c">Get the handle of a window with not fully known title. (C#)</a></li>
	<li>En MSDN: <a href="http://msdn.microsoft.com/en-us/library/system.console.title(v=vs.110).aspx">Console.Title</a></li>
	<li>En MSDN: <a href="http://msdn.microsoft.com/en-us/library/system.windows.forms.sendkeys.send(v=vs.110).aspx">SendKeys.Send</a></li>
	<li>En Github: <a href="https://gist.github.com/coldnebo/1148334">Resizing a Minecraft Window</a></li>
</ul>