PZFlow Informer and Estimator Demo

Author: Tianqing Zhang

.. code:: ipython3

    %matplotlib inline
    import os
    import matplotlib.pyplot as plt
    import numpy as np
    import pandas as pd
    
    import rail
    from rail.core.data import TableHandle
    from rail.core.stage import RailStage
    import qp
    import tables_io
    
    from rail.estimation.algos.pzflow_nf import PZFlowInformer, PZFlowEstimator


.. code:: ipython3

    DS = RailStage.data_store
    DS.__class__.allow_overwrite = True

.. code:: ipython3

    from rail.utils.path_utils import find_rail_file
    trainFile = find_rail_file('examples_data/testdata/test_dc2_training_9816.hdf5')
    testFile = find_rail_file('examples_data/testdata/test_dc2_validation_9816.hdf5')
    training_data = DS.read_file("training_data", TableHandle, trainFile)
    test_data = DS.read_file("test_data", TableHandle, testFile)

.. code:: ipython3

    pzflow_dict = dict(hdf5_groupname='photometry',output_mode = 'not_fiducial' )



.. code:: ipython3

    
    # epoch = 200 gives a reasonable converged loss
    pzflow_train = PZFlowInformer.make_stage(name='inform_pzflow',model='demo_pzflow.pkl',num_training_epochs = 30, **pzflow_dict)


.. code:: ipython3

    # training of the pzflow
    pzflow_train.inform(training_data)


.. parsed-literal::

    Training 30 epochs 
    Loss:


.. parsed-literal::

    (0) 19.3210


.. parsed-literal::

    (1) 11.9689


.. parsed-literal::

    (2) 9.4665


.. parsed-literal::

    (3) 7.6701


.. parsed-literal::

    (4) 6.4975


.. parsed-literal::

    (5) 4.8184


.. parsed-literal::

    (6) 4.9988


.. parsed-literal::

    (7) 3.6832


.. parsed-literal::

    (8) 3.3320


.. parsed-literal::

    (9) 2.6474


.. parsed-literal::

    (10) 2.4241


.. parsed-literal::

    (11) 2.0601


.. parsed-literal::

    (12) 1.7141


.. parsed-literal::

    (13) 1.6782


.. parsed-literal::

    (14) 1.4572


.. parsed-literal::

    (15) 1.2378


.. parsed-literal::

    (16) 1.1748


.. parsed-literal::

    (17) 1.0077


.. parsed-literal::

    (18) 1.2034


.. parsed-literal::

    (19) 1.0326


.. parsed-literal::

    (20) 0.9658


.. parsed-literal::

    (21) 0.7357


.. parsed-literal::

    (22) 0.3684


.. parsed-literal::

    (23) 0.4813


.. parsed-literal::

    (24) 0.6877


.. parsed-literal::

    (25) 0.3737


.. parsed-literal::

    (26) 0.1206


.. parsed-literal::

    (27) 0.5171


.. parsed-literal::

    (28) 0.3822


.. parsed-literal::

    (29) 0.6297


.. parsed-literal::

    (30) 0.0781
    Inserting handle into data store.  model_inform_pzflow: inprogress_demo_pzflow.pkl, inform_pzflow




.. parsed-literal::

    <rail.tools.flow_handle.FlowHandle at 0x7f0954dcb9a0>



.. code:: ipython3

    pzflow_dict = dict(hdf5_groupname='photometry')
    
    pzflow_estimator = PZFlowEstimator.make_stage(name='estimate_pzflow',model='demo_pzflow.pkl',**pzflow_dict, chunk_size = 20000)

.. code:: ipython3

    # estimate using the test data
    estimate_results = pzflow_estimator.estimate(test_data)


.. parsed-literal::

    Inserting handle into data store.  model: demo_pzflow.pkl, estimate_pzflow
    Process 0 running estimator on chunk 0 - 20000


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/qp/interp_pdf.py:89: RuntimeWarning: invalid value encountered in divide
      self._ycumul = (self._ycumul.T / self._ycumul[:, -1]).T


.. parsed-literal::

    Inserting handle into data store.  output_estimate_pzflow: inprogress_output_estimate_pzflow.hdf5, estimate_pzflow
    Process 0 running estimator on chunk 20000 - 20449


.. code:: ipython3

    mode = estimate_results.read(force=True).ancil['zmode']
    truth = np.array(test_data.data['photometry']['redshift'])

.. code:: ipython3

    # visualize the prediction. 
    plt.figure(figsize = (8,8))
    plt.scatter(truth, mode, s = 0.5)
    plt.xlabel('True Redshift')
    plt.ylabel('Mode of Estimated Redshift')




.. parsed-literal::

    Text(0, 0.5, 'Mode of Estimated Redshift')




.. image:: ../../../docs/rendered/estimation_examples/09_PZFlow_files/../../../docs/rendered/estimation_examples/09_PZFlow_11_1.png


