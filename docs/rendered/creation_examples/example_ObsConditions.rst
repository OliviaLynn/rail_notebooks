Generate Spatially Varying Magnitude Errors According to Observing Conditions
=============================================================================

last run successfully: April 26, 2023

The ObsCondition degrader can be used to generate spatially-varying
photometric errors using input survey condition maps in ``healpix``
format, such as survey coadd depth, airmass, sky brightness etc. The
photometric error is computed by ``photerr.LsstErrorModel``, based on
the LSST Overview Paper: https://arxiv.org/abs/0805.2366.

The degrader assigns each object in the input catalogue with a pixel
within the survey footprint and computes the magnitude error (SNR) on
each pixel. The degrader takes the following arguments:

-  ``nside``: nside used for the HEALPIX maps.
-  ``mask``: Path to the mask covering the survey footprint in HEALPIX
   format. Notice that all negative values will be set to zero.
-  ``weight``: Path to the weights HEALPIX format, used to assign sample
   galaxies in pixels. Default is weight="", which uniform weighting.
-  ``tot_nVis_flag``: If ``nVisYr`` is provided in ``map_dict`` (see
   below), this flag indicates whether the map shows the total number of
   visits in nYrObs (``tot_nVis_flag=True``), or the average number of
   visits per year (``tot_nVis_flag=False``). The default is set to
   ``True``.
-  ``random_seed``: A random seed for reproducibility.
-  ``map_dict``: A dictionary that contains the paths to the survey
   condition maps in HEALPIX format. This dictionary uses the same
   arguments as LSSTErrorModel. The following arguements, if supplied,
   may contain either a single number (as in the case of
   LSSTErrorModel), or a path to the corresponding survey condition map
   in ``healpix`` format:``m5``, ``nVisYr``, ``airmass``, ``gamma``,
   ``msky``, ``theta``, ``km``, and ``tvis``. Notice that *except*
   ``airmass`` and ``tvis``, for all other arguements, numbers/paths for
   *specific bands* should be passed. Other ``LsstErrorModel``
   parameters can also be passed in this dictionary (e.g. a necessary
   one may be ``nYrObs`` for the survey condition maps; the default
   value is 10 years, although most may be interested in early data
   releases). If any arguement is not passed, the default value in
   https://arxiv.org/abs/0805.2366 is adopted. Example:

.. code:: json

   {
      "m5": {"u": "path", ...}, 
      "theta": {"u": "path", ...},
   }

Argument defaults are determined by the defaults of the
``LsstErrorModel`` in
`PhotErr <https://github.com/jfcrenshaw/photerr>`__.

In this quick notebook we’ll generate the photometric error based on the
DC2 Y5 LSST median :math:`5\sigma` depth in :math:`i`-band generated by
OpSim ``minion_1016`` database using the Rubin Observatory Metrics
Analysis Framework (MAF).

.. code:: ipython3

    import healpy as hp
    
    %matplotlib inline
    import numpy as np
    import matplotlib.pyplot as plt
    
    from astropy.io import fits
    import os
    
    import pandas as pd
    import tables_io

.. code:: ipython3

    import rail
    from rail.core.stage import RailStage
    from rail.core.utils import find_rail_file
    DS = RailStage.data_store
    DS.__class__.allow_overwrite = True

Let’s generate some fake data.

.. code:: ipython3

    # Fake data with same magnitude in each band
    i = np.ones(50_000)*23.
    u = np.full_like(i, 23.0, dtype=np.double)
    g = np.full_like(i, 23.0, dtype=np.double)
    r = np.full_like(i, 23.0, dtype=np.double)
    y = np.full_like(i, 23.0, dtype=np.double)
    z = np.full_like(i, 23.0, dtype=np.double)
    redshift = np.random.uniform(size=len(i)) * 2

.. code:: ipython3

    mockdict = {}
    for label, item in zip(['redshift','u', 'g','r','i', 'z','y'], [redshift,u,g,r,i,z,y]):
        mockdict[f'{label}'] = item

.. code:: ipython3

    data = pd.DataFrame(mockdict)
    data.head()




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
          <th>g</th>
          <th>r</th>
          <th>i</th>
          <th>z</th>
          <th>y</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>0.905692</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>0.008942</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.629562</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1.220909</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1.987787</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
          <td>23.0</td>
        </tr>
      </tbody>
    </table>
    </div>



Now let’s import the ObsCondition from rail.

.. code:: ipython3

    from rail.creation.degradation import observing_condition_degrader
    from rail.creation.degradation.observing_condition_degrader import ObsCondition

.. code:: ipython3

    # First, let's use default arguments:
    obs_cond_degrader = ObsCondition.make_stage()

