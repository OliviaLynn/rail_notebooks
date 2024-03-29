Run a Saved Pipeline
====================

**Author:** Eric Charles

**Last Run Successfully:** April 26, 2023

This notebook shows how to:

1. Load a pipeline from a saved yaml file

2. Run the loaded pipeline

.. code:: ipython3

    import os
    import ceci
    from rail.core.utils import find_rail_file
    
    # To create a catalog, you need a model of what the distrubutions of the colors 
    # are--that's what this flow file is:
    flow_file = find_rail_file('examples_data/goldenspike_data/data/pretrained_flow.pkl')
    os.environ['FLOWDIR'] = os.path.dirname(flow_file)

Each pipeline file has an associated config file. Whenever ceci reads in
a pipeline file called ``[name].yml``, it will automatically look for a
configuration file ``[name]_config.yml`` in the same directory.

Here, we read in our ``pipe_example.yml`` pipeline, which is associated
with ``pipe_example.config.yml``:

.. code:: ipython3

    
    p = ceci.Pipeline.read('pipe_example.yml')


.. parsed-literal::

    Inserting handle into data store.  model: /opt/hostedtoolcache/Python/3.10.14/x64/lib/python3.10/site-packages/rail/examples_data/goldenspike_data/data/pretrained_flow.pkl, flow_engine_test


.. code:: ipython3

    p.run()


.. parsed-literal::

    
    Executing flow_engine_test
    Command is:
    OMP_NUM_THREADS=1   python3 -m ceci rail.creation.engines.flowEngine.FlowCreator   --model=/opt/hostedtoolcache/Python/3.10.14/x64/lib/python3.10/site-packages/rail/examples_data/goldenspike_data/data/pretrained_flow.pkl   --name=flow_engine_test   --config=pipe_example_config.yml   --output=./output_flow_engine_test.pq 
    Output writing to ./flow_engine_test.out
    


.. parsed-literal::

    Job flow_engine_test has completed successfully!


.. parsed-literal::

    
    Executing lsst_error_model_test
    Command is:
    OMP_NUM_THREADS=1   python3 -m ceci rail.creation.degradation.lsst_error_model.LSSTErrorModel   --input=./output_flow_engine_test.pq   --name=lsst_error_model_test   --config=pipe_example_config.yml   --output=./output_lsst_error_model_test.pq 
    Output writing to ./lsst_error_model_test.out
    


.. parsed-literal::

    Job lsst_error_model_test has completed successfully!


.. parsed-literal::

    
    Executing col_remapper_test
    Command is:
    OMP_NUM_THREADS=1   python3 -m ceci rail.core.util_stages.ColumnMapper   --input=./output_lsst_error_model_test.pq   --name=col_remapper_test   --config=pipe_example_config.yml   --output=./output_col_remapper_test.pq 
    Output writing to ./col_remapper_test.out
    


.. parsed-literal::

    Job col_remapper_test has completed successfully!


.. parsed-literal::

    
    Executing table_conv_test
    Command is:
    OMP_NUM_THREADS=1   python3 -m ceci rail.core.util_stages.TableConverter   --input=./output_col_remapper_test.pq   --name=table_conv_test   --config=pipe_example_config.yml   --output=./output_table_conv_test.hdf5 
    Output writing to ./table_conv_test.out
    


.. parsed-literal::

    Job table_conv_test has completed successfully!




.. parsed-literal::

    0



Yep, that’s it.
