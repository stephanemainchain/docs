Optimize the Footprint of an Application
========================================

This guide explains how a developer can analyze the footprint of an application and how he can reduce both the ROM and the RAM footprint.

Temporary Section
-----------------

To be done now:

- SOAR.map & SOAR.xml & application.map guides
- Add RAM guideline: avoid new BufferedImage => images heap usage
- Add how to configure images heap size

To be added later:

- Reference to tutorial on stripping pixel conversion algorithms (not published yet)
- LinkerMapFileInterpreter tool and BSP MMS file (not released yet)
- Partial buffer example (not developed yet)

How to Analyze the Footprint of an Application
----------------------------------------------

This section explains the process of analyzing the footprint of a MicroEJ application and the tools which can be used during the analysis.

Suggested footprint analysis process:

1. Run MicroEJ launcher on embedded platform
2. Analyze ``SOAR.map`` with the :ref`memorymapanalyzer`
3. Analyze ``SOAR.xml`` with an XML editor
4. Link the MicroEJ application with the BSP
5. Analyze ``application.map`` with a text editor

Footprint analysis tools:

- The :ref`memorymapanalyzer` allows to analyze the memory consumption of different features in the RAM and ROM.
- The :ref`heapdumper` allow to understand the contents of the Java heap and find problems such as memory leaks.
- The `Dependency discoverer <https://forum.microej.com/t/tool-dependency-discoverer/771>`_  allows to the analyze a piece of code to detect all its dependencies.

How to analyze a SOAR.map file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

TODO

Explain what is SOAR.map and how to analyze the data/categories

How to analyze a SOAR.xml file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

TODO

Explain what is SOAR.xml and which data can be extracted (strings, properties, required types, etc)

How to analyze an application.map file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

TODO

For example search for ``.bss.vm.stacks.java`` or ``ICETEA_HEAP``.
The third column is the size of the symbol.

How to Reduce the Image Size of an Application
----------------------------------------------

Generic coding rules may be found in the following tutorial: :ref:`javacodingrules`.

This section provides additional coding rules and good practices in order to reduce the image size (flash memory) of an application.

Application Resources
~~~~~~~~~~~~~~~~~~~~~

Resources such as images and fonts take a lot of memory.
For every ``.list`` file, make sure that it does not embed any unused resource. Having unused resources in the Java classpath is OK as long as they are not listed in a ``.list`` file.

Fonts
^^^^^

Removing the default font from the platform configuration
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""

By default, in the platform configuration project, a so-called system font is declared inside the microui.xml file.

When generating the platform, this file is copied from the configuration project to the actual platform project and will later on be converted to binary format and linked with your Java application, even if you use fonts that are different from the system font.

You can therefore comment the system font from the microui.xml file to reduce the flash footprint of your Java application if it does not rely on the system font. Note that you will need to rebuild the platform and then the application to benefit from the footprint reduction.

See `Display Element` section of the :ref:`section_static_init` documentation for more info on system fonts.

Character ranges
""""""""""""""""

When creating a font, reduce the list of characters embedded in the font either:

- on font creation. See `Removing Unused Characters` section of :ref:`section.tool.fontdesigner` documentation for more info on how to achieve that
- on application build. See `Fonts` section of :ref:`chapter.microej.classpath` documentation for more info on how to achieve that

Pixel Transparency
""""""""""""""""""

You can also make sure that the BPP encoding used to achieve transparency for your fonts do not exceed:

- The maximum alpha level of your display device
- The required alpha level for a good rendering of your font in the application

See `Fonts` section of :ref:`chapter.microej.classpath` documentation for more info on how to achieve that.

External Storage
""""""""""""""""

To save storage on FLASH, fonts may be accessed from external storage.

See `External Resources` section of the :ref:`section_fontgen` documentation for more info on how to achieve that.

Textual elements
""""""""""""""""

MicroEJ provides the Native Language Support (NLS for short) library to handle internationalization.

See https://github.com/MicroEJ/Example-NLS as an example of use of the NLS library.

You can of course use your own internationalization library if you want. Whatever internationalization library you use, the tips below may be relevant to the footprint optimization domain.

External Storage
""""""""""""""""

The default NLS implementation fetches text resources from internal flash, but you can replace it with your own implementation so as to fetch them from another location.

See :ref:`section_externalresourceloader` documentation for additional info on external resources management.

