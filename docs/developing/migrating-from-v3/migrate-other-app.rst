###################
Custom App Workflow
###################

Experienced developers should be able to use some of these steps individually to accomplish discrete tasks, but we generally recommend following this workflow as a whole.

.. note:: All of the commands below must be run from within your v4 project.

1. Create a new package
-----------------------

.. code-block:: shell

    python manage.py packages -o create_package -d pkg

The result should be a new package within your project named ``pkg``::

    my_project/
    └─ manage.py
    └─ my_project/
    └─ pkg/

Now go into your project's ``my_project/my_project/settings.py`` file and add this new line somewhere after the ``APP_ROOT`` line::

    PACKAGE_DIR = os.path.join(os.path.dirname(APP_ROOT),'pkg')

.. note:: You can actually name your new package whatever you want, and place it wherever you want, as long as ``PACKAGE_DIR`` holds the path to it. You can even omit ``PACKAGE_DIR`` entirely if you pass ``--target path/to/package`` to all of the ``v3`` commands below.

Finally, load this package into your project::

    python manage.py packages -o load_package -s pkg -db true

.. important:: 
    We recommend using the ``-db true`` flag here, which will completely erase your v4 project database 
    and create a fresh installation. If you have already added a lot of new user logins to your v4 project,
    these will be lost. If you have already added settings to your project like a MapBox API key, for example, 
    follow these steps to retain them `before` running the command with ``-db true``:
    
    + In your v4 project, run ``python manage.py packages -o save_system_settings``
    + Find the newly created file ``my_project/my_project/system_settings/System_Settings.json`` and move it into ``my_project/pkg/system_settings``.
    + When you do run the load package command, say "y" to the prompt about overwriting project settings (they will be imported from this new settings file).

Before moving on you should be able to view your project in a browser and login with the default ``admin/admin`` credentials.

2. Prepare your package.
------------------------

.. code-block:: shell

    python manage.py v3 start-migration

This will create some new directories and content in your package::
    
    pkg/
      └─ reference_data/
          └─ v3topconcept_lookup.json
      └─ v3data/
          └─ business_data/
          └─ graph_data/
          └─ reference_data/

3. Move your exported v3 data into the package.
-----------------------------------------------

Move ``v3resources-all-<date>.json`` from :ref:`Export v3 Business Data` into ``v3data/business_data``. This file name could be slightly different for you (or you may have multiple files) based on how you ran the v3 export.

Move ``v3relations-all-<date>.csv`` from :ref:`Export v3 Resource Relations` into ``v3data/business_data``.

Move ``v3scheme-arches-<date>.xml`` from :ref:`Export v3 Reference Data` into ``v3data/reference_data``.

4. Move the v3 resource graph _nodes.csv files from v3 into your package.
-------------------------------------------------------------------------

In your Arches v3 deployment, you should be able to find these files in your original ``source_data/resource_graphs`` directory, whose contents should be a ``_edges.csv`` and ``_nodes.csv`` for every resource graph in your database. We only want the ``_nodes.csv`` files.

Move the ``_nodes.csv`` files into ``v3data/graph_data``.

----

After completing steps 3 and 4, your v4 package should look like this::

    pkg/
      └─ v3data/
          └─ business_data/
              └─ v3resources-all-<date>.json
              └─ v3relations-all-<date>.csv
          └─ graph_data/
              └─ RESOURCE_GRAPH_NAME.Exx_nodes.csv
              └─ etc.
          └─ reference_data/
              └─ v3scheme-arches-<date>.xml
          └─ rm_configs.json

5. Convert the v3 reference data.
----------------------------------

See :ref:`Arches-HIP workflow Step 3 <3. Convert the v3 reference data.>`, and return to this page when you've finished.

6. Build the v4 Resource Models.
--------------------------------

Now that the v3 reference data has been converted and loaded, you are ready to create the v4 Resource Models. This migration process does not attempt to create them based on your old v3 graphs. There are a number of reasons for this, but most simply, v4 graphs have different constraints and support different datatypes and structures than those in v3. In other words, your v4 database will be better off with graphs that have been created natively, not translated from v3.

