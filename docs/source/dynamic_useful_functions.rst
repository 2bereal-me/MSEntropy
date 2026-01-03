================
Useful functions
================

Manually convert the index to a compact structure
==================================================

Based on the Flash Entropy Search, the index of Dynamic Entropy Search can be converted to a compact structure like Flash Entropy Search, resulting in a faster search speed.

.. code-block:: python

    from ms_entropy import DynamicEntropySearch
    from pathlib import Path
    # Choose an existing index and assign the path
    entropy_search = DynamicEntropySearch(path_data=path_of_your_library)
    
    # Manually sort the blocks in index
    entropy_search.convert_to_fast_search()

    # Manually convert the index to a compact structure
    entropy_search.read()
    group_num=len(entropy_search.group_start)
    for i in range(group_num):
        group_path=Path(path_of_your_library)/f"{i}"
        entropy_search.entropy_search.path_data=group_path
        entropy_search.entropy_search.read()
        entropy_search.convert_current_index_to_flash()

This operation internally sorts the blocks in the index and then removes reserved space in the index. 
After this process, index can be converted to the structure compatible to Flash Entropy Search. Performance of search will improve when using the search functions in ``DynamicEntropySearch``.

.. warning::
    When using ``add_new_spectra()`` functions to update index, there will be an automatic conversion of index structure if the size of this group meets the limit.
    You can set ``convert_to_flash`` in ``add_new_spectra()`` and ``build_index()`` to ``False`` to disable this feature.
    
    .. code-block:: python

        from ms_entropy import DynamicEntropySearch
        entropy_search = DynamicEntropySearch(path_data=path_of_your_library)
        entropy_search.add_new_spectra(spectra_list=spectra_1_for_library, convert_to_flash=False)
        entropy_search.add_new_spectra(spectra_list=spectra_2_for_library, convert_to_flash=False)
        entropy_search.add_new_spectra(spectra_list=spectra_3_for_library, convert_to_flash=False)
        ......
        entropy_search.build_index(convert_to_flash=False)
        entropy_search.write()

    It should be noted that if the index of group is already a compact structure, converting operation and subsequent adding operation will both result in an error.

Construct an index only for open search
========================================

If you only need to construct index for open search, it is unnecessary to process neutral loss data.
By setting ``index_for_neutral_loss`` in ``add_new_spectra()`` and ``build_index()`` to ``False``, you can construct the index for open search more efficiently.

Here's an example:

.. code-block:: python

    from ms_entropy import DynamicEntropySearch
    entropy_search = DynamicEntropySearch(path_data=path_of_your_library)
    entropy_search.add_new_spectra(spectra_list=spectra_1_for_library, index_for_neutral_loss=False)
    entropy_search.add_new_spectra(spectra_list=spectra_2_for_library, index_for_neutral_loss=False)
    entropy_search.add_new_spectra(spectra_list=spectra_3_for_library, index_for_neutral_loss=False)
    ......
    entropy_search.build_index(index_for_neutral_loss=False)
    entropy_search.write()

.. warning::
    Once ``index_for_neutral_loss`` is set to ``False``, it will no longer be possible to construct neutral loss index of this library. Keep this parameter to ``False`` all the time to avoid errors.
    What's more, only open search can be performed under this circumstance. It is necessary to check the value of ``method`` when using search functions. Performing identity search, neutral loss search or hybrid search can cause error.

