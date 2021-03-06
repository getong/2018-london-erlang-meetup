rckv
===========

A riak_core application

Build
-----

::

    rebar3 release

Test
----

::

    rebar3 ct

Run
---

::

    rebar3 run

Try
---

::

    1> rckv:ping().
    {pong,753586781748746817198774991869333432010090217472}

.. code-block: erlang

    Bucket = <<"mybucket">>.
    Key1 = <<"mykey">>.
    Val1 = <<"myvalue">>.

    rckv:get({Bucket, Key1}).
    % {not_found,707914855582156101004909840846949587645842325504,
    %            {<<"mybucket">>,<<"mykey">>}}

    rckv:put({Bucket, Key1}, Val1).
    % {ok,707914855582156101004909840846949587645842325504}

    rckv:get({Bucket, Key1}).
    % {found,707914855582156101004909840846949587645842325504,
    %        {{<<"mybucket">>,<<"mykey">>},
    %               {{<<"mybucket">>,<<"mykey">>},<<"myvalue">>}}}

    rckv:delete({Bucket, Key1}).
    % {found,707914855582156101004909840846949587645842325504,
    %        {{<<"mybucket">>,<<"mykey">>},
    %               {{<<"mybucket">>,<<"mykey">>},<<"myvalue">>}}}

    rckv:get({Bucket, Key1}).
    % {not_found,707914855582156101004909840846949587645842325504,
    %            {<<"mybucket">>,<<"mykey">>}}


Quit
----

::

    2> q().

Play with Clustering
--------------------

Build 3 releases that can run on the same machine::

    make devrel

Start them in different consoles::

    make dev1-console
    make dev2-console
    make dev3-console

join 2 nodes to the first one::

    make devrel-join

check the status of the cluster::

    make devrel-status

you should see something like this::

    ================================= Membership ==================================
    Status     Ring    Pending    Node
    -------------------------------------------------------------------------------
    joining     0.0%      --      'rckv2@127.0.0.1'
    joining     0.0%      --      'rckv3@127.0.0.1'
    valid     100.0%      --      'rckv1@127.0.0.1'
    -------------------------------------------------------------------------------
    Valid:1 / Leaving:0 / Exiting:0 / Joining:2 / Down:0

it should say that 3 nodes are joining, now check the cluster plan::

    make devrel-cluster-plan

it should display the cluster plan, now we can commit the plan::

    make devrel-cluster-commit

check the status of the cluster again::

    make devrel-status

you could see the vnodes transfering::

    ================================= Membership ==================================
    Status     Ring    Pending    Node
    -------------------------------------------------------------------------------
    valid      75.0%     25.0%    'rckv1@127.0.0.1'
    valid       9.4%     25.0%    'rckv2@127.0.0.1'
    valid       7.8%     25.0%    'rckv3@127.0.0.1'
    -------------------------------------------------------------------------------
    Valid:3 / Leaving:0 / Exiting:0 / Joining:0 / Down:0

at some point you should see something like this::

    ================================= Membership ==================================
    Status     Ring    Pending    Node
    -------------------------------------------------------------------------------
    valid      33.3%      --      'rckv1@127.0.0.1'
    valid      33.3%      --      'rckv2@127.0.0.1'
    valid      33.3%      --      'rckv3@127.0.0.1'
    -------------------------------------------------------------------------------
    Valid:3 / Leaving:0 / Exiting:0 / Joining:0 / Down:0

when you are bored you can stop them::

    make devrel-stop

Riak Core Metadata
------------------

Create some variables in all nodes you are going to run the calls:

.. code-block:: erlang

    FullPrefix = {<<"rckv">>, <<"config">>}.
    Key1 = key_1.
    Val1 = <<"value 1">>.

Run each line on any node:

.. code-block:: erlang

    riak_core_metadata:get(FullPrefix, Key1).
    % undefined

    riak_core_metadata:get(FullPrefix, Key1, [{default, default_value_here}]).
    % default_value_here

    riak_core_metadata:put(FullPrefix, Key1, Val1).
    % ok

    riak_core_metadata:get(FullPrefix, Key1).
    % <<"value 1">>

    riak_core_metadata:to_list(FullPrefix).
    % [{key_1,[<<"value 1">>]}]

    riak_core_metadata:delete(FullPrefix, Key1).
    % ok

    riak_core_metadata:to_list(FullPrefix).
    % [{key_1,['$deleted']}]

Trace Metadata Calls:

.. code-block:: erlang

	ReturnTrace = fun(_) -> return_trace() end.
	% at most 1000 calls per second
	Rate = {1000, 1000}.
	recon_trace:calls([{riak_core_broadcast, '_',
		fun ([A, _]) when A /= lazy_tick -> return_trace() end},
		   {riak_core_metadata_hashtree, '_', ReturnTrace},
		   {riak_core_metadata_object, '_', ReturnTrace},
		   {riak_core_metadata_manager, '_', ReturnTrace},
		   {riak_core_metadata_exchange_fsm, '_', ReturnTrace},
		   {riak_core_metadata, '_', ReturnTrace}], Rate).

Clear the trace:

.. code-block:: erlang

	recon_trace:clear().


TODO
----

* define license and create LICENSE file

License
-------

TODO
