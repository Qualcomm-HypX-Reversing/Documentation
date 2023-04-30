HypX Internal Logging Mechanism
---------------------------------

HypX has a fairly simple logging mechanism.

The HypX logging mechanism starts at a function called ``dprintf`` which works similar to how ``dprintf`` works in libc. 

In HypX's version of ``dprintf``, there are 2 valid fds: 3 and 4. 3 indicates a regular log while 4 indicates a panic log. If 4 is run then the code will infinite loop. 

Do note that this logging mechanism is only used for hypervisor internal logging. There is a seperate logging mechanism for the RKP log. This logging mechanism writes data to a known physical address so that the kernel can expose it in ``/proc/uh_log``.

Initializing the Log
==========================

The log is initialized in a function called ``initialize_log``. This sets up a few key variables for the logging mechanism.

The first important variable is ``log_buf`` which is where the log starts. This is always ``0x80209200``. The second is ``max_log_size`` which indicates the size of the log before it resets to the beginning. 


A look at ``dprintf`` 
==========================

``dprintf`` is a fairly complicated function but not much of it actually matters.  The main magic happens at ``__dprintf`` -> ``write_to_log``. ``write_to_log`` is where the format string stuff happens. To actually write the formatted string to the log, a function called ``putchar`` is used to write each character individually. ``putchar`` calls another subroutine called ``__putchar`` which has the following implementation:

.. code-block:: C
    :caption: ``__putchar``
    
    void __putchar(char c)

    {
    uint next_log_cursor;
    
    if (log_initialized? != 0) {
        *(char *)(log_buf + (ulong)log_cursor) = c;
        next_log_cursor = 0;
        if (max_log_size != 0) {
        next_log_cursor = (log_cursor + 1) / max_log_size;
        }
        next_log_cursor = (log_cursor + 1) - next_log_cursor * max_log_size;
        log_cursor = (ushort)next_log_cursor;
        if ((next_log_cursor & 0xffff) == 0) {
        DAT_8020900c = DAT_8020900c + 1;
        return;
        }
    }
    return;
    }

The above implementation is fairly simple. All it does is write the character to ``log_buf + log_cursor``. If ``log_cursor+1`` (The next place to write) is equal to ``max_log_size``. Then, we reset the ``log_cursor`` to ``0``. This ensures that the log doesn't overflow data. 

The format of a log entry is as follows: 

[cpu] [timestamp] [log text]


