## Introduction

The purpose is to look at MTZ BATCH data as a model for e.g. tracking unit cell variation in PDBx/mmCIF for unmerged data. These are some rough/initial ideas & notes!

## MTZ BATCH header description:
- https://www.ccp4.ac.uk/html/mtzlib.html#orientation
- http://legacy.ccp4.ac.uk/html/mtzlib.html

## Example 6W01 (52 + 130 images):
- https://www.rcsb.org/structure/6w01
- https://proteindiffraction.org/project/IDP51000_6w01/
- https://staraniso.globalphasing.org/table1/w0/6w01.html

Running autoPROC (latest 20211020 release, via `process -I /where/ever/6W01/Images`) gives scaled+unmerged data as

- aimless_alldata_unmerged.mtz
- Data_1_autoPROC-STARANISO_all.cif
  - 3rd data block, data_1_scaled_unmerged_intensities_AIMLESS
- Data_2_autoPROC-TRUNCATE_all.cif
  - 3rd data block, data_2_scaled_unmerged_intensities_AIMLESS

## MTZ (AIMLESS)

The typical set of data columns in an unmerged MTZ file:

```
 Col   Min          Max            Num      %          Resolution    Type Column
 num                             Missing complete    Low       High       label
   1            0           76         0  100.00   84.7431    1.6992   H   H
   2            0           76         0  100.00   84.7431    1.6992   H   K
   3            0           64         0  100.00   84.7431    1.6992   H   L
   4            1           12         0  100.00   84.7431    1.6992   Y   M/ISYM
   5         1001         2130         0  100.00   84.7431    1.6992   B   BATCH
   6 -1.21506E+02  2.36818E+04         0  100.00   84.7431    1.6992   J   I
   7  6.94202E-02  2.33440E+03         0  100.00   84.7431    1.6992   Q   SIGI
   8  1.02315E+00  2.40578E+00         0  100.00   84.7431    1.6992   R   SCALEUSED
   9  0.00000E+00  0.00000E+00         0  100.00   84.7431    1.6992   R   SIGSCALEUSED
  10            1            1         0  100.00   84.7431    1.6992   I   NPART
  11  0.00000E+00  0.00000E+00         0  100.00   84.7431    1.6992   R   FRACTIONCALC
  12  4.90000E+00  2.45810E+03         0  100.00   84.7431    1.6992   R   XDET
  13  4.70000E+00  2.52220E+03         0  100.00   84.7431    1.6992   R   YDET
  14  5.10000E-02  8.99500E+01         0  100.00   84.7431    1.6992   R   ROT
  15  5.13000E-03  5.38470E-01         0  100.00   84.7431    1.6992   R   LP
```

- 15 columns for each measurement
- some columns not relevant for XDS+AIMLESS mode (NPART, FRACTIONCALC)
- the important columns here are `M/ISYM` (to convert the unique (H,K,L) into the actually measured Miller indices) and `BATCH` (a pointer into the batch header data structure)
- the two sweeps are handled via a BATCH number offset - that the user of the data needs to be aware of and handle accordingly:
  - adjusted automatically by autoPROC (so it knows how to go from `BATCH` to sweep and image number)
  - 1001-1052 = sweep 1 (offset = 1000)
  - 2001-2130 = sweep 2 (offset = 2000)

## Looking at two measurements

```
  H K L M/ISYM BATCH I SIGI SCALEUSED SIGSCALEUSED NPART FRACTIONCALC XDET YDET ROT LP

    0   2   1    6  1028      0.23      0.35      1.09      0.00
                              1.00      0.00   1243.00   1227.30
                             13.55      0.02
    0   2   1   12  2022     -0.26      0.37      1.40      0.00
                              1.00      0.00   1269.80   1235.80
                             35.95      0.01
```

For a given unique (H,K,L) we have two different `BATCH` values that are a combined pointer to a sweep and an image number.

## MTZ batch header

