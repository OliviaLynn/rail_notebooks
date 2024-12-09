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

    <pzflow.flow.Flow at 0x7f680626aaa0>



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
          <td>27.790400</td>
          <td>0.932801</td>
          <td>26.639742</td>
          <td>0.156147</td>
          <td>25.921473</td>
          <td>0.073561</td>
          <td>25.245143</td>
          <td>0.065985</td>
          <td>25.275468</td>
          <td>0.128953</td>
          <td>24.956273</td>
          <td>0.214942</td>
          <td>0.003319</td>
          <td>0.002869</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1.978239</td>
          <td>inf</td>
          <td>inf</td>
          <td>28.818302</td>
          <td>0.830031</td>
          <td>27.203228</td>
          <td>0.222451</td>
          <td>26.759548</td>
          <td>0.243644</td>
          <td>27.141637</td>
          <td>0.577447</td>
          <td>26.151159</td>
          <td>0.548639</td>
          <td>0.008733</td>
          <td>0.007945</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.974287</td>
          <td>26.659164</td>
          <td>0.426108</td>
          <td>26.070172</td>
          <td>0.095293</td>
          <td>24.745387</td>
          <td>0.026010</td>
          <td>23.852435</td>
          <td>0.019433</td>
          <td>23.112955</td>
          <td>0.019336</td>
          <td>22.855325</td>
          <td>0.034317</td>
          <td>0.103938</td>
          <td>0.052162</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1.317978</td>
          <td>29.720613</td>
          <td>2.391656</td>
          <td>28.416689</td>
          <td>0.633993</td>
          <td>27.467076</td>
          <td>0.276355</td>
          <td>26.507195</td>
          <td>0.197465</td>
          <td>26.094089</td>
          <td>0.257525</td>
          <td>25.864877</td>
          <td>0.443961</td>
          <td>0.147522</td>
          <td>0.143359</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1.386366</td>
          <td>25.926909</td>
          <td>0.238096</td>
          <td>25.860569</td>
          <td>0.079257</td>
          <td>25.470441</td>
          <td>0.049311</td>
          <td>24.866885</td>
          <td>0.047168</td>
          <td>24.374602</td>
          <td>0.058411</td>
          <td>23.539310</td>
          <td>0.062929</td>
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
          <td>26.233608</td>
          <td>0.305539</td>
          <td>26.480652</td>
          <td>0.136200</td>
          <td>26.066592</td>
          <td>0.083617</td>
          <td>25.785051</td>
          <td>0.106174</td>
          <td>25.901094</td>
          <td>0.219573</td>
          <td>25.470395</td>
          <td>0.326917</td>
          <td>0.086491</td>
          <td>0.071701</td>
        </tr>
        <tr>
          <th>99996</th>
          <td>1.457508</td>
          <td>26.559297</td>
          <td>0.394733</td>
          <td>27.078520</td>
          <td>0.226077</td>
          <td>27.293689</td>
          <td>0.239767</td>
          <td>26.815084</td>
          <td>0.255026</td>
          <td>25.675790</td>
          <td>0.181717</td>
          <td>25.145769</td>
          <td>0.251457</td>
          <td>0.044537</td>
          <td>0.022302</td>
        </tr>
        <tr>
          <th>99997</th>
          <td>1.372993</td>
          <td>27.494689</td>
          <td>0.772354</td>
          <td>27.307764</td>
          <td>0.272949</td>
          <td>26.818906</td>
          <td>0.160854</td>
          <td>26.189958</td>
          <td>0.150802</td>
          <td>26.013009</td>
          <td>0.240918</td>
          <td>25.592330</td>
          <td>0.359937</td>
          <td>0.073146</td>
          <td>0.047825</td>
        </tr>
        <tr>
          <th>99998</th>
          <td>0.855022</td>
          <td>inf</td>
          <td>inf</td>
          <td>27.199786</td>
          <td>0.249889</td>
          <td>26.408420</td>
          <td>0.112844</td>
          <td>25.717657</td>
          <td>0.100093</td>
          <td>25.740112</td>
          <td>0.191865</td>
          <td>25.375039</td>
          <td>0.302937</td>
          <td>0.100551</td>
          <td>0.094662</td>
        </tr>
        <tr>
          <th>99999</th>
          <td>1.723768</td>
          <td>27.597848</td>
          <td>0.826055</td>
          <td>26.423066</td>
          <td>0.129591</td>
          <td>25.997466</td>
          <td>0.078670</td>
          <td>25.567466</td>
          <td>0.087724</td>
          <td>25.148255</td>
          <td>0.115466</td>
          <td>24.766182</td>
          <td>0.183207</td>
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
          <td>28.898760</td>
          <td>1.804574</td>
          <td>26.914059</td>
          <td>0.225742</td>
          <td>25.913165</td>
          <td>0.085908</td>
          <td>25.376738</td>
          <td>0.087815</td>
          <td>25.196445</td>
          <td>0.141162</td>
          <td>24.892021</td>
          <td>0.238585</td>
          <td>0.003319</td>
          <td>0.002869</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1.978239</td>
          <td>inf</td>
          <td>inf</td>
          <td>27.610314</td>
          <td>0.394817</td>
          <td>27.335344</td>
          <td>0.288312</td>
          <td>28.186665</td>
          <td>0.814091</td>
          <td>26.133403</td>
          <td>0.308560</td>
          <td>26.822635</td>
          <td>0.974487</td>
          <td>0.008733</td>
          <td>0.007945</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.974287</td>
          <td>26.061762</td>
          <td>0.300973</td>
          <td>25.940279</td>
          <td>0.100093</td>
          <td>24.785597</td>
          <td>0.032397</td>
          <td>23.884056</td>
          <td>0.024094</td>
          <td>23.168900</td>
          <td>0.024290</td>
          <td>22.834271</td>
          <td>0.040814</td>
          <td>0.103938</td>
          <td>0.052162</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1.317978</td>
          <td>inf</td>
          <td>inf</td>
          <td>28.937444</td>
          <td>1.031437</td>
          <td>27.074423</td>
          <td>0.247990</td>
          <td>26.145892</td>
          <td>0.182920</td>
          <td>25.749045</td>
          <td>0.240124</td>
          <td>25.641978</td>
          <td>0.459371</td>
          <td>0.147522</td>
          <td>0.143359</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1.386366</td>
          <td>26.948589</td>
          <td>0.582094</td>
          <td>25.703665</td>
          <td>0.079718</td>
          <td>25.412306</td>
          <td>0.055178</td>
          <td>24.852472</td>
          <td>0.055253</td>
          <td>24.405058</td>
          <td>0.070664</td>
          <td>23.670806</td>
          <td>0.083670</td>
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
          <td>26.418638</td>
          <td>0.398136</td>
          <td>26.283928</td>
          <td>0.134767</td>
          <td>26.266599</td>
          <td>0.119525</td>
          <td>26.186460</td>
          <td>0.180819</td>
          <td>26.175603</td>
          <td>0.325207</td>
          <td>26.090137</td>
          <td>0.612060</td>
          <td>0.086491</td>
          <td>0.071701</td>
        </tr>
        <tr>
          <th>99996</th>
          <td>1.457508</td>
          <td>26.547103</td>
          <td>0.434304</td>
          <td>27.238025</td>
          <td>0.295280</td>
          <td>26.571640</td>
          <td>0.152983</td>
          <td>26.337411</td>
          <td>0.201956</td>
          <td>25.632621</td>
          <td>0.205365</td>
          <td>26.230057</td>
          <td>0.665508</td>
          <td>0.044537</td>
          <td>0.022302</td>
        </tr>
        <tr>
          <th>99997</th>
          <td>1.372993</td>
          <td>28.684202</td>
          <td>1.642774</td>
          <td>27.452290</td>
          <td>0.352639</td>
          <td>27.035384</td>
          <td>0.228161</td>
          <td>26.137240</td>
          <td>0.172007</td>
          <td>25.831258</td>
          <td>0.244199</td>
          <td>26.476260</td>
          <td>0.790445</td>
          <td>0.073146</td>
          <td>0.047825</td>
        </tr>
        <tr>
          <th>99998</th>
          <td>0.855022</td>
          <td>30.090857</td>
          <td>2.875126</td>
          <td>27.147142</td>
          <td>0.280663</td>
          <td>26.559732</td>
          <td>0.155508</td>
          <td>25.851374</td>
          <td>0.137199</td>
          <td>25.765512</td>
          <td>0.235381</td>
          <td>25.454775</td>
          <td>0.385786</td>
          <td>0.100551</td>
          <td>0.094662</td>
        </tr>
        <tr>
          <th>99999</th>
          <td>1.723768</td>
          <td>inf</td>
          <td>inf</td>
          <td>27.113051</td>
          <td>0.268154</td>
          <td>25.920731</td>
          <td>0.087357</td>
          <td>25.828274</td>
          <td>0.131596</td>
          <td>25.341183</td>
          <td>0.161392</td>
          <td>24.491062</td>
          <td>0.172156</td>
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
          <td>26.641486</td>
          <td>0.420443</td>
          <td>26.775048</td>
          <td>0.175246</td>
          <td>25.877220</td>
          <td>0.070746</td>
          <td>25.426043</td>
          <td>0.077449</td>
          <td>25.073912</td>
          <td>0.108232</td>
          <td>24.925309</td>
          <td>0.209480</td>
          <td>0.003319</td>
          <td>0.002869</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1.978239</td>
          <td>27.939238</td>
          <td>1.021486</td>
          <td>inf</td>
          <td>inf</td>
          <td>27.626253</td>
          <td>0.314451</td>
          <td>26.441678</td>
          <td>0.187035</td>
          <td>28.666174</td>
          <td>1.466769</td>
          <td>25.771976</td>
          <td>0.414029</td>
          <td>0.008733</td>
          <td>0.007945</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.974287</td>
          <td>27.210663</td>
          <td>0.667177</td>
          <td>25.918553</td>
          <td>0.089651</td>
          <td>24.802516</td>
          <td>0.029675</td>
          <td>23.885572</td>
          <td>0.021729</td>
          <td>23.149822</td>
          <td>0.021608</td>
          <td>22.823189</td>
          <td>0.036338</td>
          <td>0.103938</td>
          <td>0.052162</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1.317978</td>
          <td>inf</td>
          <td>inf</td>
          <td>29.308805</td>
          <td>1.271183</td>
          <td>27.416588</td>
          <td>0.326018</td>
          <td>27.165439</td>
          <td>0.416078</td>
          <td>25.662316</td>
          <td>0.222728</td>
          <td>25.498363</td>
          <td>0.410658</td>
          <td>0.147522</td>
          <td>0.143359</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1.386366</td>
          <td>26.069294</td>
          <td>0.267807</td>
          <td>25.845384</td>
          <td>0.078300</td>
          <td>25.416326</td>
          <td>0.047065</td>
          <td>24.796279</td>
          <td>0.044369</td>
          <td>24.465424</td>
          <td>0.063402</td>
          <td>23.680717</td>
          <td>0.071431</td>
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
          <td>27.202137</td>
          <td>0.661884</td>
          <td>26.257397</td>
          <td>0.120172</td>
          <td>26.009227</td>
          <td>0.086033</td>
          <td>26.353747</td>
          <td>0.187771</td>
          <td>26.076332</td>
          <td>0.273047</td>
          <td>25.907498</td>
          <td>0.491285</td>
          <td>0.086491</td>
          <td>0.071701</td>
        </tr>
        <tr>
          <th>99996</th>
          <td>1.457508</td>
          <td>28.099418</td>
          <td>1.129826</td>
          <td>27.255248</td>
          <td>0.265012</td>
          <td>26.740195</td>
          <td>0.152808</td>
          <td>26.182195</td>
          <td>0.152350</td>
          <td>26.016219</td>
          <td>0.245313</td>
          <td>25.469284</td>
          <td>0.331715</td>
          <td>0.044537</td>
          <td>0.022302</td>
        </tr>
        <tr>
          <th>99997</th>
          <td>1.372993</td>
          <td>27.523608</td>
          <td>0.807526</td>
          <td>26.857585</td>
          <td>0.195760</td>
          <td>26.903585</td>
          <td>0.181251</td>
          <td>26.243699</td>
          <td>0.165970</td>
          <td>26.108723</td>
          <td>0.272719</td>
          <td>25.644473</td>
          <td>0.392054</td>
          <td>0.073146</td>
          <td>0.047825</td>
        </tr>
        <tr>
          <th>99998</th>
          <td>0.855022</td>
          <td>inf</td>
          <td>inf</td>
          <td>27.035001</td>
          <td>0.239816</td>
          <td>26.492163</td>
          <td>0.135904</td>
          <td>25.844593</td>
          <td>0.125922</td>
          <td>25.499408</td>
          <td>0.174763</td>
          <td>25.223880</td>
          <td>0.299027</td>
          <td>0.100551</td>
          <td>0.094662</td>
        </tr>
        <tr>
          <th>99999</th>
          <td>1.723768</td>
          <td>26.961490</td>
          <td>0.545744</td>
          <td>26.495552</td>
          <td>0.142628</td>
          <td>26.036214</td>
          <td>0.084652</td>
          <td>25.547488</td>
          <td>0.089793</td>
          <td>25.122309</td>
          <td>0.117322</td>
          <td>25.360692</td>
          <td>0.310707</td>
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
