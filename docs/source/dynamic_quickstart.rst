===========
Quick start
===========


Overview
========

.. code-block:: python

    from ms_entropy import DynamicEntropySearch

    # Step 1: Construct the DynamicEntropySearch class.
    entropy_search = DynamicEntropySearch(path_data=path_of_your_library)

    # Step 2: Construct or update the index from the library spectra.
    entropy_search.add_new_spectra(spectra_list=spectra_1_for_library)
    entropy_search.add_new_spectra(spectra_list=spectra_2_for_library)
    ......

    # Step 3: Call build_index() and write() lastly to end adding operation.
    entropy_search.build_index()
    entropy_search.write()

    # Step 4: Perform search.
    entropy_similarity=entropy_search.search(
        precursor_mz=query_spectrum_precursor_mz,
        peaks=query_spectrum_peaks)

------------

In detail
=========

Prepare the spectra library
---------------------------

Suppose you have a lot of spectral libraries, you need to format them like this:

.. code-block:: python

    import numpy as np
    spectra_1_for_library = [{
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
    "peaks": np.array([[100.0, 1.0], [101.0, 1.0], [302.0, 1.0]], dtype=np.float32),
    },
    ]

    spectra_2_for_library ... # Similar to spectra_1_for_library
    spectra_3_for_library ... # Similar to spectra_1_for_library

Note that the ``precursor_mz`` and ``peaks`` keys are required, the reset of the keys are optional.


Make sure you have your specrta libraries cleaned. See :ref:`clean-spectrum-function` for the details of functions. 
You can clean spectra libraries using ``clean_spectrum()`` as follows:

.. code-block:: python

    from ms_entropy import clean_spectrum
    precursor_ions_removal_da=1.6
    for spec in spectra_1_for_library:
        spec['peaks']=clean_spectrum(
            peaks=spec['peaks'],
            max_mz = spec['precursor_mz'] - precursor_ions_removal_da
        )

    for spec in spectra_2_for_library:
        spec['peaks']=clean_spectrum(
            peaks=spec['peaks'],
            max_mz = spec['precursor_mz'] - precursor_ions_removal_da
        )

    for spec in spectra_3_for_library:
        spec['peaks']=clean_spectrum(
            peaks=spec['peaks'],
            max_mz = spec['precursor_mz'] - precursor_ions_removal_da
        )

Update the index
----------------

Then you can have your spectra libraries to be added into the library no matter it is an initialization or an update step by using ``add_new_spectra()``.

.. code-block:: python
    
    from ms_entropy import DynamicEntropySearch

    # Assign the path for your library.
    entropy_search=DynamicEntropySearch(path_data=path_of_your_library)

    # Add spectra into the library one by one
    entropy_search.add_new_spectra(spectra_list=spectra_1_for_library)
    entropy_search.add_new_spectra(spectra_list=spectra_2_for_library)
    entropy_search.add_new_spectra(spectra_list=spectra_3_for_library)

    # Lastly, call build_index() and write() to end the adding operation.
    entropy_search.build_index()
    entropy_search.write()

.. note::
    It is necessary to initialize ``DynamicEntropySearch`` using a specified ``path_data``, which is the path of your library. The reset of the parameters are optional.

    If you only want to construct index for open search, you can set ``index_for_neutral_loss`` in ``add_new_spectra()`` and ``build_index()`` to ``False``.

    It is necessary to call ``build_index()`` and ``write()`` lastly after all ``add_new_spectra()`` as the end of adding operation.


Prepare the query spectra
-------------------------

Then you have your query spectrum looks like this:

.. code-block:: python

    query_spec={
        "peaks":np.array([[200.0, 1.0], [101.0, 1.0], [202.0, 1.0]], dtype=np.float32),
        "precursor_mz":250.0}

Perform search
--------------

You can call the ``DynamicEntropySearch`` class with corresponding ``path_data`` to search the library like this:

.. code-block:: python

    from ms_entropy import DynamicEntropySearch
    # Assign the path for your library
    entropy_search=DynamicEntropySearch(path_data=path_of_your_library)
    
    # Search the library and you can fetch the metadata from the results with the highest scores
    result=entropy_search.search_topn_matches(
            precursor_mz=query_spectrum['precursor_mz'],
            peaks=query_spectrum['peaks'],
            ms1_tolerance_in_da=0.01, # You can change ms1_tolerance_in_da as needed.
            ms2_tolerance_in_da=0.02, # You can change ms2_tolerance_in_da as needed.
            method='open', # or 'neutral_loss' or 'hybrid' or 'identity'.
            clean=True, # If you don't want to use the internal clean process in this function, set it to False.
            topn=3, # You can change topn as needed.
            need_metadata=True, # Set it to True if need metadata.
    )

    # After that, you can print the result like this:
    print(result)

An example of the results is shown below:

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


.. note::
    If your query spectra is a list consisting of several spectra:

    .. code-block:: python

        import numpy as np
        # For each query_spectra_list, it is a list consisting of multiple dictionaries of query MS2 spectra.

        # For each query spectrum, 'precursor_mz' and 'peaks' are necessary. 
        # 'precursor_mz' should be a float, and 'peaks' should be a 2D np.ndarray like np.ndarray([[m/z, intensity], [m/z, intensity], [m/z, intensity]...], dtype=np.float32).

        query_spectra_list = [{
                        "precursor_mz": 150.0,
                        "peaks": np.array([[100.0, 1.3], [101.0, 1.5], [102.0, 1.8]], dtype=np.float32)
                        },{
                        "precursor_mz": 250.0,
                        "peaks": np.array([[108.0, 2.4], [113.0, 1.0], [157.0, 0.9]], dtype=np.float32)
                        },{
                        "precursor_mz": 299.0,
                        "peaks": np.array([[119.0, 3.3], [145.0, 0.3], [157.0, 2.1]], dtype=np.float32)
                        },
                        ]
                        
    For a list of spectra to be queried, you can call the ``DynamicEntropySearch`` class with corresponding ``path_data`` and **iterate** the list to search the library like this:

    .. code-block:: python

        from ms_entropy import DynamicEntropySearch
        # Assign the path for your library
        entropy_search=DynamicEntropySearch(path_data=path_of_your_library)
        
        # Iterate query_spectra_list to perform search one by one
        # Search the library and you can fetch the metadata from the results with the highest scores
        for spec in query_spectra_list:
            result=entropy_search.search_topn_matches(
                    precursor_mz=spec['precursor_mz'],
                    peaks=spec['peaks'],
                    ms1_tolerance_in_da=0.01, # You can change ms1_tolerance_in_da as needed.
                    ms2_tolerance_in_da=0.02, # You can change ms2_tolerance_in_da as needed.
                    method='open', # or 'neutral_loss' or 'hybrid' or 'identity'.
                    clean=True, # If you don't want to use the internal clean process in this function, set it to False.
                    topn=3, # You can change topn as needed.
                    need_metadata=True, # Set it to True if need metadata.
            )

            # After that, you can print the result like this:
            print(result)


.. note::
    The ``search_topn_matches`` function will return the top few similarity scores and corresponding metadata. If you want to get all similarity scores and metadata, you can set ``topn`` in this function to ``None``.

------------


More information
==========

More information are provided in the rest sections.