.. code:: ipython3

    # You can see what arguments have been entered by printing the degrader:
    print(obs_cond_degrader)


.. parsed-literal::

    Loaded observing conditions from configuration file: 
    nside = 128, 
    mask file:  /opt/hostedtoolcache/Python/3.10.13/x64/lib/python3.10/site-packages/rail/creation/degradation/../../examples_data/creation_data/data/survey_conditions/DC2-mask-neg-nside-128.fits, 
    weight file:  /opt/hostedtoolcache/Python/3.10.13/x64/lib/python3.10/site-packages/rail/creation/degradation/../../examples_data/creation_data/data/survey_conditions/DC2-dr6-galcounts-i20-i25.3-nside-128.fits, 
    tot_nVis_flag = True, 
    random_seed = 42, 
    map_dict contains the following items: 
    {'m5': {'i': '/opt/hostedtoolcache/Python/3.10.13/x64/lib/python3.10/site-packages/rail/creation/degradation/../../examples_data/creation_data/data/survey_conditions/minion_1016_dc2_Median_fiveSigmaDepth_i_and_nightlt1825_HEAL.fits'}, 'nYrObs': 5.0}


Let’s run the code and see how long it takes:

.. code:: ipython3

    %%time
    data_degraded = obs_cond_degrader(data)


.. parsed-literal::

    Inserting handle into data store.  input: None, ObsCondition
    Assigning pixels.
    No ra, dec found in catalogue, randomly assign pixels with weights.
    Warning: objects found outside given mask, pixel assigned=-99. These objects will be assigned with defualt error from LSST error model!


.. parsed-literal::

    Inserting handle into data store.  output: inprogress_output.pq, ObsCondition
    CPU times: user 3.38 s, sys: 25.9 ms, total: 3.4 s
    Wall time: 3.4 s


.. code:: ipython3

    data_degraded.data.head()




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
          <th>ra</th>
          <th>decl</th>
          <th>pixel</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>0.905692</td>
          <td>22.992718</td>
          <td>0.019997</td>
          <td>23.008932</td>
          <td>0.008050</td>
          <td>22.992422</td>
          <td>0.007802</td>
          <td>23.001066</td>
          <td>0.014987</td>
          <td>22.963393</td>
          <td>0.016124</td>
          <td>22.967642</td>
          <td>0.035845</td>
          <td>61.171875</td>
          <td>-40.620185</td>
          <td>162135</td>
        </tr>
        <tr>
          <th>1</th>
          <td>0.008942</td>
          <td>22.971834</td>
          <td>0.019661</td>
          <td>23.010025</td>
          <td>0.008055</td>
          <td>23.020903</td>
          <td>0.007919</td>
          <td>22.990186</td>
          <td>0.013443</td>
          <td>22.983485</td>
          <td>0.016389</td>
          <td>23.018049</td>
          <td>0.037475</td>
          <td>63.632812</td>
          <td>-34.953865</td>
          <td>154458</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.629562</td>
          <td>23.038423</td>
          <td>0.020757</td>
          <td>22.994247</td>
          <td>0.007989</td>
          <td>23.010904</td>
          <td>0.007877</td>
          <td>22.984970</td>
          <td>0.014422</td>
          <td>23.013104</td>
          <td>0.016790</td>
          <td>23.013980</td>
          <td>0.037341</td>
          <td>52.795276</td>
          <td>-42.210370</td>
          <td>164170</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1.220909</td>
          <td>23.011367</td>
          <td>0.020303</td>
          <td>22.987832</td>
          <td>0.007962</td>
          <td>22.995811</td>
          <td>0.007816</td>
          <td>22.985820</td>
          <td>0.014616</td>
          <td>23.007550</td>
          <td>0.016714</td>
          <td>23.017984</td>
          <td>0.037473</td>
          <td>53.789062</td>
          <td>-39.450895</td>
          <td>160588</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1.987787</td>
          <td>23.004049</td>
          <td>0.020183</td>
          <td>23.008596</td>
          <td>0.008049</td>
          <td>22.992514</td>
          <td>0.007802</td>
          <td>23.000959</td>
          <td>0.014822</td>
          <td>23.004027</td>
          <td>0.016666</td>
          <td>22.964041</td>
          <td>0.035732</td>
          <td>69.609375</td>
          <td>-28.971532</td>
          <td>145763</td>
        </tr>
      </tbody>
    </table>
    </div>



