---
layout: post
title: "Viernes 22: Desarrollo para Windows Phone y Windows 8 en la Universidad de Salamanca"
date: 2013-11-18 17:41
comments: true
categories: []
---

Cuando trabajamos desarrollando aplicaciones de escritorio, antes o después tendremos que decidir si hacemos pruebas de interfaz. De acuerdo con el siguiente gráfico sobre buenas prácticas de desarrollo de software, estas pruebas, aunque no deberían ser todas las que tenga el sistema, deberían ser lo suficientemente buenas como para asegurarse de que la interfaz funciona bien.

![Automated testing pyramid]{}({{ site.url }}/assets/automatedtestingpyramid.png)

En el caso de las aplicaciones WPF, podríamos probar cosas como:

* Carga de recursos en XAML
* Data binding
* Escenarios completos en producción

En mi caso, nuestra app depende mucho de Visual Studio, con lo cual nos interesa probarla a fondo en un entorno de producción.

# Primera opción: Coded UI

Visual Studio incluye, desde 2010, una manera para generar tests automáticos conocida como Coded UI. Esta característica permite grabar las acciones del mouse.

# Segunda opción: Windows Automation

# White framework, una abstracción sobre Windows Automation