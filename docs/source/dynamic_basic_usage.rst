===========
Basic usage
===========


Step 0: Prepare the library spectra
===================================

Prepare your library spectra with correct format before beginning. The format requirements are the same as mentioned in Flash Entropy Search-Basic usage.
Library spectra should be represented as a list of dictionaries, where each dictionary corresponds to a spectrum. 
Below is the example:

.. code-block:: python

    import numpy as np
    spectra_1_for_library =  [{
        "id": "Demo spectrum 1",
        "precursor_mz": 150.0,
        "peaks": np.array([[100.0, 1.0], [101.0, 1.0], [103.0, 1.0]], dtype=np.float32),
    }, {
        "id": "Demo spectrum 2",
        "precursor_mz": 200.0,
        "peaks": np.array([[100.0, 1.0], [101.0, 1.0], [102.0, 1.0]], dtype=np.float32),
        "metadata": "ABC"
    }, {
        "id": "Demo spectrum 3",
        "precursor_mz": 250.0,
        "peaks": np.array([[200.0, 1.0], [101.0, 1.0], [202.0, 1.0]], dtype=np.float32),
        "XXX": "YYY",
    }, {
        "precursor_mz": 350.0,
        "peaks": np.array([[100.0, 1.0], [101.0, 1.0], [302.0, 1.0]], dtype=np.float32)
    }]
    
    spectra_2_for_library ... # Similar to spectra_1_for_library
    spectra_3_for_library ... # Similar to spectra_1_for_library

The keys ``precursor_mz`` and ``peaks`` are necessary for this format.
Other keys are optional and are considered as metadata of the spectrum, helping the identification.


Step 1: Building and updating the index
========================================

The search algorithm is based on a reference index, so the initial step is to construct the index.
The ``add_new_spectra()`` function of the ``DynamicEntropySearch`` class combines building and updating process together and takes your spectra library as an argument to construct index. 

The spectra in the spectra library should be cleaned using :ref:`clean_spectrum() <clean-spectrum-function>` in ``ms_entropy`` before passed into the ``add_new_spectrum()``.

.. code-block:: python

    from ms_entropy import clean_spectrum

    precursor_ions_removal_da = 1.6

    for spec in spectra_1_for_library:
        spec['peaks'] = clean_spectrum(
            peaks = spec['peaks'],
            max_mz = spec['precursor_mz'] - precursor_ions_removal_da
        )

    for spec in spectra_2_for_library:
        spec['peaks'] = clean_spectrum(
            peaks = spec['peaks'],
            max_mz = spec['precursor_mz'] - precursor_ions_removal_da
        )

    for spec in spectra_3_for_library:
        spec['peaks'] = clean_spectrum(
            peaks = spec['peaks'],
            max_mz = spec['precursor_mz'] - precursor_ions_removal_da
        )

After cleaning, spectra can be passed into ``add_new_spectra()`` as an argument.

.. code-block:: python

    from ms_entropy import DynamicEntropySearch
    entropy_search = DynamicEntropySearch(path_data = path_of_your_library)
    entropy_search.add_new_spectra(spectra_list = spectra_1_for_library)
    entropy_search.add_new_spectra(spectra_list = spectra_2_for_library)
    entropy_search.add_new_spectra(spectra_list = spectra_3_for_library)
    ......
    entropy_search.build_index()
    entropy_search.write()

.. note::
    ``add_new_spectra()`` can be performed several times if you have a lot of input lists (spectra_library).

    It is necessary to call ``build_index()`` and ``write()`` lastly after all adding operations. 

.. note::
    If you want to calculate unweighted entropy similarity instead of weighted entropy similarity, you can set the ``intensity_weight`` parameter to ``None`` when constructing the ``DynamicEntropySearch`` object. 
    For example:

    .. code-block:: python

        entropy_search=DynamicEntropySearch(path_data=path_of_your_library, intensity_weight=None)
        entropy_search.add_new_spectra(spectra_list=spectra_1_for_library)
        entropy_search.add_new_spectra(spectra_list=spectra_2_for_library)
        entropy_search.add_new_spectra(spectra_list=spectra_3_for_library)
        ......
        entropy_search.build_index()
        entropy_search.write()

.. note::
    If you only want to construct index for open search, you can set ``index_for_neutral_loss`` in ``add_new_spectra()`` and ``build_index()`` to ``False``. See details in **Useful functions**.


Step 2: Searching the library
=============================

After building and updating the index, you can search your query spectrum against the library spectra using the ``search_topn_matches()`` function. 
This function takes the query spectrum as input, returning the similarity score and metadata of top few matched spectra in the library.


The ``search_topn_matches()`` function accepts the following parameters:

- ``peaks``: The peaks of the query spectrum, which is a numpy array in format of ``[[mz, intensity], [mz, intensity], ...]``.