```
 Orientation data for batch    1028     area detector data  

   Crystal number ...................       1
   Associated dataset ID ............       1
   Cell dimensions ..................  150.67  150.67  111.48   90.00   90.00  120.00
   Cell fix flags ...................      -1       1      -1       0       0       0 
   Orientation matrix U .............    0.0402   -0.2981    0.9537 
       (including setting angles)       -0.5418    0.7954    0.2715 
                                        -0.8395   -0.5277   -0.1296 
   Reciprocal axis nearest PHI..   a*
   Mosaicity ........................  0.107 
   Datum goniostat angles (degrees)..    0.000
   Scan axis ........................  PHI 
   Start & stop Phi angles (degrees).   13.501   14.001 
   Range of Phi angles (degrees).....    0.500 
   Start & stop time (minutes).......     0.00     0.00 
    Crystal goniostat information :-  
      Number of goniostat axes..........       1 
      Goniostat vectors..... PHI ....    0.0000    0.0000    1.0000 
                       .....  ....    0.0000    0.0000    0.0000 
                       .....  ....    0.0000    0.0000    0.0000 
    Beam information :- 
      Idealized X-ray beam vector.......    -1.0000    0.0000    0.0000 
      X-ray beam vector with tilts......   -1.0000   -0.0000   -0.0009 
      Wavelength and dispersion ........   0.97918   0.00000 
 Detector information :-
   Number of detectors...............      1 
      Crystal to Detector distance (mm).  350.388
   Detector swing angle..............    0.000
   Pixel limits on detector..........    1.0 2463.0    1.0 2527.0
```

Do we need to store all that information (and possibly more) for each `BATCH` as shown? A lot is repetitive (describing the hardware environment and experimental setup, which is often constant for a given scan, i.e. set of images/frames) while others (cell dimension, mosicity, orientation matrix) are derived quantities. Can we make use of the hierarchical capabilities within mmCIF here?

## MTZ reflections -> mmCIF (status quo)

These two reflections in MTZ

```
 H K L M/ISYM BATCH I SIGI SCALEUSED SIGSCALEUSED NPART FRACTIONCALC XDET YDET ROT LP
    0   2   1    6  1028      0.23      0.35      1.09      0.00
                              1.00      0.00   1243.00   1227.30
                             13.55      0.02
    0   2   1   12  2022     -0.26      0.37      1.40      0.00
                              1.00      0.00   1269.80   1235.80
                             35.95      0.01
```

are currently converted to mmCIF as

```
loop_
_diffrn_measurement.diffrn_id
_diffrn_measurement.details
1 'images 1-52 0.00-26 degree'
2 'images 1-130 25.00-90 degree'

loop_
_diffrn_radiation.diffrn_id
_diffrn_radiation.wavelength_id
1 1
2 1

loop_
_diffrn_refln.id
_diffrn_refln.diffrn_id
_diffrn_refln.index_h
_diffrn_refln.index_k
_diffrn_refln.index_l
_diffrn_refln.pdbx_image_id
_diffrn_refln.intensity_net
_diffrn_refln.intensity_sigma
_diffrn_refln.pdbx_scale_value
_diffrn_refln.pdbx_detector_x
_diffrn_refln.pdbx_detector_y
_diffrn_refln.pdbx_scan_angle
761 1  2 -2 -1 28 2.31181e-01 3.52052e-01 1.08740e+00 1243.00 1227.30 13.551
762 2  2 0 -1 22 -2.55269e-01 3.71256e-01 1.39567e+00 1269.80 1235.80 35.950
```
To uniquely identify a given image (`BATCH` in MTZ lingo) we need to combine `diffrn_id` and `pdbx_image_id` from the `_diffrn_refln` loop.

If we e.g. go through the `_diffrn_radiation` loop we can find the `wavelength_id` corresponding to the `diffrn_id` of a given reflection. We can also get more information about a specific `diffrn_id` by looking through the `_diffrn_measurement` loop (which at the moment is just some arbitrary string and describes all images within that `diffrn_id`).

