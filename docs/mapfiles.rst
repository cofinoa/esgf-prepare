.. _mapfiles:


Generation mapfiles for ESGF publication
========================================

The publication process on the ESGF nodes requires *mapfiles*. Mapfiles are text files where each line
describes a file to publish, using the following format:

``dataset_ID | absolute_path | size_bytes [ | option=value ]``

 1. All values have to be pipe-separated.
 2. The dataset identifier, the absolute path and the size (in bytes) are required.
 3. Adding the version number to the dataset identifier is strongly recommended to publish in a in bulk.
 4. Strongly recommended optional values are:

  - ``mod_time``: last modification date of the file (since Unix EPOCH time, i.e., seconds since January, 1st, 1970),
  - ``checksum``: file checksum,
  - ``checksum_type``: checksum type (MD5 or the default SHA256).

 5. Your directory structure has to strictly follows the tree fixed by the DRS including the version facet.
 6. To store ONE mapfile PER dataset is strongly recommended.

``esgprep mapfile`` is a flexible command-line allowing you to easily generate mapfiles

Default mapfile generation
**************************

The default behavior is to pick up the latest version in the DRS. This required version with a date format
(e.g., v20151023). If the version is directly specified in positional argument, the version number from supplied
directory is used.

.. code-block:: bash

    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/

Mapfile without files checksums
*******************************

Because this could be time consuming ``--no-checksum`` allows you not include the file checksum into your mapfile(s).

.. code-block:: bash

    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --no-checksum

Mapfile without DRS versions
****************************

.. code-block:: bash

    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --no-version

Mapfile name using tokens
*************************

The mapfile name is composed by the dataset ID and the dataset version dot-separated. Another template
can be specified for the output mapfile(s) name using several tokens. Substrings ``{dataset_id}``, ``{version}``,
``{job_id}`` or ``{date}`` (in YYYYDDMM) will be substituted where found. If ``{dataset_id}`` is not present in mapfile
name, then all datasets will be written to a single mapfile, overriding the default behavior of producing ONE mapfile
PER dataset.

.. code-block:: bash

    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --mapfile {dataset_id}.{job_id}
    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --mapfile {date}.{job_id}
    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --mapfile MY_MAPFILE.{version}.{date}

Organize your mapfiles
**********************

The mapfile(s)are generated into a ``mapfile`` folder created in your working directory (if exists). This can be
changed by submitting an output directory for your mapfiles.

In addition, a ``mapfile_drs`` attribute can be added into the corresponding project section of the configuration INI
file(s) (see :ref:`configuration`). In the same way as the ``directory_format`` it defines a tree depending on the
facets. Each mapfile is then written into the corresponding output directory. This ``mapfile_drs`` directory structure
will be added to the output directory if submitted.

.. code-block:: bash

    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --outdir /PATH/TO/MY_MAPFILES/

The output directory is cleaned up prior to mapfile process to avoid uncompleted mapfiles. In the case of several
``esgprep mapfile`` instances run with the same output directory it is recommended to disable the cleanup:

.. code-block:: bash

    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --no-cleanup

Walking through *latest* directories only
*****************************************

.. code-block:: bash

    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --latest-symlink

Walking through a particular version only
*****************************************

.. code-block:: bash

    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --version VERSION

Walking through all versions
****************************

.. code-block:: bash

    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --all-versions

.. warning:: This disables ``--no-version``.

Add technical notes
*******************

.. code-block:: bash

    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --tech-notes-url URL --tech-notes-title TITLE

Overwrite the dataset identifier
********************************

.. code-block:: bash

    $> esgprep mapfile --project PROJECT_ID /PATH/TO/SCAN/ --dataset DATASET_NAME

.. warning:: All files will belong to the specified dataset, regardless of the DRS.

Exit status
***********

 * Status = 0
    All the files have been successfully scanned and the mapfile(s) properly generated.
 * Status = 1
    No files found. No mapfile(s) generated.
 * Status = 2
    Some scan errors occurred. Some files have been skipped or failed during the scan potentially leading to incomplete
    mapfiles. See the error logfile.
 * Status = 3
    All the files have been skipped or failed during the scan leading to no mapfile(s). See the error logfile.
