Views
#####

Views are the **V** in MVC. Views are responsible for generating
the specific output required for the request. Often this is in the
form of HTML, XML, or JSON, but streaming files and creating PDF's
that users can download are also responsibilities of the View
Layer.

View Templates
==============

The view layer of CakePHP is how you speak to your users. Most of
the time your views will be showing (X)HTML documents to browsers,
but you might also need to serve AMF data to a Flash object, reply
to a remote application via SOAP, or output a CSV file for a user.

CakePHP view files are written in plain PHP and have a default
extension of .ctp (CakePHP Template). These files contain all the
presentational logic needed to get the data it received from the
controller in a format that is ready for the audience you’re
serving to.

View files are stored in /app/views/, in a folder named after the
controller that uses the files, and named after the action it
corresponds to. For example, the view file for the Products
controller's "view()" action, would normally be found in
/app/views/products/view.ctp.

The view layer in CakePHP can be made up of a number of different
parts. Each part has different uses, and will be covered in this
chapter:


-  **layouts**: view files that contain presentational code that is
   found wrapping many interfaces in your application. Most views are
   rendered inside of a layout.
-  **elements**: smaller, reusable bits of view code. Elements are
   usually rendered inside of views.
-  **helpers**: these classes encapsulate view logic that is needed
   in many places in the view layer. Among other things, helpers in
   CakePHP can help you build forms, build AJAX functionality,
   paginate model data, or serve RSS feeds.

.. _view-layouts:

Layouts
=======

A layout contains presentation code that wraps around a view.
Anything you want to see in all of your views should be placed in a
layout.

Layout files should be placed in /app/views/layouts. CakePHP's
default layout can be overridden by creating a new default layout
at /app/views/layouts/default.ctp. Once a new default layout has
been created, controller-rendered view code is placed inside of the
default layout when the page is rendered.

When you create a layout, you need to tell CakePHP where to place
the code for your views. To do so, make sure your layout includes a
place for $content\_for\_layout (and optionally,
$title\_for\_layout). Here's an example of what a default layout
might look like:

::

   <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
   "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
   <html xmlns="http://www.w3.org/1999/xhtml">
   <head>
   <title><?php echo $title_for_layout?></title>
   <link rel="shortcut icon" href="favicon.ico" type="image/x-icon">
   <!-- Include external files and scripts here (See HTML helper for more info.) -->
   <?php echo $scripts_for_layout ?>
   </head>
   <body>

   <!-- If you'd like some sort of menu to 
   show up on all of your views, include it here -->
   <div id="header">
       <div id="menu">...</div>
   </div>

   <!-- Here's where I want my views to be displayed -->
   <?php echo $content_for_layout ?>

   <!-- Add a footer to each displayed page -->
   <div id="footer">...</div>

   </body>
   </html>

``$scripts_for_layout`` contains any external files and scripts
included with the built-in HTML helper. Useful for including
javascript and CSS files from views.

.. note::

    When using ``$html->css()`` or ``$javascript->link()`` in view
    files, specify 'false' for the 'in-line' argument to place the html
    source in ``$scripts_for_layout``. (See API for more details on
    usage).

``$content_for_layout`` contains the view. This is where the view
code will be placed.

``$title_for_layout`` contains the page title.

To set the title for the layout, it's easiest to do so in the
controller, setting the $title\_for\_layout variable.

::

   <?php

   class UsersController extends AppController {
       function viewActive() {
           $this->set('title_for_layout', 'View Active Users');
       }
   }
   ?>

You can create as many layouts as you wish: just place them in the
app/views/layouts directory, and switch between them inside of your
controller actions using the controller's $layout variable, or
setLayout() function.

For example, if a section of my site included a smaller ad banner
space, I might create a new layout with the smaller advertising
space and specify it as the layout for all controller's actions
using something like:

``var $layout = 'default_small_ad';``

::

   <?php
   class UsersController extends AppController {
       function viewActive() {
           $this->set('title_for_layout', 'View Active Users');
           $this->layout = 'default_small_ad';
       }

       function viewImage() {
           $this->layout = 'image';
           //output user image
       }
   }
   ?>

CakePHP features two core layouts (besides CakePHP’s default
layout) you can use in your own application: ‘ajax’ and ‘flash’.
The Ajax layout is handy for crafting Ajax responses - it’s an
empty layout (most ajax calls only require a bit of markup in
return, rather than a fully-rendered interface). The flash layout
is used for messages shown by the controllers flash() method.

Three other layouts xml, js, and rss exist in the core for a quick
and easy way to serve up content that isn’t text/html.

.. _view-elements:

Elements
========

Many applications have small blocks of presentation code that need
to be repeated from page to page, sometimes in different places in
the layout. CakePHP can help you repeat parts of your website that
need to be reused. These reusable parts are called Elements. Ads,
help boxes, navigational controls, extra menus, login forms, and
callouts are often implemented in CakePHP as elements. An element
is basically a mini-view that can be included in other views, in
layouts, and even within other elements. Elements can be used to
make a view more readable, placing the rendering of repeating
elements in its own file. They can also help you re-use content
fragments in your application.

Elements live in the /app/views/elements/ folder, and have the .ctp
filename extension. They are output using the element method of the
view.

::

    <?php echo $this->element('helpbox'); ?>

Passing Variables into an Element
---------------------------------

You can pass data to an element through the element's second
argument:

