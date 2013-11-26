---
layout: post
title: "Manipulating the Windows registry from .NET applications"
date: 2013-11-26 11:00
comments: true
categories: []
published: true
---

The windows registry that has given us many headaches is nothing more than a way to store information in a tree. Is used to store preferences by applications and also by the operating system. In this article we will see how to access, store and delete keys from our .NET application.

# Introduction

The first thing we have to bear in mind is that Windows Registry is somewhat "delicate", because that is shared by all applications and the system (in fact, Windows Store applications for Windows 8 have no access to it), so when we manipulate it, the best advice is to limit ourselves to the folder that stores the information of our application.

There are multiple routes, the two best known are:

* **HKEY\_LOCAL\_MACHINE** (or HKLM), which contains the system-wide information.
* **HKEY\_CURRENT\_USER** (or HKCU), which contains information about the current user's session.

To use the registry in our applications, we need the following _using_ clause:

{% highlight csharp %}
using Microsoft.Win32;
{% endhighlight %}

# Accessing a key and retrieving values

To access the information contained in a key, first we access access the CurrentUser field, equivalent to HKCU defined above, then to the "Software" folder and finally our application folder.

{% highlight csharp %}
string name = "Foo";
RegistryKey key = Registry.CurrentUser.OpenSubKey(@"Software\MyApp\", true);
object selectedValue = key.GetValue(name);
{% endhighlight %}

The second parameter adds writing permissions, otherwise we would receive the key in read-only mode. Any writing operation in read-only would cause an exception. If the requested key is not found, the result will be null. Once the key is retrieved, we can get any value by it's name.

# Creating a key and inserting values

To create a key, the process is similar to the previous one, however, in this case the permissions include the posibility to write by default:

{% highlight csharp %}
string name = "Foo";
string value = "Bar";
key = Registry.CurrentUser.CreateSubKey(@"Software\MyApp\");
key.SetValue(name, value);
{% endhighlight %}

Once created, the values can be easily inserted.

# Deleting values on keys

Finally, to eliminate a key value we need to access them in the same way that we've done before:

{% highlight csharp %}
RegistryKey key = Registry.CurrentUser.OpenSubKey(@"Software\MyApp\", true);
key.DeleteValue(name);
{% endhighlight %}

Deletion of values is performed similarly to the query before, returning an exception if the value to delete is not present in the key.

# Overview

In this brief article we have seen how to create subkeys in registry, how to access them and their values, and finally, how to clear the values of the keys. The article from the MSDN documentation also contains overloads of the methods that we have seen here.

Additional links

* [Registry Hive](http://pcsupport.about.com/od/termsr/g/registryhive.htm)
* On MSDN [Registry Key](http://msdn.microsoft.com/en-us/library/microsoft.win32.registrykey.aspx)