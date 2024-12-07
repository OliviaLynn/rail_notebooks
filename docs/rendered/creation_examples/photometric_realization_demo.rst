Photometric Realization from Different Magnitude Error Models
=============================================================

author: John Franklin Crenshaw, Sam Schmidt, Eric Charles, Ziang Yan

last run successfully: August 2, 2023

This notebook demonstrates how to do photometric realization from
different magnitude error models. For more completed degrader demo, see
``degradation-demo.ipynb``

.. code:: ipython3

    import matplotlib.pyplot as plt
    from pzflow.examples import get_example_flow
    from rail.creation.engines.flowEngine import FlowCreator
    from rail.creation.degraders.photometric_errors import LSSTErrorModel
    from rail.core.stage import RailStage


Specify the path to the pretrained ‘pzflow’ used to generate samples

.. code:: ipython3

    import pzflow
    import os
    
    flow_file = os.path.join(
        os.path.dirname(pzflow.__file__), "example_files", "example-flow.pzflow.pkl"
    )


We’ll start by setting up the RAIL data store. RAIL uses
`ceci <https://github.com/LSSTDESC/ceci>`__, which is designed for
pipelines rather than interactive notebooks, the data store will work
around that and enable us to use data interactively. See the
``rail/examples/goldenspike_examples/goldenspike.ipynb`` example
notebook for more details on the Data Store.

.. code:: ipython3

    DS = RailStage.data_store
    DS.__class__.allow_overwrite = True


“True” Engine
~~~~~~~~~~~~~

First, let’s make an Engine that has no degradation. We can use it to
generate a “true” sample, to which we can compare all the degraded
samples below.

Note: in this example, we will use a normalizing flow engine from the
`pzflow <https://github.com/jfcrenshaw/pzflow>`__ package. However,
everything in this notebook is totally agnostic to what the underlying
engine is.

The Engine is a type of RailStage object, so we can make one using the
``RailStage.make_stage`` function for the class of Engine that we want.
We then pass in the configuration parameters as arguments to
``make_stage``.

.. code:: ipython3

    n_samples = int(1e5)
    flowEngine_truth = FlowCreator.make_stage(
        name="truth", model=flow_file, n_samples=n_samples
    )



.. parsed-literal::

    Inserting handle into data store.  model: /opt/hostedtoolcache/Python/3.10.15/x64/lib/python3.10/site-packages/pzflow/example_files/example-flow.pzflow.pkl, truth


Let’s check that the Engine correctly read the underlying PZ Flow
object:

.. code:: ipython3

    flowEngine_truth.get_data("model")





.. parsed-literal::

    <pzflow.flow.Flow at 0x7fb7644655a0>



Now we invoke the ``sample`` method to generate some samples
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note that this will return a ``DataHandle`` object, which can keep both
the data itself, and also the path to where the data is written. When
talking to rail stages we can use this as though it were the underlying
data and pass it as an argument. This allows the rail stages to keep
track of where their inputs are coming from.

To calculate magnitude error for extended sources, we need the
information about major and minor axes of each galaxy. Here we simply
generate random values

.. code:: ipython3

    samples_truth = flowEngine_truth.sample(n_samples, seed=0)
    
    import numpy as np
    
    samples_truth.data["major"] = np.abs(
        np.random.normal(loc=0.01, scale=0.1, size=n_samples)
    )  # add major and minor axes
    b_to_a = 1 - 0.5 * np.random.rand(n_samples)
    samples_truth.data["minor"] = samples_truth.data["major"] * b_to_a
    
    print(samples_truth())
    print("Data was written to ", samples_truth.path)