::

    <?php echo
    $this->element('helpbox', 
        array("helptext" => "Oh, this text is very helpful."));
    ?>

Inside the element file, all the passed variables are available as
members of the parameter array (in the same way that ``set()`` in
the controller works with view files). In the above example, the
/app/views/elements/helpbox.ctp file can use the ``$helptext``
variable.

::

    <?php
    echo $helptext; //outputs "Oh, this text is very helpful."
    ?>

The ``element()`` function combines options for the element with
the data for the element to pass. The two options are 'cache' and
'plugin'. An example:

::

    <?php echo
    $this->element('helpbox', 
        array(
            "helptext" => "This is passed to the element as $helptext",
            "foobar" => "This is passed to the element as $foobar",
            "cache" => "+2 days", //sets the caching to +2 days.
            "plugin" => "" //to render an element from a plugin
        )
    );
    ?>

To cache different versions of the same element in an application,
provide a unique cache key value using the following format:

::

    <?php
    $this->element('helpbox',
        array(
            "cache" => array('time'=> "+7 days",'key'=>'unique value')
        )
    );
    ?>

You can take full advantage of elements by using
``requestAction()``. The ``requestAction()`` function fetches view
variables from a controller action and returns them as an array.
This enables your elements to perform in true MVC style. Create a
controller action that prepares the view variables for your
elements, then call ``requestAction()`` inside the second parameter
of ``element()`` to feed the element the view variables from your
controller.

To do this, in your controller add something like the following for
the Post example.

::

    <?php
    class PostsController extends AppController {
        ...
        function index() {
            $posts = $this->paginate();
            if (isset($this->params['requested'])) {
                return $posts;
            } else {
                $this->set('posts', $posts);
            }
        }
    }
    ?>

And then in the element we can access the paginated posts model. To
get the latest five posts in an ordered list we would do something
like the following:

::

    <h2>Latest Posts</h2>
    <?php $posts = $this->requestAction('posts/index/sort:created/direction:asc/limit:5'); ?>
    <?php foreach($posts as $post): ?>
    <ol>
        <li><?php echo $post['Post']['title']; ?></li>
    </ol>
    <?php endforeach; ?>

Caching Elements
----------------

You can take advantage of CakePHP view caching if you supply a
cache parameter. If set to true, it will cache for 1 day.
Otherwise, you can set alternative expiration times. See
:doc:`/core-libraries/caching` for more information on setting
expiration.

::

    <?php echo $this->element('helpbox', array('cache' => true)); ?>

If you render the same element more than once in a view and have
caching enabled be sure to set the 'key' parameter to a different
name each time. This will prevent each succesive call from
overwriting the previous element() call's cached result. E.g.

::

    <?php
    echo $this->element('helpbox', array('cache' => array('key' => 'first_use', 'time' => '+1 day'), 'var' => $var));
    
    echo $this->element('helpbox', array('cache' => array('key' => 'second_use', 'time' => '+1 day'), 'var' => $differentVar));
    ?>

The above will ensure that both element results are cached
separately.

Requesting Elements from a Plugin
---------------------------------

If you are using a plugin and wish to use elements from within the
plugin, just specify the plugin parameter. If the view is being
rendered for a plugin controller/action, it will automatically
point to the element for the plugin. If the element doesn't exist
in the plugin, it will look in the main APP folder.

::

    <?php echo $this->element('helpbox', array('plugin' => 'pluginname')); ?>

View methods
============

.. php:class:: View

View methods are accessible in all view, element and layout files.
To call any view method use ``$this->method()``

.. php:method:: set(string $var, mixed $value)

    Views have a ``set()`` method that is analogous to the ``set()``
    found in Controller objects. It allows you to add variables to the
    . Using set() from your view file will add the variables to the
    layout and elements that will be rendered later. See
    :ref:`controller-methods` for more
    information on using set().

    In your view file you can do::

        <?php
        $this->set('activeMenuButton', 'posts');

    Then in your layout the ``$activeMenuButton`` variable will be
    available and contain the value 'posts'.

.. php:method:: getVar(string $var)

    Gets the value of the viewVar with the name $var

.. php:method:: getVars()

    Gets a list of all the available view variables in the current
    rendering scope. Returns an array of variable names.

.. php:method:: element(string $elementPath, array $data, bool $loadHelpers)

    Renders an element or view partial. See the section on
    :ref:`view-elements` for more information and
    examples.

.. php:method:: uuid(string $object, mixed $url)

    Generates a unique non-random DOM ID for an object, based on the
    object type and url. This method is often used by helpers that need
    to generate unique DOM ID's for elements such as the :php:class:`JsHelper`::

        <?php
        $uuid = $this->uuid('form', array('controller' => 'posts', 'action' => 'index'));
        //$uuid contains 'form0425fe3bad'

.. php:method:: addScript(string $name, string $content)

    Adds content to the internal scripts buffer. This buffer is made
    available in the layout as ``$scripts_for_layout``. This method is
    helpful when creating helpers that need to add javascript or css
    directly to the layout. Keep in mind that scripts added from the
    layout, or elements in the layout will not be added to
    ``$scripts_for_layout``. This method is most often used from inside
    helpers, like the :doc:`/core-libraries/core-helpers/js` and
    :doc:`/core-libraries/core-helpers/html` Helpers.

More about Views
================

.. toctree::

    views/themes
    views/media-view
    views/helpers