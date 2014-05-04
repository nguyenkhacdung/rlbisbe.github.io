---
layout: post
title: "Migrando de Wordpress a Jekyll"
date: 2013-11-18 17:41
comments: true
categories: []
---

Wordpress es una gran plataforma de gestión de contenidos, y Wordpress.com es un servicio muy potente que da alojamiento a millones de blogs cada día. Sin embargo, para mis necesidades, que son pocas pero muy específicas, no es la primera vez que me da problemas, ya que el editor no se lleva demasiado bien con HTML.

## Opciones disponibles

Lancé un par de tweets incendiarios hace un par de días:

Y la respuesta que tuve fue:

* Ghost
* Joomla
* "algo con Markdown"

De ghost hemos hablado ya en [este blog]() y parece una herramienta interesante, pero en este caso quería huir de los editores que hacían "magia" en la nube. Joomla me parecía un tanto desproporcionado para un blog. así que he tirado por la tercera opción. Para hacer algo con Markdown tenemos, por una parte [Jekyll]() y por la otra [Pretzel](). EL primero es un proyecto que lleva ya varios años, y del que también hemos hablado en este blog cuando hablamos de [trillo]()

Tanto jekyll como pretzel son generadores estáticos de contenido, con lo cual el resultado es una página 100% estática. No necesito más para mi blog.

## Instalando Jekyll

Instalar Jekyll en windows se nos puede antojar a un poco infierno. Por suerte, tenemos un paquete portable que incluye todo lo que podamos necesitar y que podemos obtener desde [aqui]()

## Importando información de Wordpress

De momento solamente estoy importando unos pocos artículos, ya que tendría que hacer varios cambios. El importador de Jekyll directamente no funciona en Windows 8.1 x64 por limitaciones en una biblioteca. Sin embargo tenemos este proyecto [wpXml...]() que lo que permite es, dado un fichero xml de exportación de wordpress, generar archivos markdown. Si has sido tan listo como yo para usar etiquetas como [sourcecode] ye va a tocar editar unos cuantos artículos (en mi caso tenía más de 150, así que será un proceso lento).

## Hosting

Esto es muy interesante, ya que con Jekyll y github pages, puedo tener mi propia página con la siguiente url http://nombre_de_usuario.github.io, y además permiten usar un dominio propio.

## Redireccionando

Esto es aún algo en lo que tengo que trabajar un poco