.. parsed-literal::

    Inserting handle into data store.  output_truth: inprogress_output_truth.pq, truth
           redshift          u          g          r          i          z  \
    0      0.890625  27.370831  26.712660  26.025223  25.327185  25.016500   
    1      1.978239  29.557047  28.361183  27.587227  27.238544  26.628105   
    2      0.974287  26.566013  25.937716  24.787411  23.872454  23.139563   
    3      1.317978  29.042736  28.274597  27.501110  26.648792  26.091452   
    4      1.386366  26.292624  25.774778  25.429960  24.806530  24.367950   
    ...         ...        ...        ...        ...        ...        ...   
    99995  2.147172  26.550978  26.349937  26.135286  26.082020  25.911032   
    99996  1.457508  27.362209  27.036276  26.823141  26.420132  26.110037   
    99997  1.372993  27.736042  27.271955  26.887583  26.416138  26.043432   
    99998  0.855022  28.044554  27.327116  26.599014  25.862329  25.592169   
    99999  1.723768  27.049067  26.526747  26.094597  25.642973  25.197958   
    
                   y     major     minor  
    0      24.926819  0.003319  0.002869  
    1      26.248560  0.008733  0.007945  
    2      22.832047  0.103938  0.052162  
    3      25.346504  0.147522  0.143359  
    4      23.700008  0.010929  0.009473  
    ...          ...       ...       ...  
    99995  25.558136  0.086491  0.071701  
    99996  25.524906  0.044537  0.022302  
    99997  25.456163  0.073146  0.047825  
    99998  25.506388  0.100551  0.094662  
    99999  24.900501  0.059611  0.049181  
    
    [100000 rows x 9 columns]
    Data was written to  output_truth.pq


LSSTErrorModel
~~~~~~~~~~~~~~

Now, we will demonstrate the ``LSSTErrorModel``, which adds photometric
errors using a model similar to the model from `Ivezic et
al. 2019 <https://arxiv.org/abs/0805.2366>`__ (specifically, it uses the
model from this paper, without making the high SNR assumption. To
restore this assumption and therefore use the exact model from the
paper, set ``highSNR=True``.)

Let’s create an error model with the default settings for point sources:

.. code:: ipython3

    errorModel = LSSTErrorModel.make_stage(name="error_model")


For extended sources:

.. code:: ipython3

    errorModel_auto = LSSTErrorModel.make_stage(
        name="error_model_auto", extendedType="auto"
    )


.. code:: ipython3

    errorModel_gaap = LSSTErrorModel.make_stage(
        name="error_model_gaap", extendedType="gaap"
    )


Now let’s add this error model as a degrader and draw some samples with
photometric errors.

.. code:: ipython3

    samples_w_errs = errorModel(samples_truth)
    samples_w_errs()



