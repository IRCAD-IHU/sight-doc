Error handling and logging
=============================

A correct handling of errors is very important is any software. If you never read it, it is strongly advised to read
about `error handling <https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines.html#S-errors>`_. *Sight* proposes
several mechanisms to handle them properly.

But first of all, different types of errors should be distinguished:

- **assertions**: check pre/post conditions inside section of codes
- **programming errors**: inform the developer about coding errors
- **user errors**: inform the user about an unhandled situation

In addition, errors can have different levels of criticity. Some can be safely ignored, while others may prevent the
application from running properly.

Apart from errors, developers sometimes want to track important steps of the application for debugging purposes, or to
help users reporting bugs.

In order to collect all this information, *Sight* proposes a full logging system called `SpyLog`. On Linux, the log is
redirected to the standard output and on Windows, it is redirected by default on a file ``SLM.log`` located in the
current working directory. However, it is possible to change this behavior when calling ``fwlauncher`` or any
application shortcut :

.. code-block:: sh

    # Output in a file named "foo.log"
    bin/fwlauncher --flog foo.log
    # Force output in the terminal
    bin/fwlauncher --clog
    # Output in a file named "foo.log", disable output in the terminal
    bin/fwlauncher --flog foo.log --no-clog
    # Disable output in the SLM.log file
    bin/fwlauncher --no-flog

All macros presented below and prefixed by ``SLM`` use this logging system.


Assertions
-------------

The C language provides the function ``assert()`` which can be used to check pre/post conditions. In *Sight*, we propose
an extended version called ``SLM_ASSERT(const std::stringstream&, CONDITION)`` which allows to specify from a formatted
message string and to benefit from our logging system.

Programming errors
-------------------

These kind of errors is meant to help debugging or to help users reporting bugs. *Sight* proposes three ways to report
them:

- ``SLM_WARN(const std::stringstream&)``: when something potentially harmful happens.
- ``SLM_ERROR(const std::stringstream&)``: when something unexpected occurs.
- ``SLM_FATAL(const std::stringstream&)``: when an unrecoverable errors occurs. This exits the program or triggers a
  breakpoint if a debugger is running.

In addition with these three macros, *Sight* provides alternatives allowing to specify a condition to be tested:

- ``SLM_WARN_IF(const std::stringstream&, CONDITION)``
- ``SLM_ERROR_IF(const std::stringstream&, CONDITION)``
- ``SLM_FATAL_IF(const std::stringstream&, CONDITION)``

Keep in mind reporting errors this way is for developers. To handle a real error and inform the user, please use real
user errors, usually using GUI features as shown in the section below.

Trace code
-----------

For regular logging, *Sight* proposes two levels:

- ``SLM_DEBUG(const std::stringstream&)``: to debug the code, display internal states, variables, etc... **It is not**
  **recommended to let such macros in production code.**
- ``SLM_INFO(const std::stringstream&)``:  to keep track of important events in the code. Only meaningful information
  for all developers should be kept.

As before, *Sight* provides alternative signatures with a condition:

- ``SLM_DEBUG_IF(const std::stringstream&, CONDITION)``
- ``SLM_INFO_IF(const std::stringstream&, CONDITION)``

Please note that by default, ``fwlauncher`` does not display these logging levels. To enable them, please read the
next section.

Filtering
------------

A complete application may generate a big log. To avoid this, developers should first try to logging to the minimum,
following two rules:

- do not let ``SLM_DEBUG()`` messages in production code
- do not let **any kind** of logging in a tight loop, in production code

Even following these two rules, the log may remain large. It is possible to reduce the level of logging on the command
line, using options of ``fwlauncher``:

.. code-block:: sh

    Log options:
    --log-debug                 Set loglevel to debug
    --log-info                  Set loglevel to info
    --log-warn                  Set loglevel to warn (default)
    --log-error                 Set loglevel to error
    --log-fatal                 Set loglevel to fatal

Last, when looking for a specific output, it is strongly recommended to use a log explorer such as
`glogg <https://glogg.bonnefon.org/>`_. Alternatively, remember you can quickly filter the standard output in this way:

.. code-block:: sh

    bin/tuto01basic --log-debug 2> >(grep PATTERN >&2)


Exceptions
------------

Exceptions can benefit from the logging system as well. They will be reported as **warnings**, so like if they would be
reported by ``SLM_WARN()``. For this, use one of the following macros:

.. code-block:: cpp

    // Basic versions
    FW_RAISE(Message)
    FW_RAISE_IF(Message, Condition)

    // Alternatives with exception class specified, inheriting from ::fwCore::Exception
    FW_RAISE_EXCEPTION(ExceptionType)
    FW_RAISE_EXCEPTION_MSG(ExceptionType, Message)
    FW_RAISE_EXCEPTION_IF(ExceptionType, Condition)

User errors
------------

All the logging features shown above are mainly intended for developers. They might be used for end-users in the case of
command line applications. But in the more frequent case of graphical applications, it is irrelevant to use these to
report errors to end-users.

To report end-users errors or notifications, it is necessary to use popup dialogs available in ``::fwGui::dialog``
and notably the helper function ``::fwGui::dialog::MessageDialog``:

.. code-block:: cpp

    IMessageDialog::Buttons MessageDialog::showMessageDialog(
        const std::string& title, const std::string& message, ::fwGui::dialog::IMessageDialog::Icons icon)

    // Example
    ::fwGui::dialog::MessageDialog::showMessageDialog(
        "Pop-up window title",
        "This is a undesirable event.\nPlease report this to the nearest developer.",
        ::fwGui::dialog::IMessageDialog::CRITICAL);

where the last parameter can be one of:

.. code-block:: cpp

    typedef enum
    {
        CRITICAL,
        WARNING,
        INFO,
        QUESTION,
        NONE
    } Icons;

Last, for less intrusive notifications and a more modern approach to give user feedback, we also provide support for
notifications. This mechanism can be used in two ways. The first way is to use a library function, directly from your
C++ code:

.. code-block:: cpp

    void NotificationDialog::showNotificationDialog( const std::string&,
                                                     INotificationDialog::Type, INotificationDialog::Position )

    // Example
    ::dial::NotificationDialog::showNotificationDialog("Notification Test !", m_type,
                                                        ::dial::NotificationDialog::Position::TOP_LEFT );

The second and preferred way is to use a dedicated service  called
`SNotifier <https://sight.pages.ircad.fr/sight/classguiQt_1_1SNotifier.html#details>`_.
This service triggers notification on the given GUI container through different slots. Its usage is well demonstrated in
the sample ``ExNotifications``, please refer to it for more detailed explanations.