It might be better if we would use the ``_diffrn_refln.frame_id`` item defined in [imgCIF 1.8.4](https://github.com/yayahjb/cbflib/tree/main/doc) to e.g. provide a pointer into the various DIFFRN_SCAN* categories - which will allow us to define scans (goniostat settings, rotation axis, start/end times ... although those seem to be limited to yyyy-mm-dd).

The ``diffrn_id`` item allows us access to the full set of DIFFRN_DETECTOR*, DIFFRN_MEASUREMENT* and DIFFRN_RADIATION* categories/items defined in [imgCIF 1.8.4](https://github.com/yayahjb/cbflib/tree/main/doc).

### Caveat/Questions

The following items are defined both in [imgCIF 1.8.4](https://github.com/yayahjb/cbflib/tree/main/doc) as well as in [base/mmcif_pdbx-base.dic](https://github.com/wwpdb-dictionaries/mmcif_pdbx/blob/master/base/mmcif_pdbx-base.dic):
```
_diffrn_detector.details
_diffrn_detector.detector
_diffrn_detector.diffrn_id
_diffrn_detector.dtime
_diffrn_detector.type
_diffrn_measurement.details
_diffrn_measurement.device
_diffrn_measurement.device_details
_diffrn_measurement.device_type
_diffrn_measurement.diffrn_id
_diffrn_measurement.method
_diffrn_measurement.specimen_support
_diffrn_radiation.collimation
_diffrn_radiation.diffrn_id
_diffrn_radiation.filter_edge
_diffrn_radiation.inhomogeneity
_diffrn_radiation.monochromator
_diffrn_radiation.polarisn_norm
_diffrn_radiation.polarisn_ratio
_diffrn_radiation.probe
_diffrn_radiation.type
_diffrn_radiation.wavelength_id
_diffrn_radiation.xray_symbol
```
Would we have to copy all potentially used imgCIF category/item definitions over as well (e.g. to make use of ``frame_id``)? Or include the full imgCIF dictionary as-is into the ``base`` directory of the [mmcif_pdbx](https://github.com/wwpdb-dictionaries/mmcif_pdbx) code? Ultimately, we probably want to avoid duplication and at the same time provide access to the full imgCIF categories and items.

Most of these have identical descriptions - apart from:
```
_diffrn_detector.diffrn_id : additional text "The value of _diffrn.id uniquely defines a set of diffraction data."

_diffrn_detector.dtime     : introduces plutal "detector(s)"

_diffrn_measurement.device : additional text

              If the value of _diffrn_measurement.device is not given,
              it is implicitly equal to the value of
              _diffrn_measurement.diffrn_id.

              Either _diffrn_measurement.device or
              _diffrn_measurement.id may be used to link to other
              categories.  If the experimental setup admits multiple
              devices, then _diffrn_measurement.id is used to provide
              a unique link.

_diffrn_radiation.filter_edge : transformed from "angstroms" to "\%angstr\"oms"

_diffrn_radiation.polarisn_norm : typo in mmCIF version

                                  "_diffrn_radiation_polarisn_ratio" should be
                                  "_diffrn_radiation.polarisn_ratio"

_diffrn_radiation.polarisn_ratio : different spelling "parallel-polarized" becomes "parallel polarized"

_diffrn_radiation.probe : very different text

 imgCIF:      The nature of the radiation used (i.e. the name of the
              subatomic particle or the region of the electromagnetic
              spectrum). It is strongly recommended that this information
              is given, so that the probe radiation can be simply determined.

 mmCIF:       Name of the type of radiation used. It is strongly
              recommended that this be given so that the
              probe radiation is clearly specified.

```

Some items are only defined in [base/mmcif_pdbx-base.dic](https://github.com/wwpdb-dictionaries/mmcif_pdbx/blob/master/base/mmcif_pdbx-base.dic) or [extensions/xfel-extensions-v3.dic](https://github.com/wwpdb-dictionaries/mmcif_pdbx/blob/master/extensions/xfel-extensions-v3.dic):
```
_diffrn_detector.area_resol_mean
_diffrn_detector.pdbx_collection_date
_diffrn_detector.pdbx_collection_time_total
_diffrn_detector.pdbx_frames_total
_diffrn_detector.pdbx_frequency
_diffrn_measurement.pdbx_date
_diffrn_radiation.pdbx_analyzer
_diffrn_radiation.pdbx_diffrn_protocol
_diffrn_radiation.pdbx_monochromatic_or_laue_m_l
_diffrn_radiation.pdbx_scattering_type
_diffrn_radiation.pdbx_wavelength
_diffrn_radiation.pdbx_wavelength_list
_diffrn_radiation_wavelength.id
_diffrn_radiation_wavelength.wavelength
_diffrn_radiation_wavelength.wt
```

## mimicking BATCH (MTZ) in mmCIF

We can make use of a lot of the [imgCIF 1.8.4](https://github.com/yayahjb/cbflib/tree/main/doc) definitions to describe the experimental setup (detector, source, goniostat, scans etc). This would require (1) including the imgCIF dictionary into PDBx/mmCIF and (2) sorting out conflicts and duplication. 

But how would we store derived quantities as a function of image/frame/scan (like cell parameters, scale parameters, mosaicity etc)? This is something that the MTZ BATCH header provides and we could use the combination of diffrn_id and pdbx_image_id with basically the same meaning as the `BATCH` number of MTZ files:

```
loop_
_diffrn_refln.id
_diffrn_refln.diffrn_id
_diffrn_refln.index_h
_diffrn_refln.index_k
_diffrn_refln.index_l
_diffrn_refln.pdbx_image_id
_diffrn_refln.intensity_net
_diffrn_refln.intensity_sigma
_diffrn_refln.pdbx_scale_value
_diffrn_refln.pdbx_detector_x
_diffrn_refln.pdbx_detector_y
_diffrn_refln.pdbx_scan_angle
761 1 2 -2 -1 28 2.31181e-01 3.52052e-01 1.08740e+00 1243.00 1227.30 13.551
762 2 2 0 -1 22 -2.55269e-01 3.71256e-01 1.39567e+00 1269.80 1235.80 35.950
```

This could then point to a batch-specific category:

```
loop_
_diffrn_batch.diffrn_id
_diffrn_batch.pdbx_image_id
_diffrn_batch.xyz
1 28 .
2 22 .
```

that itself contains further values (or pointers) of derived information (like cell parameters). If we had different cell dimensions for (ranges of) images/frames, we could use

```
loop_
_diffrn_batch.diffrn_id
_diffrn_batch.pdbx_image_id
_diffrn_batch.cell_id
1 28 1
2 22 2
```
and then (values from XDS/CORRECT post-refinement):

```
loop_
_diffrn_cell.id
_diffrn_cell.length_a
_diffrn_cell.length_a_esd
_diffrn_cell.length_b
_diffrn_cell.length_b_esd
_diffrn_cell.length_c
_diffrn_cell.length_c_esd
_diffrn_cell.angle_alpha
_diffrn_cell.angle_alpha_esd
_diffrn_cell.angle_beta
_diffrn_cell.angle_beta_esd
_diffrn_cell.angle_gamma
_diffrn_cell.angle_gamma_esd
1 150.672 0.042 150.672 0.042 111.477 0.16  90.00 0.0  90.00 0.0 120.00 0.0
2 150.607 0.068 150.607 0.068 111.465 0.061  90.00 0.0  90.00 0.0 120.00 0.0
```

Of course, here the cell dimensions (and their ESDs) are constant for a whole ``diffrn_id``, but that doesn't need to be the case - which is why we need that intermediate ``_diffrn_batch`` layer. If we want to record the unit-cell parameters as refined in blocks of images (e.g. what XDS/INTEGRATE would/could do with the 6W01 example, at DELPHI ~ 5.0 degree for 0.5 degree images):
```
loop_
_diffrn_batch.diffrn_id
_diffrn_batch.pdbx_image_id
_diffrn_batch.cell_id
1   1  1
1   2  1
...
1  10  1
1  11  1
1  12  2
1  13  2
...
1  22  2
1  23  3
...
1  32  3
1  33  4
...
1  42  4
1  43  5
...
1  52  5
2   1  6
2  10  6
2  11  7
...
2 130 18
```

and then
```
loop_
_diffrn_cell.id
_diffrn_cell.length_a
_diffrn_cell.length_a_esd
_diffrn_cell.length_b
_diffrn_cell.length_b_esd
_diffrn_cell.length_c
_diffrn_cell.length_c_esd
_diffrn_cell.angle_alpha
_diffrn_cell.angle_alpha_esd
_diffrn_cell.angle_beta
_diffrn_cell.angle_beta_esd
_diffrn_cell.angle_gamma
_diffrn_cell.angle_gamma_esd
1 150.539 . 150.539 . 111.133 . 90.000 . 90.000 . 120.000
2 150.558 . 150.558 . 111.202 . 90.000 . 90.000 . 120.000
3 150.601 . 150.601 . 111.295 . 90.000 . 90.000 . 120.000
4 150.631 . 150.631 . 111.381 . 90.000 . 90.000 . 120.000
5 150.666 . 150.666 . 111.461 . 90.000 . 90.000 . 120.000
6 150.687 . 150.687 . 111.503 . 90.000 . 90.000 . 120.000
7 150.712 . 150.712 . 111.554 . 90.000 . 90.000 . 120.000
8 150.687 . 150.687 . 111.542 . 90.000 . 90.000 . 120.000
9 150.665 . 150.665 . 111.527 . 90.000 . 90.000 . 120.000
10 150.660 . 150.660 . 111.519 . 90.000 . 90.000 . 120.000
11 150.625 . 150.625 . 111.479 . 90.000 . 90.000 . 120.000
12 150.592 . 150.592 . 111.445 . 90.000 . 90.000 . 120.000
13 150.575 . 150.575 . 111.418 . 90.000 . 90.000 . 120.000
14 150.547 . 150.547 . 111.379 . 90.000 . 90.000 . 120.000
15 150.540 . 150.540 . 111.367 . 90.000 . 90.000 . 120.000
16 150.534 . 150.534 . 111.349 . 90.000 . 90.000 . 120.000
17 150.507 . 150.507 . 111.318 . 90.000 . 90.000 . 120.000
18 150.497 . 150.497 . 111.299 . 90.000 . 90.000 . 120.000
```

We could envisage adding additional information, e.g. scale parameters by adding ``_diffrn_batch.scale_id``
```
loop_
_diffrn_batch.diffrn_id
_diffrn_batch.pdbx_image_id
_diffrn_batch.cell_id
_diffrn_batch.scale_id
1   1  1   1
1   2  1   1
...
1  10  1   5
1  11  1   6
1  12  2   6
1  13  2   7
...
1  22  2  11
1  23  3  12
...
1  32  3  16
1  33  4  17
...
1  42  4  21
1  43  5  22
...
1  52  5  26
2   1  6  27
...
2  10  6  31
2  11  7  31
...
2 130 18  91
```
and then something like (AIMLESS scale parameters for 6W01 example, given at 1-degree intervals for those 0.5-degree images):
```
loop_
_diffrn_scale.id
_diffrn_scale.scale_k
_diffrn_scale.scale_B
1 1.0008 -0.6110
2 1.0021 -0.6031
3 1.0051 -0.5939
4 1.0111 -0.5834
5 1.0212 -0.5715
6 1.0356 -0.5580
7 1.0499 -0.5428
8 1.0628 -0.5261
9 1.0753 -0.5079
10 1.0897 -0.4883
11 1.1087 -0.4675
12 1.1263 -0.4459
13 1.1420 -0.4236
14 1.1565 -0.4012
15 1.1728 -0.3790
16 1.1920 -0.3573
17 1.2146 -0.3365
18 1.2332 -0.3169
19 1.2510 -0.2987
20 1.2715 -0.2820
21 1.2963 -0.2669
22 1.3232 -0.2534
23 1.3432 -0.2414
24 1.3556 -0.2309
25 1.3617 -0.2218
26 1.3641 -0.2139
27 1.3657 -0.0480
28 1.3706 -0.1441
29 1.3777 -0.2398
30 1.3893 -0.3339
31 1.4074 -0.4255
32 1.4325 -0.5139
33 1.4565 -0.5988
34 1.4770 -0.6805
35 1.4958 -0.7591
36 1.5167 -0.8355
37 1.5423 -0.9104
38 1.5640 -0.9847
39 1.5813 -1.0595
40 1.5949 -1.1357
41 1.6078 -1.2142
42 1.6220 -1.2956
43 1.6328 -1.3804
44 1.6405 -1.4687
45 1.6450 -1.5604
46 1.6477 -1.6547
47 1.6495 -1.7508
48 1.6498 -1.8477
49 1.6493 -1.9993
50 1.6476 -2.0984
51 1.6445 -2.1971
52 1.6399 -2.2944
53 1.6355 -2.3898
54 1.6312 -2.4827
55 1.6264 -2.5732
56 1.6204 -2.6616
57 1.6131 -2.7487
58 1.6072 -2.8353
59 1.6035 -2.9224
60 1.6023 -3.0113
61 1.6030 -3.1031
62 1.6040 -3.1989
63 1.6049 -3.2997
64 1.6034 -3.4060
65 1.5984 -3.5181
66 1.5895 -3.6360
67 1.5771 -3.7590
68 1.5654 -3.8860
69 1.5555 -4.0157
70 1.5464 -4.1462
71 1.5363 -4.3240
72 1.5243 -4.4519
73 1.5146 -4.5769
74 1.5074 -4.6974
75 1.5023 -4.8123
76 1.4982 -4.9206
77 1.4942 -5.0221
78 1.4915 -5.1168
79 1.4899 -5.2052
80 1.4896 -5.2878
81 1.4903 -5.3657
82 1.4913 -5.4399
83 1.4921 -5.5112
84 1.4919 -5.5809
85 1.4902 -5.6496
86 1.4868 -5.7181
87 1.4815 -5.7868
88 1.4760 -5.8561
89 1.4704 -5.9258
90 1.4633 -5.9957
91 1.4539 -6.0653
```
(or similar ...)

## TODO

- We should also look at v5-next (``_pdbx_diffrn_unmerged_refln.scan_id`` and ``_pdbx_diffrn_scan``): can/should some of those categories/items be re-used? Maybe not immediately needed if we first take advantage of existing imCIF items and categories.