Compression
"""""""""""

The default NLS implementation relies on text resources that are not compressed, but you can replace it with your own so as to load them from compressed resources.

Images
^^^^^^

Encoding
""""""""

If you are tight on FLASH memory but have enough RAM and CPU power to decode PNG images on the fly, consider storing your images as PNG resources.
If you are in the opposite configuration (lots of FLASH, but little RAM and CPU power), consider storing your images in raw format.

See :ref:`section_image_generator` documentation for more info on how to achieve that.

Bits Per Pixel (BPP)
""""""""""""""""""""

Make sure to use images with a color depth not exceeding the one of your display device so as to avoid:

- wasting memory
- rendering differences between the target device and the original image resource

External Storage
""""""""""""""""

To save storage on FLASH, images may be accessed from external storage.

See :ref:`section_externalresourceloader` documentation for more info on how to achieve that.

Application Code
~~~~~~~~~~~~~~~~

The following application code guidelines are recommended in order to minimize the size of the application:

- Avoid using MicroUI 2 and MWT 2, use MicroUI 3 and MWT 3 instead. Many optimizations have been done in the new versions.
- Avoid manipulating ``String`` objects when possible. For example, prefer using integers to represent IDs. Indeed, strings take a lot of memory.
- Avoid using logging library or ``println()``, use the trace library (``ej.api#trace``) instead. The logging library uses strings, while the trace library is light and uses error codes.
- Avoid manipulating ``Integer`` or ``Long`` objects for example, manipulate primitive types instead. Objects take more memory and require boxing/unboxing operations.
- Avoid using service library, use singletons instead. The service library adds extra code which doesn't add any feature to your application. It also embeds reflection methods of EDC.
- Avoid using ``List`` objects, use arrays and ``ArrayTools`` instead. Even though the collections framework is very user-friendly, the code size and the heap usage are much important than when manipulating arrays.
- Avoid using ``Map`` objects, use ``PackedMap`` instead. Packed maps provide the same features than collection maps but are much lighter.
- Avoid using ``StringBuffer``, use ``StringBuilder`` instead. They do the same thing, except that ``StringBuffer`` is synchronized and thus it is heavier.
- Avoid using ``java.util.Timer``, use ``ej.bon.Timer`` instead. EDC's timers are now deprecated.
- Avoid serializing/deserializing data from byte arrays using manual bitwise operations, use ``ByteArray`` instead.
- Use BON constants when executing debug code or optional code, such as ``if (Constants.getBoolean()) { ... }``. That way the optional code will not be embedded if the constant is ``false``.
- Avoid using system properties, use BON constants instead. Constants checks are computed at compile time rather than at run time. Also, manipulating properties requires to embed their name, and strings take a lot of memory.
- Avoid using synchronization. A ``synchronized`` block takes a lot of extra code size, even though it is only a few characters of code.
- Avoid calling ``equals()`` and ``hashCode()`` on ``Object`` references. If you do, the method will be embedded for every class which overrides the method.
- Avoid using the string concatenation operator (``+``) when concatenating more than 2 objects into a single string, use ``StringBuilder`` instead. Multiple ``+`` take more code size than multiple ``StringBuilder.append()`` calls.
- Avoid using ``java.util.Calendar``, use an other calendar implementation instead. The calendar implementation of EDC is very heavy even when only a few methods are used.
- Avoid creating anonymous objects (such as ``Runnable`` objects), re-use other classes instead. Indeed, these objects are treated like as whole new class, and each enclosed final variable is treated as a field of the class.
- Avoid accessing the same field multiple times in the same method, copy the value of the field into a local variable instead. Accessing fields leads o bigger code size and may induce synchronization issues.

Platform Configuration
~~~~~~~~~~~~~~~~~~~~~~

The following platform configuration guidelines are recommended in order to minimize the size of the application:

- Use the latest MicroEJ architecture.
- Use tiny MEJ32 architecture. It reduces the code size by ~20% but it is only possible if the size of the application code is lower than 256KB (resources excluded). See dedicated documentation: :ref:`core-tiny`
- Don't embed unnecessary pixel conversion algorithms. This can save up to ~8KB of code size but it requires knowing the format of the resources embedded in the application.
- Use the best optimization level for every source file (for example ``-O3`` or ``-Os`` on GCC).
- Use an optimal compiler such as IAR rather than GCC.
- Get the linker command line and check that every parameter is OK. The linker command line can be found in the project settings and it may be printed during link. For example, if there is ``-u _printf_float`` in the parameters, you can go in the project settings and disable printf float.
- In the ``application.map``, check that debug methods are not embedded in production, such as SystemView.
- In the ``application.map``, check that every embedded method is necessary. For example, timers or HAL components may be initialized by default in the BSP but they may not be useful in your application.

Application Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

The following application configuration guidelines are recommended in order to minimize the size of the application:

- Don't embed class names by setting the ``soar.generate.classnames`` property to ``false``. Class names are only useful for logging and for reflection, in which case the name of a specific class can be embedded by adding the class to the ``*.types.list`` file. Refer to :ref`stripclassnames` for a dedicated tutorial.
- Don't embed UTF-8 encoding by setting the ``cldc.encoding.utf8.included`` property to ``false``. The default encoding (``ISO-8859-1``) is enough for most applications.
- Don't embed ``SecurityManager`` checks by setting the ``com.microej.library.edc.securitymanager.enabled`` property to ``false``. This feature is only useful for multi-app firmwares.
- Don't embed ``toString()`` methods by setting the ``com.microej.library.edc.tostring.included`` property to ``false``.

How to Reduce the Runtime Size of an Application
------------------------------------------------

Generic coding rules may be found in the following tutorial: :ref:`javacodingrules`.

This section provides additional coding rules and good practices in order to reduce the runtime size (RAM) of an application.

Application Code
~~~~~~~~~~~~~~~~

The following application code guidelines are recommended in order to minimize the size of the application:

- Define fields as ``short`` or ``byte`` rather than ``int``.
- Don't declare arrays in ``static final`` fields, use immutables instead.
- Make sure your widget hierarchy is as flat as possible (avoid unnecessary containers).
- Make sure that the size of the buffers can be configured (by a parameter in the constructor or by a BON constant for example).
- Avoid using immortal arrays to call native methods, use regular arrays instead.
- Avoid creating multiple threads, timers, or executors, share the instances instead when possible. Each thread requires to allocate dedicated VM stacks which take a lot of memory.

Platform Configuration
~~~~~~~~~~~~~~~~~~~~~~

The following platform configuration guidelines are recommended in order to minimize the size of the application:

- Check the size of the stack of each RTOS task. For example, 1.0KB may be enough for the MicroJVM task but it can be increased to allow deep native calls.
- Check the size of the heap allocated by the RTOS (for example ``configTOTAL_HEAP_SIZE`` for FreeRTOS).
- Check that the size of the frame buffer matches the size of the screen. Use a partial buffer if the frame buffer does not fit in the RAM.

Debugging Stack Overflows
^^^^^^^^^^^^^^^^^^^^^^^^^

If the size that you allocate for a given RTOS task is too low, a stack overflow will occur. To be aware of stack overflows, proceed with the following steps (for FreeRTOS):

1. Enable the stack overflow check in ``FreeRTOS.h``:

.. code-block:: c

	#define configCHECK_FOR_STACK_OVERFLOW 1

2. Define the hook function in any file of your project (`main.c` for example):

.. code-block:: c

	void vApplicationStackOverflowHook(TaskHandle_t xTask, signed char *pcTaskName) { }

3. Add a new breakpoint inside this function
4. When a stack overflow occurs, the execution will stop at this breakpoint

Application Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

The following application configuration guidelines are recommended in order to minimize the size of the application.

Java Heap and Immortals Heap
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Configure the immortals heap to be as small as possible. The minimum value can be known by calling ``Immortals.freeMemory()`` after all immortals object have been created.
- Configure the Java heap to fit the needs of the application. The maximum heap usage can be known by calling ``Runtime.freeMemory()`` after ``System.gc()`` at different moments in the lifecycle of the application.

Thread Stacks
^^^^^^^^^^^^^

- Configure the maximum number of threads. This number can be known accurately by counting in the code how many ``Thread`` and ``Timer`` objects may run concurrently. Calling ``Thread.getAllStackTraces()`` can help in knowing which threads are running at a given moment.
- Configure the number of allocated thread stack blocks. Keep the default value for the size of a block (``512``) and figure out how many blocks each thread requires. This can be done empirically by starting with a low number of blocks and increasing this number as long as the application throws a ``StackOverflowError``.
- Configure the maximum number of blocks per thread. The best choice is to set it to the number of blocks required by the most greedy thread. An other acceptable choice is to set it to the same value than the total number of allocated blocks.
- Configure the maximum number of monitors. This number can be known accurately by counting the number of concurrent ``synchronized`` blocks. This can also be done empirically by starting with a low number of monitors and increasing this number as long as no exception occurs. Either way, it is recommended to set a slightly higher value than calculated.