We see that extra columns containing the magnitude errors: ``u_err``,
``g_err``\ … have been added to the catalogue. Notice that since we have
only provided the limiting magnitude for :math:`i`-band, the errors in
all other bands except :math:`i` are computed using the default
parameters in ``LsstErrorModel`` (see:
https://github.com/jfcrenshaw/photerr/blob/main/photerr/lsst.py).

The last column shows the pixel of the survey condition map that is
assigned to each object.

We can check if the spatial dependence has been implemented by looking
at the SNR at different area of the sky, and compare that with the
:math:`i`-band depth:

.. code:: ipython3

    mask = hp.read_map(find_rail_file("examples_data/creation_data/data/survey_conditions/DC2-mask-neg-nside-128.fits"))
    weight = hp.read_map(find_rail_file("examples_data/creation_data/data/survey_conditions/DC2-dr6-galcounts-i20-i25.3-nside-128.fits"))
    Med_5sd_i = hp.read_map(find_rail_file("examples_data/creation_data/data/survey_conditions/minion_1016_dc2_Median_fiveSigmaDepth_i_and_nightlt1825_HEAL.fits"))


.. code:: ipython3

    # Set negative values in mask to zero
    mask[mask<0]=0

.. code:: ipython3

    # Compute the average SNR in each pixel
    avg_SNR_i = np.zeros(len(mask))
    for pix, pix_cat in (data_degraded.data).groupby("pixel"):
        avg_SNR_i[pix] = np.mean((pix_cat["i"]/pix_cat["i_err"]).to_numpy())

.. code:: ipython3

    # View the healpix map
    
    fig,axarr=plt.subplots(1,3,figsize=[12,6])
    
    plt.sca(axarr[0])
    hp.gnomview(weight*mask/sum(weight), rot=(62, -36.5, 0), xsize=100,ysize=100, reso=16, title="weight",
               hold=True)
    plt.sca(axarr[1])
    hp.gnomview(Med_5sd_i*mask, rot=(62, -36.5, 0), xsize=100,ysize=100, reso=16, title="5sigmadepth i",
               hold=True)
    plt.sca(axarr[2])
    hp.gnomview(avg_SNR_i, rot=(62, -36.5, 0), xsize=100,ysize=100, reso=16, title="avg SNR i",
                min=1400, max=1750,
               hold=True)



.. image:: ../../../docs/rendered/creation_examples/example_ObsConditions_files/../../../docs/rendered/creation_examples/example_ObsConditions_22_0.png


Now if we want to change any of the default settings, we can supply them
in ``ObsCondition.make_stage()``. In this example, instead of supplying
the median :math:`5\sigma` depth in :math:`i`-band, we supply the median
airmass in :math:`i`-band. In this case, the :math:`i`-band limiting
magnitude ``m5`` will be computed explicitly (notice that if ``m5`` is
also supplied, then it will overwrite the explicitly computed ``m5``).

.. code:: ipython3

    airmass_degrader = ObsCondition.make_stage(
        map_dict={"airmass": find_rail_file("examples_data/creation_data/data/survey_conditions/minion_1016_dc2_Median_airmass_i_and_nightlt1825_HEAL.fits"),
                 "nYrObs": 5.0}
    )

.. code:: ipython3

    print(airmass_degrader)


.. parsed-literal::

    Loaded observing conditions from configuration file: 
    nside = 128, 
    mask file:  /opt/hostedtoolcache/Python/3.10.13/x64/lib/python3.10/site-packages/rail/creation/degradation/../../examples_data/creation_data/data/survey_conditions/DC2-mask-neg-nside-128.fits, 
    weight file:  /opt/hostedtoolcache/Python/3.10.13/x64/lib/python3.10/site-packages/rail/creation/degradation/../../examples_data/creation_data/data/survey_conditions/DC2-dr6-galcounts-i20-i25.3-nside-128.fits, 
    tot_nVis_flag = True, 
    random_seed = 42, 
    map_dict contains the following items: 
    {'airmass': '/opt/hostedtoolcache/Python/3.10.13/x64/lib/python3.10/site-packages/rail/examples_data/creation_data/data/survey_conditions/minion_1016_dc2_Median_airmass_i_and_nightlt1825_HEAL.fits', 'nYrObs': 5.0}


.. code:: ipython3

    data_degraded_airmass = airmass_degrader(data)


.. parsed-literal::

    Assigning pixels.
    No ra, dec found in catalogue, randomly assign pixels with weights.
    Warning: objects found outside given mask, pixel assigned=-99. These objects will be assigned with defualt error from LSST error model!


.. parsed-literal::

    Inserting handle into data store.  output: inprogress_output.pq, ObsCondition


.. code:: ipython3

    data_degraded_airmass.data.head()




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
          <th>ra</th>
          <th>decl</th>
          <th>pixel</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>0.905692</td>
          <td>22.992845</td>
          <td>0.019652</td>
          <td>23.008888</td>
          <td>0.008011</td>
          <td>22.992444</td>
          <td>0.007780</td>
          <td>23.000725</td>
          <td>0.010207</td>
          <td>22.963483</td>
          <td>0.016085</td>
          <td>22.967855</td>
          <td>0.035617</td>
          <td>61.171875</td>
          <td>-40.620185</td>
          <td>162135</td>
        </tr>
        <tr>
          <th>1</th>
          <td>0.008942</td>
          <td>22.972841</td>
          <td>0.018975</td>
          <td>23.009923</td>
          <td>0.007972</td>
          <td>23.020776</td>
          <td>0.007870</td>
          <td>22.992634</td>
          <td>0.010121</td>
          <td>22.983570</td>
          <td>0.016306</td>
          <td>23.017796</td>
          <td>0.036954</td>
          <td>63.632812</td>
          <td>-34.953865</td>
          <td>154458</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.629562</td>
          <td>23.037129</td>
          <td>0.020051</td>
          <td>22.994301</td>
          <td>0.007915</td>
          <td>23.010844</td>
          <td>0.007834</td>
          <td>22.989520</td>
          <td>0.010105</td>
          <td>23.013042</td>
          <td>0.016710</td>
          <td>23.013801</td>
          <td>0.036869</td>
          <td>52.795276</td>
          <td>-42.210370</td>
          <td>164170</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1.220909</td>
          <td>23.010959</td>
          <td>0.019578</td>
          <td>22.987953</td>
          <td>0.007883</td>
          <td>22.995835</td>
          <td>0.007770</td>
          <td>22.990243</td>
          <td>0.010105</td>
          <td>23.007511</td>
          <td>0.016629</td>
          <td>23.017736</td>
          <td>0.036960</td>
          <td>53.789062</td>
          <td>-39.450895</td>
          <td>160588</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1.987787</td>
          <td>23.003878</td>
          <td>0.019339</td>
          <td>23.008493</td>
          <td>0.007953</td>
          <td>22.992566</td>
          <td>0.007749</td>
          <td>23.000657</td>
          <td>0.010165</td>
          <td>23.004003</td>
          <td>0.016566</td>
          <td>22.964613</td>
          <td>0.035182</td>
          <td>69.609375</td>
          <td>-28.971532</td>
          <td>145763</td>
        </tr>
      </tbody>
    </table>
    </div>



Again, we can examine whether the spatial dependence is indeed applied.
Here, ``LsstErrorModel`` does not have band-dependent airmass, so it
affects all bands. The default airmass is :math:`X=1.2`, but the input
median airmass is more optimistic, thus reducing the magnitude errors.

.. code:: ipython3

    Med_airmass_i = hp.read_map(find_rail_file("examples_data/creation_data/data/survey_conditions/minion_1016_dc2_Median_airmass_i_and_nightlt1825_HEAL.fits"))

Compute the average SNR in each pixel for i and r bands:

.. code:: ipython3

    avg_SNR_i_airmass = np.zeros(len(mask))
    avg_SNR_r_airmass = np.zeros(len(mask))
    for pix, pix_cat in (data_degraded_airmass.data).groupby("pixel"):
        avg_SNR_i_airmass[pix] = np.mean((pix_cat["i"]/pix_cat["i_err"]).to_numpy())
        avg_SNR_r_airmass[pix] = np.mean((pix_cat["r"]/pix_cat["r_err"]).to_numpy())

View the healpix map:

.. code:: ipython3

    fig,axarr=plt.subplots(1,3,figsize=[12,6])
    
    plt.sca(axarr[0])
    hp.gnomview(Med_airmass_i*mask, rot=(62, -36.5, 0), xsize=100,ysize=100, reso=16, title="airmass i",
               hold=True)
    plt.sca(axarr[1])
    hp.gnomview(avg_SNR_i_airmass, rot=(62, -36.5, 0), xsize=100,ysize=100, reso=16, title="avg SNR i",
                min=2240, max=2280,
               hold=True)
    
    plt.sca(axarr[2])
    hp.gnomview(avg_SNR_r_airmass, rot=(62, -36.5, 0), xsize=100,ysize=100, reso=16, title="avg SNR r",
                min=2930, max=2970,
               hold=True)



.. image:: ../../../docs/rendered/creation_examples/example_ObsConditions_files/../../../docs/rendered/creation_examples/example_ObsConditions_33_0.png


In both cases, we see a negative correlation between the airmass and the
SNR in :math:`i` and :math:`r` bands, as expected.

