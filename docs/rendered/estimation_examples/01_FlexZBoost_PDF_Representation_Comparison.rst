FlexZBoost PDF Representation Comparison
========================================

**Author:** Drew Oldag

**Last Run Successfully:** September 26, 2023

This notebook does a quick comparison of storage requirements for
Flexcode output using two different storage techniques. We’ll compare
``qp.interp`` (x,y interpolated) output against the native
parameterization of ``qp_flexzboost``.

.. code:: ipython3

    import os
    import matplotlib.pyplot as plt
    import numpy as np
    
    import qp
    
    from rail.core.data import TableHandle
    from rail.core.stage import RailStage
    
    %matplotlib inline 

.. code:: ipython3

    DS = RailStage.data_store
    DS.__class__.allow_overwrite = True

Create references to the training and test data.

.. code:: ipython3

    from rail.utils.path_utils import find_rail_file
    trainFile = find_rail_file('examples_data/testdata/test_dc2_training_9816.hdf5')
    testFile = find_rail_file('examples_data/testdata/test_dc2_validation_9816.hdf5')
    training_data = DS.read_file("training_data", TableHandle, trainFile)
    test_data = DS.read_file("test_data", TableHandle, testFile)

Define the configurations for the ML model to be trained by Flexcode.
Specifically we’ll use Xgboost with a set of 35 cosine basis functions.

.. code:: ipython3

    fz_dict = dict(zmin=0.0, zmax=3.0, nzbins=301,
                   trainfrac=0.75, bumpmin=0.02, bumpmax=0.35,
                   nbump=20, sharpmin=0.7, sharpmax=2.1, nsharp=15,
                   max_basis=35, basis_system='cosine',
                   hdf5_groupname='photometry',
                   regression_params={'max_depth': 8,'objective':'reg:squarederror'})
    
    fz_modelfile = 'demo_FZB_model.pkl'

Define the RAIL stage to train the model

.. code:: ipython3

    from rail.estimation.algos.flexzboost import FlexZBoostInformer, FlexZBoostEstimator
    inform_pzflex = FlexZBoostInformer.make_stage(name='inform_fzboost', model=fz_modelfile, **fz_dict)

Then we’ll run that stage to train the model and store the result in a
file name ``demo_FZB_model.pkl``.

.. code:: ipython3

    %%time
    inform_pzflex.inform(training_data)


.. parsed-literal::

    stacking some data...
    read in training data
    fit the model...


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:33] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:33] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:33] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:33] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:34] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:34] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:34] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:34] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:35] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:35] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:35] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:35] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:36] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:36] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:36] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:36] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:37] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:37] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:37] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:37] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:38] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:38] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:38] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:38] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:39] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:39] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:39] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:39] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:40] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [06:59:40] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    finding best bump thresh...


.. parsed-literal::

    finding best sharpen parameter...


.. parsed-literal::

    Retraining with full training set...


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:30] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:30] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:30] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:30] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:31] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:31] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:31] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:31] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:32] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:32] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:32] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:32] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:33] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:33] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:33] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:33] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:34] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:34] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:34] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:34] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:35] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:35] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:35] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:35] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:36] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:36] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:36] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:36] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:37] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:37] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:37] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:37] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:38] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)
    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/xgboost/training.py:183: UserWarning: [07:00:38] WARNING: /workspace/src/learner.cc:738: 
    Parameters: { "silent" } are not used.
    
      bst.update(dtrain, iteration=i, fobj=obj)


.. parsed-literal::

    Inserting handle into data store.  model_inform_fzboost: inprogress_demo_FZB_model.pkl, inform_fzboost
    CPU times: user 57.4 s, sys: 1.06 s, total: 58.4 s
    Wall time: 1min 7s




.. parsed-literal::

    <rail.core.data.ModelHandle at 0x7f7d4c98f520>



Now we configure the RAIL stage that will evaluate test data using the
saved model. Note that we specify ``qp_representation='flexzboost'``
here to instruct ``rail_flexzboost`` to store the model weights using
``qp_flexzboost``.

