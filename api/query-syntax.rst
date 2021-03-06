.. _api query:

API Query Syntax
================

Alerta API supports two types of query syntax.

 * standard URL query parameters
 * queries based on Lucene query syntax

Queries are supported by the following resource endpoints:

 * :ref:`alerts <get_alerts>`
 * :ref:`environments <environments>`
 * :ref:`services <services>`
 * :ref:`blackouts <get_blackouts>`
 * :ref:`heartbeats <get_heartbeats>`
 * :ref:`users <get_users>`
 * :ref:`customers <get_customers>`
 * :ref:`oembed <oembed>`

.. _url_query_params:

Standard URL Query parameters
-----------------------------

Alert attributes can be used as search parameters:

  * Any combination of valid alert attributes can be used to narrow down results.

  * Search syntax is ``=`` (equals), ``!=`` (not equals), ``=~`` (regex match)
    and ``!=~`` (regex exclude).

  * When searching for alert ``id`` the query will attempt to match against ``id``
    and ``lastReceiveId``. The "short id" (ie. first 8-characters) can
    be used. eg. ``id=ba358336`` instead of ``id=ba358336-802d-40ee-8ace-bf5fa8529280``.

  * Use `"dot notation"`_ to query custom attributes. eg. ``attributes.city=Berlin``

  * Alert ``history`` is limited to the 100 most recent status or severity changes.
    (set using ``HISTORY_LIMIT``)

  * If "customer views" is enabled then the appropriate ``customer`` filter for
    that user will be automatically applied.

.. _"dot notation": https://docs.mongodb.com/v3.2/core/document/#document-dot-notation

.. _query_string_syntax:

Query String Syntax
-------------------

The query string syntax is used by the ``q`` query string parameter. It is based
on the Lucene query string syntax and is described below.

.. contents::
   :local:
   :depth: 2

Search terms
~~~~~~~~~~~~

A search term can be a single word::

    foo

or::

    bar

A search term can also be a phrase, surrounded by double quotes, which searches
for all the words in the phrase, in the same order::

    "foo bar"

Field names
~~~~~~~~~~~

When no explicit field name is specified to search on in the query string
the default field ``text`` will be used unless a prefix is specified.

For example, where ``status`` field contains "ack"::

    status:ack

Where the ``group`` field contains "Network" or "Performance"::

    group:(Network OR Performance)
    group:(Network Performance)

Where the ``text`` field contains the exact phrase "kernel panic"::

    text:"kernel panic"

Where the custom attribute ``region`` has any non-null value::

    _exists_:region

Wildcards
~~~~~~~~~

Wildcard searches can be used on individual terms using ``?`` to replace
single characters and ``*`` to replace one or more characters:

To search for "foo", "fu", "bar" or "baz" use::

    f* ba?
    
To search for "test" or "text" use::

    te?t

Regular expressions
~~~~~~~~~~~~~~~~~~~

Regular expression patterns can be embedded in the query string by wrapping
them in forward-slashes (``/``). Typical examples include::

    /[mb]oat/

and::

    name:/joh?n(ath[oa]n)/

To search for numbered devices beginning with "net", "netwrk" or "network" use::

    resource:/net(wo?rk)?[0-9]/

.. note:: Regular expressions are implemented by the database backends so
    there may be subtle differences between `Postgres POSIX regular expressions`_
    and `MongoDB PCRE $regex pattern matching`_ in practice.

.. _Postgres POSIX regular expressions: https://www.postgresql.org/docs/9.6/static/functions-matching.html#FUNCTIONS-POSIX-REGEXP
.. _MongoDB PCRE $regex pattern matching: https://docs.mongodb.com/manual/reference/operator/query/regex/

Ranges
~~~~~~

Ranges can be specified for numeric or string fields. Inclusive
ranges are specified with square brackets ``[min TO max]`` and exclusive
ranges with curly brackets ``{min TO max}``::

    timeout:[1 TO 86400]
    group:{alpha TO zulu}
    value:{* TO 300}
    value:[500 TO *]

Ranges with one side unbounded (using ``*``) can use a simplified syntax::

    value:>500
    value:>=500
    value:<500
    value:<=500

Grouping
~~~~~~~~

Multiple terms or clauses can be grouped together with parentheses,
to form sub-queries::

    (foo OR bar) AND baz

Field Grouping
~~~~~~~~~~~~~~

Parentheses can be used to group multiple clauses to a single field::

    status:(open OR ack)
    text:(full text search)

.. note:: The following are not currently supported: boolean operators (+,-), range
    queries by date, and range queries based on severity levels.

.. note:: The following will not be supported: fuzziness, proximity searches, and
    boosting which are features specific to Lucene and/or Elasticsearch.
