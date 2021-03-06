---
title: Templating 
layout: manpage 
author: Jan Decaluwe
---

Overview
========

With templates you specify the html layout for a particular type of a page.  In
a template you can mix plain html with control structures and variable
interpolation. The actual html page is generated by evaluating the template
with the appropriate evaluation context, provided by Urubu.

Templates are used to set up a project properly. If you are a content
contributor, you may not have to worry about them.  Once a project is set up,
all you have to do is specify the appropriate layout in the YAML front matter
of your content files.

In general, it is better to keep logic in Python code.  However, for the
purpose of generation of pages in a format such as html, this would not be very
practical.  A template is therefore a compromise, in which some of the logic is
put into the template itself. Template logic is good for the following common
tasks:

* iteration over a list of items
* testing whether an item is defined
* filtering items, possibly with user-defined filters 
* comparing the loop item object with the current object, to see whether it is active
and should be highlighted

Urubu interacts with templates by providing an evaluation context with the
appropriate objects. These have a number of attributes that the templates can
use. The goal is to make the job of the templates as easy as possible with
ready-to-use attributes. 

Urubu supports Python hooks that make the task of the template easier.
You can define validators that validate and possibly change an
item before passing it to the template. Also, you can define custom filters
that for use by the templates.

The template library
====================

Urubu uses the [jinja2] templating language library.

Jinja2 has great [documentation][jinja2_docs] and you should
check it out when using templates in Urubu.

[jinja2_docs]: http://jinja2.pocoo.org/docs

A great feature of Jinja2 is template inheritance. With this technique,
you can easily generate small variations of a parent template.

Evaluation context
==================

Link objects
------------

### Description

Link objects are the primary objects that you use in templates. 
They come in a number of flavours: 

Link object        | Description
-------------------|---------------
global link object | Defined in the `_site.yml` file.
local link object  | Defined in the `content` attribute of an index page.
folder object      | Corresponds to a project directory.
page objects       | Correspond to a Markdown content file.

Global link, folder and page objects are identified by a
unique reference id, as described in the [authoring].
Local link objects don't have an id and are only
available in the `content` of a particular index page.

### Common link object attributes

Attribute      | Description 
---------------|---------------------------
`id`           | The unique id by which the object is known in the project. Not defined for local link objects.
`url`          | The url of the object. 
`title`        | The title of the object.

### Common page and folder attributes 

Attribute      | Description 
---------------|---------------------------
`fn`           | The pathname of the file or directory corresponding the object. 
`components`   | The components of the object's pathname, without file extension, as a list.

### Folder attributes

Attribute      | Description 
---------------|---------------------------
`content`      | The content of the folder as a list of page & folder objects.

In addition, all attributes specified in the YAML front matter
of the corresponding index file will be available as attributes of
the folder object.

### Page attributes

Attribute      | Description 
---------------|---------------------------
`body`         | The page content in html.
`toc`          | The table of contents of the page as an unordered html list.
`breadcrumbs`  | Breadcrumbs as a list. The current page object is at position 0, the containing folder objects are at the higher positions.
`prev`         | The previous page object in the content, or `None` if there is none
`next`         | The next page object in the content, or `None` if there is none

In addition, all attributes specified in the YAML front matter
of the corresponding file are available as attributes of
the page object.

### Index pages

Index pages are associated with `index.md` files. They are special in the sense
that they define the attributes and the content of a folder. Therefore, they
have the same `content` attribute as the corresponding folder object.

Context variables
-----------------

Urubu makes the context available to templates with
two context variables.

### `site` 

This variable holds site-wide information.

It has one predefined attribute:

Attribute      | Description 
---------------|---------------------------
`reflinks`     | A mapping from all reference ids to link objects.

Note that the id of the root object is `/`. Starting from there,
you can traverse the whole site.

In addition, all the attributes specified in the `_site.yml` file
will be available as attributes of the `site` variable.

### `this`

This variable holds the current page object.

Python hooks
============

Urubu support Python hooks to make templating easier. Upon
a build, it tries to import a `_python` module or package,
and looks for hook variables with predefined names. 

Variable              | Description
----------------------|-------------
`validators`          | A mapping from layout names to page object validation functions.
`filters`             | A mapping from filter names to filter functions.

A validator function should be defined with a page object as its argument. It
can inspect the attributes, verify and possibly modify them, and even add
additional ones. The appropriate validator function is looked up via the
`layout` attribute of a page. If it exists, it is run on the page object before
calling the template. 

Filters functions should be defined as [custom filters in
Jinja2][jinja2_filters].

[jinja2_filters]: http://jinja.pocoo.org/docs/api/#custom-filters