.. parsed-literal::

    Inserting handle into data store.  output_error_model: inprogress_output_error_model.pq, error_model




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>redshift</th>
          <th>u</th>
          <th>u_err</th>
          <th>g</th>
          <th>g_err</th>
          <th>r</th>
          <th>r_err</th>
          <th>i</th>
          <th>i_err</th>
          <th>z</th>
          <th>z_err</th>
          <th>y</th>
          <th>y_err</th>
          <th>major</th>
          <th>minor</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>0.890625</td>
          <td>inf</td>
          <td>inf</td>
          <td>26.718708</td>
          <td>0.167033</td>
          <td>26.026499</td>
          <td>0.080712</td>
          <td>25.425660</td>
          <td>0.077412</td>
          <td>25.006561</td>
          <td>0.102029</td>
          <td>24.843894</td>
          <td>0.195623</td>
          <td>0.003319</td>
          <td>0.002869</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1.978239</td>
          <td>27.943050</td>
          <td>1.023356</td>
          <td>30.302692</td>
          <td>1.851632</td>
          <td>27.501921</td>
          <td>0.284277</td>
          <td>27.607379</td>
          <td>0.475130</td>
          <td>27.355511</td>
          <td>0.670794</td>
          <td>26.381512</td>
          <td>0.645908</td>
          <td>0.008733</td>
          <td>0.007945</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.974287</td>
          <td>26.574780</td>
          <td>0.399468</td>
          <td>26.012684</td>
          <td>0.090606</td>
          <td>24.717912</td>
          <td>0.025396</td>
          <td>23.877734</td>
          <td>0.019853</td>
          <td>23.119759</td>
          <td>0.019447</td>
          <td>22.801512</td>
          <td>0.032727</td>
          <td>0.103938</td>
          <td>0.052162</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1.317978</td>
          <td>26.359988</td>
          <td>0.337857</td>
          <td>29.656087</td>
          <td>1.354836</td>
          <td>27.166456</td>
          <td>0.215740</td>
          <td>26.478707</td>
          <td>0.192786</td>
          <td>26.229318</td>
          <td>0.287486</td>
          <td>25.049417</td>
          <td>0.232248</td>
          <td>0.147522</td>
          <td>0.143359</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1.386366</td>
          <td>26.292190</td>
          <td>0.320171</td>
          <td>25.724566</td>
          <td>0.070293</td>
          <td>25.425985</td>
          <td>0.047402</td>
          <td>24.820055</td>
          <td>0.045247</td>
          <td>24.430537</td>
          <td>0.061383</td>
          <td>23.710705</td>
          <td>0.073242</td>
          <td>0.010929</td>
          <td>0.009473</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>99995</th>
          <td>2.147172</td>
          <td>26.304160</td>
          <td>0.323235</td>
          <td>26.387126</td>
          <td>0.125622</td>
          <td>26.251313</td>
          <td>0.098362</td>
          <td>26.305149</td>
          <td>0.166414</td>
          <td>26.390512</td>
          <td>0.327143</td>
          <td>25.122748</td>
          <td>0.246743</td>
          <td>0.086491</td>
          <td>0.071701</td>
        </tr>
        <tr>
          <th>99996</th>
          <td>1.457508</td>
          <td>27.420754</td>
          <td>0.735373</td>
          <td>27.448872</td>
          <td>0.305895</td>
          <td>26.644753</td>
          <td>0.138515</td>
          <td>26.278810</td>
          <td>0.162717</td>
          <td>26.271343</td>
          <td>0.297400</td>
          <td>25.597984</td>
          <td>0.361534</td>
          <td>0.044537</td>
          <td>0.022302</td>
        </tr>
        <tr>
          <th>99997</th>
          <td>1.372993</td>
          <td>27.062539</td>
          <td>0.573924</td>
          <td>27.023236</td>
          <td>0.215915</td>
          <td>26.669676</td>
          <td>0.141523</td>
          <td>27.048316</td>
          <td>0.308119</td>
          <td>25.826831</td>
          <td>0.206367</td>
          <td>25.294691</td>
          <td>0.283931</td>
          <td>0.073146</td>
          <td>0.047825</td>
        </tr>
        <tr>
          <th>99998</th>
          <td>0.855022</td>
          <td>27.008954</td>
          <td>0.552264</td>
          <td>28.709292</td>
          <td>0.773154</td>
          <td>26.628032</td>
          <td>0.136531</td>
          <td>25.895944</td>
          <td>0.116957</td>
          <td>25.973716</td>
          <td>0.233221</td>
          <td>25.182666</td>
          <td>0.259180</td>
          <td>0.100551</td>
          <td>0.094662</td>
        </tr>
        <tr>
          <th>99999</th>
          <td>1.723768</td>
          <td>26.471477</td>
          <td>0.368751</td>
          <td>26.433668</td>
          <td>0.130785</td>
          <td>26.022320</td>
          <td>0.080415</td>
          <td>25.687984</td>
          <td>0.097523</td>
          <td>25.109021</td>
          <td>0.111585</td>
          <td>24.826994</td>
          <td>0.192859</td>
          <td>0.059611</td>
          <td>0.049181</td>
        </tr>
      </tbody>
    </table>
    <p>100000 rows × 15 columns</p>
    </div>



.. code:: ipython3

    samples_w_errs_gaap = errorModel_gaap(samples_truth)
    samples_w_errs_gaap.data



