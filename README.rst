GHA: fiddling with manual dispatch
==================================

Is ``github.job`` available to the job itself on ``if:``?

The context of what I am trying to is described in the context. It seems to
indicate that ``github.job`` is not available. The SO answer here:

https://stackoverflow.com/questions/68382898/github-actions-run-job-from-input

Seems to indicate that as well.

Can somebody confirm whether or not ``github.job`` is available to the job
itself on ``if:``?

Thanks!

Context
=======

The idea is that on some projects it would be nice to just have a very quick
smoketest, and have remaining workflow only execute when manually triggered. So
something like this::

  on:
    push:
      branches:
      - '*'
      tags:
      - '*'
    pull_request:
      branches:
      - '*'
    workflow_dispatch:
      inputs:
        job:
          description: 'Job'
          required: true
          default: all
          type: choice
          options:
          - all
          - analyze
          - build
          - test
          - docs

And then for each job, utilize ``if:`` to guard them from invocation, something
like this::

    if: ((github.event_name == 'workflow_dispatch') && ((github.event.inputs.job == 'all') || (github.job == github.event.inputs.job)))

For this approach the following works:

  * Guarded jobs are not run on push and pull-request
  * Guarded jobs are run manually when 'all' is provided as input

Here is what does **not** work:

  * Guarded jobs are not running when the jobid is provided as input

For some reason the comparison between ``github.job`` and
``github.event.inputs.job`` never evaluates to True. Have tried comparison
using ``==``, ``contains`` and ``startsWith``. As well as directly comparing to
the jobid, e.g. ``github.job == 'docs'``.

The above seems to indicate ``github.job`` is not available to the ``if:`` on a
"job".

Work-around
-----------

One can replace the use of ``github.job`` with a hardcoded comparison for the
job in question, e.g.::

    if: ((github.event_name == 'workflow_dispatch') && ((github.event.inputs.job == 'all') || (github.event.inputs.job == 'test')))

However, this is error-prone... would be much better if ``github.job`` could be
used.