Generally, we would expect the v4 graphs to look like their v3 analogs, but we have built in quite a bit of wiggle room:

    * The graph names can differ
    * The node names can differ
    * The graph structure can differ (though maintaining the same general branching structure is advisable)

However, there must still be a one-to-one relationship between v3 and v4 graphs and their nodes.

When it comes to node datatypes, the translation from v3 to v4 is pretty straight-forward.

.. table:: Datatype Translation -- v3 to v4
   :widths: auto

   ====================  ==================================
     v3 businesstable      v4 datatype
   ====================  ==================================
   ``strings``           ``string``
   ``dates``             ``date`` or ``edtf``
   ``geometries``        ``geojson-feature-collection``
   ``domains``           ``concept`` - if single value per v3 branch
   ``domains``           ``concept-list`` - if multiple values per v3 branch were allowed
   ====================  ==================================

.. important::

    When you set a v4 node to ``concept`` or ``concept-list``, you will need to select which collection to use. This is why it's best to have migrated and loaded your RDM scheme (step 5 above) before making the Resource Models.

.. seealso::
    
    Refer to :ref:`Designing the Database` for help on this task. Within the Arches Designer itself, click |help-btn| for detailed help on each page.

----

Once you have built all of the Resource Models, **export them into your package**. You can do this one-by-one from the Arches Designer interface, or use::

    python manage.py packages -o export_graphs -d pkg/graphs/resource_models -g "all"

.. warning::

    If you have made any Branches, using the ``-g "all"`` argument will export them as well, which you don't want. You'll have to remove them from ``pkg/graph/resource_models`` and/or move them into ``pkg/graph/branches`` before moving on.
    
By the end of this step, you should have one JSON file per Resource Model in ``pkg/graphs/resource_models``.

7. Generate and populate the node lookup files.
-----------------------------------------------

Begin by running::

    python manage.py v3 generate-rm-configs

which will create ``v3data/rm_configs.json``. This file will be used to link the name of your v4 Resource Models with the names of their corresponding v3 graphs, as well as point to the files that link each node. Initially its content will look like::

    {
        "Activity": {
            "v3_entitytypeid": "<fill out manually>", 
            "v3_nodes_csv": "run 'python manage.py v3 generate-lookups", 
            "v3_v4_node_lookup": "run 'python manage.py v3 generate-lookups"
        }
    }

where ``"Activity"`` is the name of a v4 Resource Model. As the file says, you must now fill out the ``v3_entitytypeid`` value for all items. Typically, this will look something like ``"ACTIVITY.E7"``--upper-case with a CRM class appended to it.

Now, also as the file says, run::

    python manage.py v3 generate-lookups
    
and you'll see the rest of the values get filled out.

----

There will now be more CSV files in the ``v3data/graph_data`` directory. There is one per v3 graph, and they are used to match the names of v3 node names (column one), with v4 node names (column two). All of the v3 nodes will be listed for you, but **you have to fill out the v4 node names manually**, using your new Resource Models for reference. A portion of a filled out file could look like:

.. table:: ACTIVITY.E7_v4_lookup.csv
   :widths: auto

   ====================  ================
     v3_node              v4_node
   ====================  ================
   ACTIVITY_TYPE.E55     Activity Type
   ADDRESS_TYPE.E55      Address Type
   etc...                etc...
   ====================  ================

Finally, you can use::

    python manage.py v3 test-lookups

to check your work. Once this test passes, you can move on.

8. Convert the v3 JSON/JSONL business data
------------------------------------------

See :ref:`Arches-HIP workflow Step 4 <4. Convert the v3 JSON/JSONL business data.>`. (You can continue
using that workflow until you are finished with the migration.)

9. Write the v4 resource relations file.
----------------------------------------

See :ref:`Arches-HIP workflow Step 5 <5. Convert the v3 resource relations.>`. (You can continue
using that workflow until you are finished with the migration.)

10. Load the entire package (optional)
--------------------------------------

See :ref:`Arches-HIP workflow Step 6 <6. Load the entire package.>`.

.. |help-btn| image:: ../../images/in-app-help-icon.png
  :align: middle