- ``precursor_mz``: The precursor m/z of the query spectrum.

- ``ms1_tolerance_in_da``: The mass tolerance to apply to the precursor m/z in Da, used only for identity search. Default is ``0.01``.

- ``ms2_tolerance_in_da``: The mass tolerance to apply to the fragment ions in Da. Default is ``0.02``.

- ``method``: The search method to employ. Available methods include ``identity``, ``open``, ``neutral_loss``, and ``hybrid``. A string is acceptable. The default value is ``open``.

- ``clean``: Whether to clean the query spectrum before searching. Default is ``True``.

- ``precursor_ions_removal_da``: The mass tolerance for removing the precursor ions in Da. Fragment ions with m/z larger than ``precursor_mz - precursor_ions_removal_da`` will be removed. Based on our tests, removing precursor ions can enhance search performance. Default is ``1.6``.

- ``noise_threshold``: The intensity threshold for removing noise peaks. Peaks with intensity smaller than ``noise_threshold * max(fragment ion's intensity)`` will be removed. Default is ``0.01``.

- ``min_ms2_difference_in_da``:  Minimum spacing allowed between MS/MS peaks during cleaning. Default is ``0.05`` Da.

- ``max_peak_num``: Maximum number of peaks to keep after cleaning. ``None`` keeps all peaks. Default is ``None``.

- ``topn``: Number of top-matching spectra to return. If ``None``, all spectra are returned. Default is ``3``.

- ``need_metadata``: If ``True`` (default), return the metadata dictionary for each matched spectrum. If ``False``, return `(global_index, similarity)` tuples instead. Default is ``True``.


``search_topn_matches()`` function can be used as follows:

.. code-block:: python

    entropy_similarity = entropy_search.search_topn_matches(
        precursor_mz = 150.0,
        peaks = np.array([[100.0, 1.0], [101.0, 1.0], [102.0, 1.0]], dtype=np.float32),
        method = "open"
    )# Other arguments can be set as you need.

The result returned by ``search_topn_matches()`` function includes metadata:

.. code-block:: python

    [{
    'id': 'Demo spectrum 3', 
    'precursor_mz': 250.0, 
    'peaks': array([[101.        ,   0.33333334], [200.        ,   0.33333334], [202.        ,   0.33333334]], dtype=float32), 
    'XXX': 'YYY', 
    'open_search_entropy_similarity': np.float32(0.99999994)
    }, {
    'id': 'Demo spectrum 2', 
    'precursor_mz': 200.0, 
    'peaks': array([[100.        ,   0.33333334], [101.        ,   0.33333334], [102.        ,   0.33333334]], dtype=float32), 
    'metadata': 'ABC', 
    'open_search_entropy_similarity': np.float32(0.3333333)
    }, {
    'precursor_mz': 350.0, 
    'peaks': array([[100.        ,   0.33333334], [101.        ,   0.33333334], [302.        ,   0.33333334]], dtype=float32), 
    'open_search_entropy_similarity': np.float32(0.3333333)}]

---------------------------------------------

Besides ``search_topn_matches()`` function, you can also use ``search()`` function. 
Taking the query spectrum as input, this function returns the similarity score of each spectrum in the library, in the same order as the whole spectral index.

The ``search()`` function accepts the following parameters:

- ``precursor_mz``: The precursor m/z of the query spectrum.

- ``peaks``: The peaks of the query spectrum, which is a numpy array in format of ``[[mz, intensity], [mz, intensity], ...]``.

- ``ms1_tolerance_in_da``: The mass tolerance to apply to the precursor m/z in Da, used only for identity search. Default is ``0.01``.

- ``ms2_tolerance_in_da``: The mass tolerance to apply to the fragment ions in Da. Default is ``0.02``.

- ``method``: The search method to employ. Available methods include ``identity``, ``open``, ``neutral_loss``, and ``hybrid``. You can use a string or a list/set of these four strings like ``{'identity', 'open'}``. Use ``all`` to apply all four methods. The default value is ``all``.

- ``precursor_ions_removal_da``: The mass tolerance for removing the precursor ions in Da. Fragment ions with m/z larger than ``precursor_mz - precursor_ions_removal_da`` will be removed. Based on our tests, removing precursor ions can enhance search performance. Default is ``1.6``.

- ``clean``: Whether to clean the query spectrum before searching. Default is ``True``.

- ``noise_threshold``: The intensity threshold for removing noise peaks. Peaks with intensity smaller than ``noise_threshold * max(fragment ion's intensity)`` will be removed. Default is ``0.01``.

- ``min_ms2_difference_in_da``:  Minimum spacing allowed between MS/MS peaks during cleaning. Default is ``0.05`` Da.

- ``max_peak_num``: Maximum number of peaks to keep after cleaning. ``None`` keeps all peaks. Default is ``None``.

Use ``search()`` function as follows:

.. code-block:: python

    entropy_similarity = entropy_search.search(
        precursor_mz = 150.0,
        peaks = np.array([[100.0, 1.0], [101.0, 1.0], [102.0, 1.0]], dtype=np.float32),
        method = "all"
    )

Here is an example of the result returned by ``search()`` function:

.. code-block:: python

    {
    'identity_search': array([0.        , 0.        , 0.99999994, 0.        , 0.        , 0.        ], dtype=float32), 
    'open_search': array([0.3333333 , 0.3333333 , 0.99999994, 0.3333333 , 0.        , 0.        ], dtype=float32), 
    'neutral_loss_search': array([0.3333333 , 0.        , 0.99999994, 0.3333333 , 0.        , 0.        ], dtype=float32), 
    'hybrid_search': array([0.6666666 , 0.3333333 , 0.99999994, 0.6666666 , 0.        , 0.        ], dtype=float32)}



Alternative: individual search functions
========================================

Both ``search_topn_matches()`` function and ``search()`` function include internal cleaning of query spectrum before performing search. 
If you want to seperate these two process, you can set ``clean`` in these two functions to ``False`` and use an external clean function.

You can use the ``clean_spectrum`` function to clean the query spectrum and then use individual search functions to search the library. 

.. _clean-spectrum-function:

Clean spectrum
---------------

Before performing a library search, the query spectrum should be pre-processed using the ``clean_spectrum()`` function. This function accomplishes the following:

1. Remove empty peaks (m/z <= 0 or intensity <= 0).

2. Remove peaks with m/z values greater than ``precursor_mz - precursor_ions_removal_da`` (removes precursor ions to improve the quality of spectral comparison).

3. Centroid the spectrum by merging peaks within +/- ``min_ms2_difference_in_da`` and sort the resulting spectrum by m/z.

4. Remove peaks with intensity less than ``noise_threshold`` * maximum intensity.

5. Retain only the top max_peak_num peaks and remove all others.

6. Normalize the intensity to sum to 1.

Assuming you have your query spectrum as:

.. code-block:: python

    query_spectrum = {"precursor_mz": 150.0,
                      "peaks": np.array([[100.0, 1.0], [101.0, 1.0], [102.0, 1.0]], dtype=np.float32)}


Here's how to use ``clean_spectrum()`` function:

.. code-block:: python

    from ms_entropy import clean_spectrum

    precursor_ions_removal_da = 1.6

    query_spectrum['peaks'] = clean_spectrum(
        peaks = query_spectrum['peaks'],
        max_mz = query_spectrum['precursor_mz'] - precursor_ions_removal_da
    )



Performing library search using individual search functions
-----------------------------------------------------------

There are four individual search functions available for library searching:

- ``identity_search`` for Identity search
- ``open_search`` for Open search
- ``neutral_loss_search`` for Neutral loss search
- ``hybrid_search`` for Hybrid search

For these functions, query spectrum should be pre-cleaned as input. These functions will return an array of similarity scores in the order of the whole spectral index.

.. warning::
    If the query spectrun are not pre-processed by ``clean_spectrum()``, there will be error when performing search.
    
Each search function accepts the following parameters:

- ``precursor_mz``: The precursor m/z of the query spectrum. Not required in ``open_search()``.
- ``peaks``: The peaks of the query spectrum.
- ``ms1_tolerance_in_da``: The mass tolerance to use for the precursor m/z in Da. Only for ``identity_search()``.
- ``ms2_tolerance_in_da``: The mass tolerance to use for the fragment ions in Da.

Here's an example of how you can use these functions:

.. code-block:: python
        
    # Identity search
    entropy_similarity = entropy_search.identity_search(
        precursor_mz = query_spectrum['precursor_mz'],
        peaks = query_spectrum['peaks'],
        ms1_tolerance_in_da = 0.01,
        ms2_tolerance_in_da = 0.02
    )

    # Open search
    entropy_similarity = entropy_search.open_search(
        peaks = query_spectrum['peaks'],
        ms2_tolerance_in_da = 0.02
    )

    # Neutral loss search
    entropy_similarity = entropy_search.neutral_loss_search(
        precursor_mz = query_spectrum['precursor_mz'],
        peaks = query_spectrum['peaks'],
        ms2_tolerance_in_da = 0.02
    )

    # Hybrid search
    entropy_similarity = entropy_search.hybrid_search(
        precursor_mz = query_spectrum['precursor_mz'],
        peaks = query_spectrum['peaks'],
        ms2_tolerance_in_da = 0.02
    )

Result is like this:

.. code-block:: python

    [0.3333333  0.3333333  0.99999994 0.3333333  0.         0.        ] # An array of similarity scores in the order of the whole spectral index.