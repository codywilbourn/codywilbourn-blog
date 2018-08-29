---
title: "Playing with Python Structured Logs"
date: 2018-08-23T15:25:39-05:00
author: "Cody Wilbourn"
---

Recently I've been experimenting with the Python project [structlog](http://www.structlog.org/en/stable/index.html)
to add structured logging to our in-house applications. The eventual plan
would be to emit the logs to an ELK stack with JSON parsing, instead of
the much more complicated set of rules we have to custom define for each
type of log file ingested today.

This post is my current understanding of `structlog`, reordered from how the
docs are arranged to make a bit more sense to me. If you're familiar with
the library and have something to add--[let me know]({{< ref "contact" >}}),
I'd love to hear from you.


# Basic Usage
Structlog is installed with `pip install structlog`, optionally adding
`colorama` for color output in a terminal.

{{< highlight python >}}
# Old code
import logging
log = logging.getLogger()

# New code
import structlog
log = structlog.getLogger() # or structlog.get_logger()
{{< / highlight >}}

At this point the logging usage is identical, but you can add key-value
pairs to the log calls.

{{< highlight python >}}
log.info("Hello World", a=1)
# 2018-08-23 17:59.51 Hello World                    a=1
{{< / highlight >}}

Already this is an improvement for tools like `awk` or `sed`.


# Powerful Structured Logging
## Configuring structlog
While the basic usage is great for getting started, it still has a lot
of training wheels attached.

The power of structlog comes from the concept of "processors", which are
a pipeline of functions that modify the event -- the `event_dict` in all
of the example code. Since `event_dict` is a dictionary, keys can be
modified as needed in each function before the next function is called.

The last step in the processor function chain is the `Renderer` function,
which takes in the `event_dict` and "renders" it appropriately for the
output handler. By default, and with most of the examples, this is
the `logging` library from Python's stdlib. So effectively there's an
arbitrary `dict` that gets massaged into the kwargs for `logging.getLogger().log()`.

{{< highlight python >}}
import structlog
# This configures the module, needed only once per process.
# Any subsequent code only calls structlog.get_logger()
structlog.configure(processors=[
    structlog.stdlib.add_log_level,
    structlog.processors.KeyValueRenderer(sort_keys=True)
])

log = structlog.get_logger()
log.info("Hello World")

# This outputs
# event='Hello World' level='info'
{{< / highlight >}}


However, the basic usage comes with a bunch of processors that have to
be explicitly defined when `processors` is modified, since it's a list
rather than a numerically ordered set of middlewares like Scrapy.

This is the [documented suggested configuration](http://www.structlog.org/en/stable/standard-library.html#suggested-configurations)
for use with `logging`:

{{< highlight python >}}
import structlog

structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.stdlib.PositionalArgumentsFormatter(),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
        structlog.processors.UnicodeDecoder(),
        structlog.stdlib.render_to_log_kwargs,
    ],
    context_class=dict,
    logger_factory=structlog.stdlib.LoggerFactory(),
    wrapper_class=structlog.stdlib.BoundLogger,
    cache_logger_on_first_use=True,
)
{{< / highlight >}}

At this point I came to a screeching halt and had to take a step back
to understand what was going on.


## Breaking Down The Processors

The processors in question do the normal pipeline of operations that `logging`
does, but I've never gone deep into `logging` except to know that:

1. `logging` will only output when the desired [log level](https://docs.python.org/3/library/logging.html#levels) meets the logging level threshold.
1. I should use `%` formatted strings and pass the substitutions as args to `log.log()`

These are optimizations in the logging library to minimize unnecessary processing.
The first rule allows us to entirely skip DEBUG messages when we're not logging
DEBUG. The second rule allows us to bypass string interpolation on those calls.

So similarly, our processors do this work.


{{< highlight python >}}
import structlog

structlog.configure(
    processors=[
        # This performs the initial filtering, so we don't
        # evaluate e.g. DEBUG when unnecessary
        structlog.stdlib.filter_by_level,
        # Adds logger=module_name (e.g __main__)
        structlog.stdlib.add_logger_name,
        # Adds level=info, debug, etc.
        structlog.stdlib.add_log_level,
        # Performs the % string interpolation as expected
        structlog.stdlib.PositionalArgumentsFormatter(),
        # Include the stack when stack_info=True
        structlog.processors.StackInfoRenderer(),
        # Include the exception when exc_info=True
        # e.g log.exception() or log.warning(exc_info=True)'s behavior
        structlog.processors.format_exc_info,
        # Decodes the unicode values in any kv pairs
        structlog.processors.UnicodeDecoder(),
        # Creates the necessary args, kwargs for log()
        structlog.stdlib.render_to_log_kwargs,
    ],
    # Our "event_dict" is explicitly a dict
    # There's also structlog.threadlocal.wrap_dict(dict) in some examples
    # which keeps global context as well as thread locals
    context_class=dict,
    # Provides the logging.Logger for the underlaying log call
    logger_factory=structlog.stdlib.LoggerFactory(),
    # Provides predefined methods - log.debug(), log.info(), etc.
    wrapper_class=structlog.stdlib.BoundLogger,
    # Caching of our logger
    cache_logger_on_first_use=True,
)
{{< / highlight >}}


### Other batteries included
Other nifty processors that come out of the box include:

* `structlog.processors.ExceptionPrettyPrinter()`, which will pop the exception
information off the `event_dict` and print the familiar "Traceback (most recent call last):" text.
* `structlog.dev.ConsoleRenderer()`, which provides some nicer terminal layouts than the unconfigured structlog.
* `structlog.processors.JSONRenderer()`, JSON. Call with the `sort_keys=True` for human readability
* `structlog.processors.TimeStamper()`, adds a timestamp key, defaulting to UTC.

The structlog processor library is a mix of classes and functions. The
classes initially tripped me up with an error message about `__init__`
positional arguments before I realized not everything was a function.

Any classes have to be instantiated (generally with a configuration)
before being included in the processor pipeline. The processor then makes
a call to the object, treating it as a function. This works because all
of these included classes are defined with the magic method `__call__`.


## Custom Processors

This idea of processors opens up significant power to customize the
logging pipeline globally. As mentioned, the pipeline is a list of
Python functions.

Beyond tagging enforcement, like adding a timestamp and host info, or
modifying the pipeline based on a deployment target, these custom functions
allow for event sampling--very necessary to avoid data collection overload.

Any processor can prevent an event from continuing through the processing
chain by raising the exception `structlog.DropEvent`.

Here's an example with the ever popular FizzBuzz that will drop
every "Fizz", as well as every other "FizzBuzz". While just a toy example,
this could be extended to anything -- log every delete but only 1 of 100
search queries.

{{< highlight python >}}
class FizzDropper(object):
    def __call__(self, logger, method_name, event_dict):
        # An example of keying off log message
        if event_dict.get('event') == "Fizz":
            raise structlog.DropEvent
        # An example of keying off additional data
        count = event_dict.get('count')
        if count and count % 3 == 0 and count % 5 != 0:
            raise structlog.DropEvent
        return event_dict


class FizzBuzzDropper(object):
    counter = 0
    def __call__(self, logger, method_name, event_dict):
        if event_dict.get('event') == "Fizz Buzz":
            self.counter += 1
            if self.counter % 2 == 0:
                raise structlog.DropEvent
        return event_dict


structlog.configure(
    processors=[
        FizzDropper(),
        FizzBuzzDropper(),
        structlog.dev.ConsoleRenderer(pad_event=15)
    ],
)
log = get_logger()


for i in range(1, 100):
    if i % 3 == 0 and i % 5 == 0:
        log.error("Fizz Buzz", count=i)
    elif i % 3 == 0:
        # This should go missing!
        log.info("Fizz", count=i)
    elif i % 5 == 0:
        log.info("Buzz", count=i)
    else:
        log.info(str(i), count=i)

{{< / highlight >}}

Simple sampling like this is easy, effectively sampling is a hard problem.
I recommend reading Baron Schwartz's blog post on [selecting representative samples](https://www.vividcortex.com/blog/2015/07/03/representative-samples-stream-queries/)
if you're interested more in sampling.

# Migrating to structlog

If I could update all the code in my environment at once, switching to `structlog`
could be done in an afternoon. However, I have multiple projects with
their own update schedules. I also have [Sentry](https://sentry.io) to contend with, which is
tied into the `logging` library.

## Sentry
I found [one Github issue](https://github.com/hynek/structlog/issues/18)
relating to configuring structlog with Sentry, but it ended up confusing
me more than helping. So here's what did work for me in testing:


{{< highlight python >}}
import logging
from raven import Client
from raven.handlers.logging import SentryHandler

sentry_handler = SentryHandler(Client(SENTRY_DSN))
sentry_handler.setLevel(logging.ERROR)
logging.getLogger().addHandler(sentry_handler)

log = structlog.get_logger()

try:
    1 / 0
except ZeroDivisionError:
    log.exception("Div by Zero")

{{< / highlight >}}

Conceptually what's happening is we configure a logging handler,
attach it to the root logger, and then let structlog fetch the root logger
via the default `logger_factory`.


## External libraries

Without changing everything to `structlog` at once, I also have to contend
with loggers in imported libraries that have not yet been updated or are
outside my control--otherwise the logs will be half JSON, half traditional
Python messages. That's a sure guarantee to break any parser reading
these logs.

The [structlog docs](http://www.structlog.org/en/stable/standard-library.html?#rendering-using-logging-based-formatters) suggest `python-json-logger`.

Pieced together from a few code snippets in the examples, we get a setup
that looks like this:

{{< highlight python >}}
import logging
import sys
from pythonjsonlogger import jsonlogger

# Setup jsonlogger to print JSON
json_handler = logging.StreamHandler(sys.stdout)
json_handler.setFormatter(jsonlogger.JsonFormatter())

# Setup Sentry to only report on errors
sentry_handler = SentryHandler(Client(SENTRY_DSN))
sentry_handler.setLevel(logging.ERROR)

# Add both handlers to logging
logging.basicConfig(
    format="%(message)s",
    handlers=[json_handler, sentry_handler],
    level=logging.INFO,  # Or whatever the general level should be
)

# The processor list
processors = [
    structlog.stdlib.filter_by_level,  # First step, filter by level to
    structlog.stdlib.add_logger_name,  # module name
    structlog.stdlib.add_log_level,  # log level
    structlog.stdlib.PositionalArgumentsFormatter(),  # % formatting
    structlog.processors.StackInfoRenderer(),  # adds stack if stack_info=True
    structlog.processors.format_exc_info,  # Formats exc_info
    structlog.processors.UnicodeDecoder(),  # Decodes all bytes in dict to unicode
    structlog.processors.TimeStamper(),  # Because timestamps! UTC by default
    structlog.stdlib.render_to_log_kwargs,  # Preps for logging call
]

# Configure structlog
configure(
    processors=processors,
    context_class=dict,
    logger_factory=structlog.stdlib.LoggerFactory(),
    wrapper_class=structlog.stdlib.BoundLogger,
    cache_logger_on_first_use=True,
)

log = get_logger()

# App code
log.info("Starting app")
from module import foo
foo()  # function with logging.getLogger().info("foo")

{{< / highlight >}}

Now any modules I import in my application will log with JSON, albeit
without my pipeline additions. My text in the `log.log()` becomes the
`"message"` key of the JSON.

{{< highlight json >}}
{"message": "Starting app", "logger": "__main__", "level": "info", "timestamp": 1535063721.076024}
{"message": "foo"}
{{< / highlight >}}

If I used a renderer other than `render_to_log_kwargs`, there's a possibility
the entire JSON will instead be encoded as the value in the message key, like
`{"message": "event='Starting' level='info' logger='__main__' timestamp=1535058769.622922"}`.
It's important to double check the format of the output while testing and
that Sentry is correctly grouping the log messages sent.

Now I can gradually migrate over my internal libraries to structlog,
but all of the application's logs switch to JSON at one time.