.. code:: ipython3

    pzflex_qp_flexzboost = FlexZBoostEstimator.make_stage(name='fzboost_flexzboost', hdf5_groupname='photometry',
                                model=inform_pzflex.get_handle('model'),
                                output='flexzboost.hdf5',
                                qp_representation='flexzboost')

Now we actually evaluate the test data, 20,449 example galaxies, using
the trained model, and then print out the size of the file that was
saved.

Note that the final output size will depend on the number of basis
functions used by the model. Again, for this experiment, we used 35
basis functions.

.. code:: ipython3

    %%time
    output_file_name = './flexzboost.hdf5'
    try:
        os.unlink(output_file_name)
    except FileNotFoundError:
        pass
    
    fzresults_qp_flexzboost = pzflex_qp_flexzboost.estimate(test_data)
    file_size = os.path.getsize(output_file_name)
    print("File Size is :", file_size, "bytes")


.. parsed-literal::

    Process 0 running estimator on chunk 0 - 10000
    Process 0 estimating PZ PDF for rows 0 - 10,000


.. parsed-literal::

    Inserting handle into data store.  output_fzboost_flexzboost: inprogress_flexzboost.hdf5, fzboost_flexzboost
    Process 0 running estimator on chunk 10000 - 20000
    Process 0 estimating PZ PDF for rows 10,000 - 20,000


.. parsed-literal::

    Process 0 running estimator on chunk 20000 - 20449
    Process 0 estimating PZ PDF for rows 20,000 - 20,449
    File Size is : 2870476 bytes
    CPU times: user 541 ms, sys: 70.7 ms, total: 611 ms
    Wall time: 1.35 s


Example calculating median and mode. Note that we’re using the
``%%timeit`` magic command to get an estimate of the time required for
calculating ``median``, but we’re using ``%%time`` to estimate the
``mode``. This is because ``qp`` will cache the output of the ``pdf``
function for a given grid. If we used ``%%timeit``, then the resulting
estimate would average the run time of one non-cached calculation and
N-1 cached calculations.

.. code:: ipython3

    zgrid = np.linspace(0, 3., 301)

.. code:: ipython3

    %%time
    fz_medians_qp_flexzboost = fzresults_qp_flexzboost().median()


.. parsed-literal::

    CPU times: user 9.12 s, sys: 17.6 ms, total: 9.13 s
    Wall time: 8.8 s


.. code:: ipython3

    %%time
    fz_modes_qp_flexzboost = fzresults_qp_flexzboost().mode(grid=zgrid)


.. parsed-literal::

    CPU times: user 11.3 s, sys: 43.8 ms, total: 11.3 s
    Wall time: 10.9 s


Plotting median values.

.. code:: ipython3

    fz_medians_qp_flexzboost = fzresults_qp_flexzboost().median()
    
    plt.hist(fz_medians_qp_flexzboost, bins=np.linspace(-.005,3.005,101));
    plt.xlabel("redshift")
    plt.ylabel("Number")
    bins = np.linspace(-5, 5, 11)



.. image:: ../../../docs/rendered/estimation_examples/01_FlexZBoost_PDF_Representation_Comparison_files/../../../docs/rendered/estimation_examples/01_FlexZBoost_PDF_Representation_Comparison_20_0.png


Example convertion to a ``qp.hist`` histogram representation.

.. code:: ipython3

    %%timeit
    bins = np.linspace(-5, 5, 11)
    fzresults_qp_flexzboost().convert_to(qp.hist_gen, bins=bins)


.. parsed-literal::

    8.27 s ± 27.6 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


Now we’ll repeat the experiment using ``qp.interp`` storage. Again,
we’ll define the RAIL stage to evaluate the test data using the saved
model, but instruct ``rail_flexzboost`` to store the output as x,y
interpolated values using ``qp.interp``.

.. code:: ipython3

    pzflex_qp_interp = FlexZBoostEstimator.make_stage(name='fzboost_interp', hdf5_groupname='photometry',
                                model=inform_pzflex.get_handle('model'),
                                output='interp.hdf5',
                                qp_representation='interp',
                                calculated_point_estimates=[])

