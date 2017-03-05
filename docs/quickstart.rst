************************************************************************* 
Quickstart
*************************************************************************

=========================================================================
Displaying a ProgressBar
=========================================================================

Using a ProgressBar in your code is very easy. Let us see some ways by
which we can show a functioning progress bar during a task.

-------------------------------------------------------------------------
Usage In a Loop
-------------------------------------------------------------------------

You can use an AdvancedProgressBar instance named `bar` to display a
ProgressBar. Here we are publishing the percentage of the task completed
to `bar`. Note that we need to call `begin` and `end` at appropriate time.

.. code:: python
   
   from progress_manager import AdvancedProgressBar
   bar = AdvancedProgressBar()
   bar.begin()
   for i in range(1000):
       bar.publish((i+1)/10)
   bar.end()

-------------------------------------------------------------------------
Wrapping a Generator
-------------------------------------------------------------------------

Let us define a generator that just yields numbers from 0 to n.

.. code:: python

   def generator(n):
       for x in range(n):
           yield x

You can wrap this generator with a ProgressBar instance and you should
get a nice progress indicator. Note that you should use a Non-deterministic
progress bar since there is no way to automatically calculate the progress
of a generator since it can even be infinite. 

.. code:: python

   for i in bar(generator(n)):
       time.sleep(0.01)
       
Also Note that, there is no need to call `begin`, `publish`, or `end`.
   
-------------------------------------------------------------------------
Wrapping a Iterator
-------------------------------------------------------------------------

Iterators can also be wrapped with a ProgressBar instance. Similar to a 
generator there is no need to call `begin`, `publish`, or `end`. If the
iterator provides a __len__ implementation, The progress is automatically
calculated, like in the following case.

.. code:: python

   for i in bar(range(n)):
       time.sleep(0.01)
   
-------------------------------------------------------------------------
Using with statement
-------------------------------------------------------------------------

To avoid calling `begin` and `end`, you can wrap your code using a with
statement that ensures that `begin` and `end` are automatically called
at appropriate times.

.. code:: python

   with AdvancedProgressBar() as bar:
       for i in range(n):
           time.sleep(0.01)
           bar.publish(100*(i+1)/n)
           
           
=========================================================================
Creating a Custom ProgressBar
=========================================================================

The default PrgressBar is enough for most purposes. But if you need to
customize the look of the ProgressBar, you can do so via `Extensions`.

-------------------------------------------------------------------------
Building The ProgressBar
-------------------------------------------------------------------------

ProgressManager takes a parameter named components which is an iterable
of string or extensions. Using this, you can customize the look of your
progress bar as per your wish. 

.. code:: python

   from progress_manager.core import ProgressManager
   from progress_manager.extensions import Percentage, Bar
   bar = ProgressManager(components=[Bar(), "Progress =", Percentage()])

-------------------------------------------------------------------------
Built-in Extensions
-------------------------------------------------------------------------

A large number of extensions are provided by default. More details on them
can be found in the API Reference.

* :class:`~progress_manager.extensions.Bar`
* :class:`~progress_manager.extensions.BouncingBar`
* :class:`~progress_manager.extensions.Ellipses`
* :class:`~progress_manager.extensions.Alternator`
* :class:`~progress_manager.extensions.Spinner`
* :class:`~progress_manager.extensions.Loader`
* :class:`~progress_manager.extensions.Timer`
* :class:`~progress_manager.extensions.ETA`
* :class:`~progress_manager.extensions.Rate`
* :class:`~progress_manager.extensions.Percentage`

-------------------------------------------------------------------------
Writing your own Extensions
-------------------------------------------------------------------------

Although the extensions provided by default should be enough, but you can
always create your own extensions by subclassing
`progress_manager.base.BaseExtension`. More detail on this can be found
in the API Reference.

Every extension should call the `__init__` method of the BaseExtension
class by passing a list of strings as `requirements`. The strings which
can be passed are known as tags. Following is the list of all supported
tags.

* value
* max_value
* min_value
* begin_time
* end_time
* iterations
* percentage
* time_since_begin
* time_since_update
* deltatime
* eta
* rate

You can then override several event methods of
:class:`~progress_manager.base.BaseExtension`, such as `on_begin`,
`on_update`, `on_validated`, `on_invalidated`, `on_end` to suit your
needs. In each of these methods you recieve a list of values
corresponding to the requirements you passed in the `__init__` method.
Note that `on_validated` and `on_invalidated` are called by the default
implementation of `on_update`. If you override `on_update`, those methods
will no longer be called unless you call them explicitly.
In general, you should use `on_validated` and `on_invalidated` for
most of the purposes. 
To set the string that is to be displayed by your extension, just call
the `set_value` method from your extension

-------------------------------------------------------------------------
Writing your own Providers
-------------------------------------------------------------------------

But what if you want need some parameter which is not provided by the
built-in tags? You can also create Custom Providers to calculate values
and specify a new tag for them that can be used by other extensions.

Creating a provider is similar to creating an extension. But note that,
instead of `on_update`, here we can override `on_publish`. Also the
__init__ method does not take update_interval as a parameter, Instead
it takes a parameter `tag` which takes a string. Rest of api is same.
The tag should not collide with any built-in tag.
Prior to using a provider you need to register it. To register,
just call the `register_provider` method of the ProgressManager class
and pass it an instance of your provider.