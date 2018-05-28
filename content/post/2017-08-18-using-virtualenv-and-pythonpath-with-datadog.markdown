---
author: "Cody Wilbourn"
categories:
- Alerting
comments: true
date: "2017-08-18T17:55:40Z"
slug: using-virtualenv-and-pythonpath-with-datadog
tags:
- datadog
- monitoring
title: Using virtualenv and PYTHONPATH with Datadog
---

Datadog is a great service I've used for monitoring. Since the agent is Python-based it's very extensible through a collection of `pip` installable libraries, but the documentation is limited on how to handle these libraries.

If you use the provided `datadog-agent` package, Datadog comes with its own set of embedded applications to monitor your server, including `python` for the agent, `supervisord` to manage the Datadog processes, and `pip`. Since this is all just Python, surely this can lead to something. Can't we import our own custom libraries in our custom checks? Yes we can.

<!--more-->

`datadog-agent` is unpacked (on my systems) to `/opt/datadog-agent`.

If you look inside the installation directory, of particular note is the `embedded` subdirectory. This contains the embedded python interpreter that's used in order to avoid conflicts with your system python. The Datadog Agent's Python is currently `v2.7.13`.

You can only go so far with what's included in the existing `pip` environment, found by running `/opt/datadog-agent/embedded/bin/pip freeze`. You may find yourself wanting for more.

Luckily, you can extend this environment by running `/opt/datadog-agent/embedded/bin/pip install package_name` to add more packages to Datadog. These packages will be available only for Datadog Agent's python, since the packages are installed into the `embedded` directory.

You might feel a bit dirty about modifying the contents of a directory that's provided as a vendor package. That's alright, there's a solution for this as well -- Datadog supports modifying `$PYTHONPATH` of the running checks, and we can use Python virtual environments to provide packages for the running agent.

Basically, you can modify the YAML for a check to provide additional search paths for Python. It looks like this:

{{< highlight text >}}
init_config:
# Fill with your init_config

instances:
# Fill with your instance config

# Configure Datadog to use an external python path
pythonpath:
- "/path/to/virtualenv"
- "/path/to/your/code"
{{< / highlight >}}

This configuration will allow you to run Python code out of existing virtual environments and/or directories, so you can leverage your custom code to write checks. (Thanks to Datadog support for helping me out with this!)

However, it is highly likely the Python interpreter provided by Datadog and the Python interpreter used by your code are different. Chiefly, the Datadog interpreter is a "narrow" build of python, using `ucs2` for internal encoding, while your code's python is most likely a "wide" build using `ucs4`. As soon as your AgentCheck tries to use any `pip` packages with compiled extensions from this other virtual environment, your AgentCheck will break.

This issue is fixable by creating a virtualenv dedicated to the Datadog Agent. A separate virtual environment has the added benefit of being an easier solution to deploy via Chef or another configuration management tool, instead of installing packages into the `embedded/bin/pip`

{{< highlight shell >}}
mkvirtualenv --python=/opt/datadog-agent/embedded/bin/python my_datadog_virtualenv
# If you have the virtualenv activated, $VIRTUAL_ENV is the path to the venv
$VIRTUAL_ENV/bin/pip install -r requirements.txt
{{< / highlight >}}

Happy Monitoring!
