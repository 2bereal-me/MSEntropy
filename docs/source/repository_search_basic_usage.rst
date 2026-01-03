===========
Basic usage
===========


Overview
========

We offer prebuilt index for metabolomics repositories, comprising more than 1.4 billion spectra.
Users can use ``RepositorySearch`` class to search against these public metabolomics repositories. 
We have built the indexes and uploaded them to `Hugging Face repository <https://huggingface.co/datasets/YuanyueLiZJU/dynamic_entropy_search/tree/main>`_.
By downloading these indexes and extracting them to a specified path on your local machine, you can perform search like this:

.. code-block:: python

    from ms_entropy import RepositorySearch

    # Assign the RepositorySearch class: suppose you have downloaded and extracted the indexes to the path_repository_indexes on your local machine
    search_engine = RepositorySearch(path_data=path_repository_indexes)

    query_spec={
        "peaks":np.array([[200.0, 1.0], [101.0, 1.0], [202.0, 1.0]]),
        "precursor_mz":250.0,
        "charge":-1
                        
    }
    query_spec['peaks']=clean_spectrum(
            peaks=query_spec['peaks'],
            max_mz = query_spec['precursor_mz'] - precursor_ions_removal_da
        )

    # Perform search
    search_result = search_engine.search_topn_matches(
        method="open", # or 'open' or  'neutral_loss' or 'identity'
        charge=query_spectrum_charge,
        precursor_mz = query_spectrum_precursor_mz, 
        peaks = query_spectrum_peaks,
        )
    
    print(search_result)

If you want to extract any spectrum from the results, you can change ``spec_idx`` to select a spectrum:

.. code-block:: python

    def get_spectrum_data(search_engine: RepositorySearch, charge, spec_idx):
        spec = search_engine.get_spectrum(charge, spec_idx)
        spec.pop("scan", None)
        return spec

    spec_data = get_spectrum_data(search_engine, query_spec["charge"], search_result[0].pop("spec_idx")) # Or search_result[1], etc.
    spec_data.update(search_result[0]) # Or search_result[1], etc.
    print(f"Top match spectrum data: {spec_data}")

An example of the extracted result:

.. code-block:: python

    Top match spectrum data: {'precursor_mz': 512.233642578125, 'charge': -1, 'rt': 76.76499938964844, 'peaks': array([[200.00693   ,   0.74098176],
       [202.0056    ,   0.2590183 ]], dtype=float32), 'file_name': 'metabolomics_workbench/ST003745/x01997_NEG.mzML.gz', 'scan': np.uint64(1139), 'similarity': np.float64(0.8030592799186707)}


.. note::
    In the part of Dynamic Entropy Search, you can find instructions on how to prepare the query spectrum.

    It should be noted that, query spectrum in this part should have a key ``charge`` with an integer value (-1 or 1). 


------------


Example
========

You can refer to this example to use this package. 
This example can be found in the root directory of DynamicEntropySearch `Github <https://github.com/2bereal-me/DynamicEntropySearch>`_.

example.py
----------

This script demonstrates how to use the RepositorySearch module from scratch.
If you want to use our prebuilt indexes, replace the ``path_data`` with the index path that you have downloaded and extracted to your local machine. Then search can be performed directly.

If you want to build or add your own reference spectra, you can follows the instruction in this example.