Finally we evaluate the test data again using the trained model, and
then print out the size of the file that was saved using the x,y
interpolated technique.

The final file size will depend on the size of the x grid that defines
the interpolation. However, we can see that in order to match the
storage requirements of ``qp_flexzboost``, the x grid would need to be
smaller than the number of basis functions used by the model. For this
experiment, we used 35 basis functions.

.. code:: ipython3

    %%time
    output_file_name = './interp.hdf5'
    try:
        os.unlink(output_file_name)
    except FileNotFoundError:
        pass
    
    fzresults_qp_interp = pzflex_qp_interp.estimate(test_data)
    file_size = os.path.getsize(output_file_name)
    print("File Size is :", file_size, "bytes")


.. parsed-literal::

    Process 0 running estimator on chunk 0 - 10000
    Process 0 estimating PZ PDF for rows 0 - 10,000


.. parsed-literal::

    Inserting handle into data store.  output_fzboost_interp: inprogress_interp.hdf5, fzboost_interp
    Process 0 running estimator on chunk 10000 - 20000
    Process 0 estimating PZ PDF for rows 10,000 - 20,000


.. parsed-literal::

    Process 0 running estimator on chunk 20000 - 20449
    Process 0 estimating PZ PDF for rows 20,000 - 20,449


.. parsed-literal::

    File Size is : 49248094 bytes
    CPU times: user 12.5 s, sys: 179 ms, total: 12.7 s
    Wall time: 12.4 s


Example calculating median and mode. Note that we’re using the
``%%timeit`` magic command to get an estimate of the time required for
calculating ``median``, but we’re using ``%%time`` to estimate the
``mode``. This is because ``qp`` will cache the output of the ``pdf``
function for a given grid. If we used ``%%timeit``, then the resulting
estimate would average the run time of one non-cached calculation and
N-1 cached calculations.

.. code:: ipython3

    zgrid = np.linspace(0, 3., 301)

.. code:: ipython3

    %%timeit
    fz_medians_qp_interp = fzresults_qp_interp().median()


.. parsed-literal::

    865 ms ± 3.84 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


.. code:: ipython3

    %%time
    fz_modes_qp_interp = fzresults_qp_interp().mode(grid=zgrid)


.. parsed-literal::

    CPU times: user 185 ms, sys: 77.2 ms, total: 262 ms
    Wall time: 262 ms


Plotting median values.

.. code:: ipython3

    fz_medians_qp_interp = fzresults_qp_interp().median()
    plt.hist(fz_medians_qp_interp, bins=np.linspace(-.005,3.005,101));
    plt.xlabel("redshift")
    plt.ylabel("Number")




.. parsed-literal::

    Text(0, 0.5, 'Number')




.. image:: ../../../docs/rendered/estimation_examples/01_FlexZBoost_PDF_Representation_Comparison_files/../../../docs/rendered/estimation_examples/01_FlexZBoost_PDF_Representation_Comparison_32_1.png


Example convertion to a ``qp.hist`` histogram representation.

.. code:: ipython3

    %%timeit
    bins = np.linspace(-5, 5, 11)
    fzresults_qp_interp().convert_to(qp.hist_gen, bins=bins)


.. parsed-literal::

    35.3 ms ± 253 μs per loop (mean ± std. dev. of 7 runs, 10 loops each)


We’ll clean up the files that were produced: the model pickle file, and
the output data file.

.. code:: ipython3

    model_file_name = 'demo_FZB_model.pkl'
    flexzboost_file_name = './flexzboost.hdf5'
    interp_file_name = './interp.hdf5'
    
    try:
        os.unlink(model_file_name)
    except FileNotFoundError:
        pass
    
    try:
        os.unlink(flexzboost_file_name)
    except FileNotFoundError:
        pass
    
    try:
        os.unlink(interp_file_name)
    except FileNotFoundError:
        pass