.. parsed-literal::

    Inserting handle into data store.  output_error_model_gaap: inprogress_output_error_model_gaap.pq, error_model_gaap




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>redshift</th>
          <th>u</th>
          <th>u_err</th>
          <th>g</th>
          <th>g_err</th>
          <th>r</th>
          <th>r_err</th>
          <th>i</th>
          <th>i_err</th>
          <th>z</th>
          <th>z_err</th>
          <th>y</th>
          <th>y_err</th>
          <th>major</th>
          <th>minor</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>0.890625</td>
          <td>28.554000</td>
          <td>1.534254</td>
          <td>26.675370</td>
          <td>0.184837</td>
          <td>25.923431</td>
          <td>0.086688</td>
          <td>25.452865</td>
          <td>0.093891</td>
          <td>24.828337</td>
          <td>0.102534</td>
          <td>25.317431</td>
          <td>0.336633</td>
          <td>0.003319</td>
          <td>0.002869</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1.978239</td>
          <td>26.975825</td>
          <td>0.593424</td>
          <td>inf</td>
          <td>inf</td>
          <td>27.905141</td>
          <td>0.450204</td>
          <td>27.765490</td>
          <td>0.612330</td>
          <td>26.259063</td>
          <td>0.340998</td>
          <td>25.917548</td>
          <td>0.531587</td>
          <td>0.008733</td>
          <td>0.007945</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.974287</td>
          <td>25.920941</td>
          <td>0.268602</td>
          <td>25.801897</td>
          <td>0.088662</td>
          <td>24.835418</td>
          <td>0.033849</td>
          <td>23.860177</td>
          <td>0.023602</td>
          <td>23.100019</td>
          <td>0.022891</td>
          <td>22.822829</td>
          <td>0.040402</td>
          <td>0.103938</td>
          <td>0.052162</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1.317978</td>
          <td>inf</td>
          <td>inf</td>
          <td>28.931234</td>
          <td>1.027634</td>
          <td>27.105850</td>
          <td>0.254474</td>
          <td>26.202570</td>
          <td>0.191887</td>
          <td>26.288140</td>
          <td>0.370335</td>
          <td>25.678966</td>
          <td>0.472267</td>
          <td>0.147522</td>
          <td>0.143359</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1.386366</td>
          <td>26.153125</td>
          <td>0.318858</td>
          <td>25.685224</td>
          <td>0.078434</td>
          <td>25.393654</td>
          <td>0.054272</td>
          <td>24.800478</td>
          <td>0.052762</td>
          <td>24.284126</td>
          <td>0.063490</td>
          <td>23.672517</td>
          <td>0.083796</td>
          <td>0.010929</td>
          <td>0.009473</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>99995</th>
          <td>2.147172</td>
          <td>26.167467</td>
          <td>0.327153</td>
          <td>26.347710</td>
          <td>0.142379</td>
          <td>26.065865</td>
          <td>0.100322</td>
          <td>25.979444</td>
          <td>0.151570</td>
          <td>25.911028</td>
          <td>0.262721</td>
          <td>25.103307</td>
          <td>0.289279</td>
          <td>0.086491</td>
          <td>0.071701</td>
        </tr>
        <tr>
          <th>99996</th>
          <td>1.457508</td>
          <td>27.813643</td>
          <td>1.028005</td>
          <td>27.156219</td>
          <td>0.276379</td>
          <td>27.148132</td>
          <td>0.248411</td>
          <td>26.551353</td>
          <td>0.241310</td>
          <td>25.874890</td>
          <td>0.251092</td>
          <td>25.963629</td>
          <td>0.551494</td>
          <td>0.044537</td>
          <td>0.022302</td>
        </tr>
        <tr>
          <th>99997</th>
          <td>1.372993</td>
          <td>inf</td>
          <td>inf</td>
          <td>27.218373</td>
          <td>0.292722</td>
          <td>26.706596</td>
          <td>0.173100</td>
          <td>26.794369</td>
          <td>0.296627</td>
          <td>25.763780</td>
          <td>0.230958</td>
          <td>25.977916</td>
          <td>0.561254</td>
          <td>0.073146</td>
          <td>0.047825</td>
        </tr>
        <tr>
          <th>99998</th>
          <td>0.855022</td>
          <td>26.855252</td>
          <td>0.555110</td>
          <td>27.322181</td>
          <td>0.323029</td>
          <td>27.179647</td>
          <td>0.261476</td>
          <td>25.763226</td>
          <td>0.127132</td>
          <td>25.381536</td>
          <td>0.170536</td>
          <td>25.367620</td>
          <td>0.360465</td>
          <td>0.100551</td>
          <td>0.094662</td>
        </tr>
        <tr>
          <th>99999</th>
          <td>1.723768</td>
          <td>29.820693</td>
          <td>2.609516</td>
          <td>26.506914</td>
          <td>0.161613</td>
          <td>25.917243</td>
          <td>0.087090</td>
          <td>25.510288</td>
          <td>0.099771</td>
          <td>25.167545</td>
          <td>0.139051</td>
          <td>24.651807</td>
          <td>0.197216</td>
          <td>0.059611</td>
          <td>0.049181</td>
        </tr>
      </tbody>
    </table>
    <p>100000 rows × 15 columns</p>
    </div>



.. code:: ipython3

    samples_w_errs_auto = errorModel_auto(samples_truth)
    samples_w_errs_auto.data



.. parsed-literal::

    Inserting handle into data store.  output_error_model_auto: inprogress_output_error_model_auto.pq, error_model_auto




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>redshift</th>
          <th>u</th>
          <th>u_err</th>
          <th>g</th>
          <th>g_err</th>
          <th>r</th>
          <th>r_err</th>
          <th>i</th>
          <th>i_err</th>
          <th>z</th>
          <th>z_err</th>
          <th>y</th>
          <th>y_err</th>
          <th>major</th>
          <th>minor</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>0.890625</td>
          <td>inf</td>
          <td>inf</td>
          <td>26.491425</td>
          <td>0.137487</td>
          <td>26.049981</td>
          <td>0.082412</td>
          <td>25.341419</td>
          <td>0.071867</td>
          <td>24.972257</td>
          <td>0.099022</td>
          <td>24.814014</td>
          <td>0.190785</td>
          <td>0.003319</td>
          <td>0.002869</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1.978239</td>
          <td>26.733525</td>
          <td>0.451027</td>
          <td>inf</td>
          <td>inf</td>
          <td>27.371777</td>
          <td>0.255902</td>
          <td>27.293639</td>
          <td>0.374363</td>
          <td>26.473839</td>
          <td>0.349723</td>
          <td>27.156050</td>
          <td>1.062373</td>
          <td>0.008733</td>
          <td>0.007945</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.974287</td>
          <td>26.402077</td>
          <td>0.367956</td>
          <td>26.020417</td>
          <td>0.098027</td>
          <td>24.768283</td>
          <td>0.028798</td>
          <td>23.888092</td>
          <td>0.021776</td>
          <td>23.110251</td>
          <td>0.020891</td>
          <td>22.867387</td>
          <td>0.037786</td>
          <td>0.103938</td>
          <td>0.052162</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1.317978</td>
          <td>inf</td>
          <td>inf</td>
          <td>28.109447</td>
          <td>0.599705</td>
          <td>27.464889</td>
          <td>0.338744</td>
          <td>26.297305</td>
          <td>0.207050</td>
          <td>26.245778</td>
          <td>0.357131</td>
          <td>25.275646</td>
          <td>0.345362</td>
          <td>0.147522</td>
          <td>0.143359</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1.386366</td>
          <td>27.774656</td>
          <td>0.924397</td>
          <td>25.933171</td>
          <td>0.084594</td>
          <td>25.371536</td>
          <td>0.045231</td>
          <td>24.840805</td>
          <td>0.046158</td>
          <td>24.408723</td>
          <td>0.060293</td>
          <td>23.567582</td>
          <td>0.064622</td>
          <td>0.010929</td>
          <td>0.009473</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>99995</th>
          <td>2.147172</td>
          <td>27.118241</td>
          <td>0.624425</td>
          <td>26.507356</td>
          <td>0.149109</td>
          <td>26.262322</td>
          <td>0.107420</td>
          <td>26.154431</td>
          <td>0.158512</td>
          <td>25.692290</td>
          <td>0.198711</td>
          <td>25.675360</td>
          <td>0.412459</td>
          <td>0.086491</td>
          <td>0.071701</td>
        </tr>
        <tr>
          <th>99996</th>
          <td>1.457508</td>
          <td>27.173952</td>
          <td>0.626715</td>
          <td>26.663443</td>
          <td>0.161578</td>
          <td>26.767478</td>
          <td>0.156421</td>
          <td>26.449377</td>
          <td>0.191223</td>
          <td>25.749941</td>
          <td>0.196534</td>
          <td>25.928837</td>
          <td>0.472665</td>
          <td>0.044537</td>
          <td>0.022302</td>
        </tr>
        <tr>
          <th>99997</th>
          <td>1.372993</td>
          <td>27.589858</td>
          <td>0.842767</td>
          <td>27.267004</td>
          <td>0.274695</td>
          <td>27.044161</td>
          <td>0.204045</td>
          <td>26.061755</td>
          <td>0.142009</td>
          <td>25.963012</td>
          <td>0.242032</td>
          <td>25.198221</td>
          <td>0.275144</td>
          <td>0.073146</td>
          <td>0.047825</td>
        </tr>
        <tr>
          <th>99998</th>
          <td>0.855022</td>
          <td>26.830237</td>
          <td>0.519120</td>
          <td>27.051121</td>
          <td>0.243024</td>
          <td>26.525297</td>
          <td>0.139844</td>
          <td>25.815929</td>
          <td>0.122830</td>
          <td>25.259339</td>
          <td>0.142327</td>
          <td>26.794625</td>
          <td>0.926018</td>
          <td>0.100551</td>
          <td>0.094662</td>
        </tr>
        <tr>
          <th>99999</th>
          <td>1.723768</td>
          <td>27.697044</td>
          <td>0.897313</td>
          <td>26.714759</td>
          <td>0.172031</td>
          <td>26.143914</td>
          <td>0.093065</td>
          <td>25.765362</td>
          <td>0.108686</td>
          <td>25.376838</td>
          <td>0.146218</td>
          <td>24.761587</td>
          <td>0.189723</td>
          <td>0.059611</td>
          <td>0.049181</td>
        </tr>
      </tbody>
    </table>
    <p>100000 rows × 15 columns</p>
    </div>



Notice some of the magnitudes are inf’s. These are non-detections
(i.e. the noisy flux was negative). You can change the nSigma limit for
non-detections by setting ``sigLim=...``. For example, if ``sigLim=5``,
then all fluxes with ``SNR<5`` are flagged as non-detections.

Let’s plot the error as a function of magnitude

.. code:: ipython3

    %matplotlib inline
    
    fig, axes_ = plt.subplots(ncols=3, nrows=2, figsize=(15, 9), dpi=100)
    axes = axes_.reshape(-1)
    for i, band in enumerate("ugrizy"):
        ax = axes[i]
        # pull out the magnitudes and errors
        mags = samples_w_errs.data[band].to_numpy()
        errs = samples_w_errs.data[band + "_err"].to_numpy()
        
        # sort them by magnitude
        mags, errs = mags[mags.argsort()], errs[mags.argsort()]
        
        # plot errs vs mags
        #ax.plot(mags, errs, label=band) 
        
        #plt.plot(mags, errs, c='C'+str(i))
        ax.scatter(samples_w_errs_gaap.data[band].to_numpy(),
                samples_w_errs_gaap.data[band + "_err"].to_numpy(),
                    s=5, marker='.', color='C0', alpha=0.8, label='GAAP')
        
        ax.plot(mags, errs, color='C3', label='Point source')
        
        
        ax.legend()
        ax.set_xlim(18, 31)
        ax.set_ylim(-0.1, 3.5)
        ax.set(xlabel=band+" Band Magnitude (AB)", ylabel="Error (mags)")




.. image:: ../../../docs/rendered/creation_examples/photometric_realization_demo_files/../../../docs/rendered/creation_examples/photometric_realization_demo_24_0.png


.. code:: ipython3

    %matplotlib inline
    
    fig, axes_ = plt.subplots(ncols=3, nrows=2, figsize=(15, 9), dpi=100)
    axes = axes_.reshape(-1)
    for i, band in enumerate("ugrizy"):
        ax = axes[i]
        # pull out the magnitudes and errors
        mags = samples_w_errs.data[band].to_numpy()
        errs = samples_w_errs.data[band + "_err"].to_numpy()
        
        # sort them by magnitude
        mags, errs = mags[mags.argsort()], errs[mags.argsort()]
        
        # plot errs vs mags
        #ax.plot(mags, errs, label=band) 
        
        #plt.plot(mags, errs, c='C'+str(i))
        ax.scatter(samples_w_errs_auto.data[band].to_numpy(),
                samples_w_errs_auto.data[band + "_err"].to_numpy(),
                    s=5, marker='.', color='C0', alpha=0.8, label='AUTO')
        
        ax.plot(mags, errs, color='C3', label='Point source')
        
        
        ax.legend()
        ax.set_xlim(18, 31)
        ax.set_ylim(-0.1, 3.5)
        ax.set(xlabel=band+" Band Magnitude (AB)", ylabel="Error (mags)")




.. image:: ../../../docs/rendered/creation_examples/photometric_realization_demo_files/../../../docs/rendered/creation_examples/photometric_realization_demo_25_0.png


You can see that the photometric error increases as magnitude gets
dimmer, just like you would expect, and that the extended source errors
are greater than the point source errors. The extended source errors are
also scattered, because the galaxies have random sizes.

Also, you can find the GAaP and AUTO magnitude error are scattered due
to variable galaxy sizes. Also, you can find that there are gaps between
GAAP magnitude error and point souce magnitude error, this is because
the additional factors due to aperture sizes have a minimum value of
:math:`\sqrt{(\sigma^2+A_{\mathrm{min}})/\sigma^2}`, where
:math:`\sigma` is the width of the beam, :math:`A_{\min}` is an offset
of the aperture sizes (taken to be 0.7 arcmin here).

You can also see that there are *very* faint galaxies in this sample.
That’s because, by default, the error model returns magnitudes for all
positive fluxes. If you want these galaxies flagged as non-detections
instead, you can set e.g. ``sigLim=5``, and everything with ``SNR<5``
will be flagged as a non-detection.
