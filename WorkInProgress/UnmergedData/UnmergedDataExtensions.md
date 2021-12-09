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

Do we need to store all that information (and possibly more) for each `BATCH` as shown? A lot is repetitive (describing the hardware environment) and we should make use of the hierarchical capabilities within mmCIF here.

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

So if we go through the `_diffrn_radiation` loop we can find the `wavelength_id` corresponding to the `diffrn_id`. We can also get more information about a specific `diffrn_id` by looking through the `_diffrn_measurement` loop - but that is just some arbitrary string and describes all images within that `diffrn_id`. To uniquely identify a given image (`BATCH` in MTZ lingo) we would need to combine `diffrn_id` and `pdbx_image_id` from the `_diffrn_refln` loop ... not very elegant.

## mimicking BATCH (MTZ) in mmCIF

We could use the combination of diffrn_id and pdbx_image_id with basically the same meaning as the `BATCH` number of MTZ files:

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
_diffrn_batch.detector_setting_id
_diffrn_batch.beam_id
_diffrn_batch.goniostat_setting_id
_diffrn_batch.axis_id
_diffrn_batch.rotation_offset
_diffrn_batch.time_start
_diffrn_batch.time_end
_diffrn_batch.cell_id
_diffrn_batch.mosaicity
1 28 1 1 1 'omega' 13.5 '2020-02-25T12:54:10.385' '2020-02-25T12:54:10.635' 1 0.106
2 22 2 1 1 'omega' 10.5 '2020-02-25T12:54:08.885' '2020-02-25T12:54:09.135' 2 0.061
```

that itself contains further pointers for instrument settings (detector, goniostat, beam) as well as derived information (like cell parameters).

The following is not necessarily taking already existing categories and items from e.g. the [imgCIF 1.8.4][https://github.com/yayahjb/cbflib/tree/main/doc] and/or [imgCIF 1.3.2][https://www.iucr.org/__data/iucr/cifdic_html/2/cif_img.dic/index.html] definitions. Anyway, some ideas (working from the MTZ BATCH philosophy) ...

The detector can be split into an actual detector description and a detector setting one:

```
loop_
_diffrn_detector_setting.id
_diffrn_detector_setting.detector_id
_diffrn_detector_setting.distance
_diffrn_detector_setting.twotheta_value
1 1 350.0  0.0
2 1 349.94 0.0
```

```
loop_
_diffrn_detector.id
_diffrn_detector.axis_fast_direction_x
_diffrn_detector.axis_fast_direction_y
_diffrn_detector.axis_fast_direction_z
_diffrn_detector.axis_fast_pixel_num
_diffrn_detector.axis_fast_pixel_size
_diffrn_detector.axis_slow_direction_x
_diffrn_detector.axis_slow_direction_y
_diffrn_detector.axis_slow_direction_z
_diffrn_detector.axis_slow_pixel_num
_diffrn_detector.axis_slow_pixel_size
_diffrn_detector.twotheta_axis_x
_diffrn_detector.twotheta_axis_y
_diffrn_detector.twotheta_axis_z
1 1.0 0.0 0.0 2463 0.172 0.0 1.0 0.0 2527 0.172 1.0 0.0 1.0
```

The incident beam could be described as

```
loop_
_diffrn_beam.id
_diffrn_beam.direction_x
_diffrn_beam.direction_y
_diffrn_beam.direction_z
_diffrn_beam.flux
_diffrn_beam.wavelength_id
1 0.0 0.0 1.0 9.49e+11 1
```

and

```
loop_
_diffrn_radiation_wavelength.id
_diffrn_radiation_wavelength.wavelength
1 0.97918
```

For the goniostat we would have a similar distinction:

```
loop_
_diffrn_goniostat_setting.id
_diffrn_goniostat_setting.goniostat_id
_diffrn_goniostat_setting.omega_value
_diffrn_goniostat_setting.kappa_value
_diffrn_goniostat_setting.chi_value
_diffrn_goniostat_setting.phi_value
1 1  0.001 0.0 . 0.0
2 1 25.000 0.0 . 0.0
```

and

```
loop_
_diffrn_goniostat.id
_diffrn_goniostat.omega_axis_x
_diffrn_goniostat.omega_axis_y
_diffrn_goniostat.omega_axis_z
_diffrn_goniostat.kappa_axis_x
_diffrn_goniostat.kappa_axis_y
_diffrn_goniostat.kappa_axis_z
_diffrn_goniostat.chi_axis_x
_diffrn_goniostat.chi_axis_y
_diffrn_goniostat.chi_axis_z
_diffrn_goniostat.phi_axis_x
_diffrn_goniostat.phi_axis_y
_diffrn_goniostat.phi_axis_z
1 1.0 0.0 0.0 0.64758809 0.0 -0.76199059 . . . 1.0 0.0 0.0
```

Or use the [imgCIF definition][http://www.bernstein-plus-sons.com/software/CBF/doc/cif_img_1.8.4.html#diffrn_detector_axis] of the goniostat/axes:

```
loop_
_axis.id
_axis.type
_axis.equipment
_axis.depends_on
_axis.vector[1]
_axis.vector[2]
_axis.vector[3]
omega rotation goniometer     .    1.0 0.0 0.0
kappa rotation goniometer omega    0.64758809 0.0 -0.76199059
phi   rotation goniometer kappa    1.0 0.0 0.0
```

But that would work only for a single goniostat. So maybe a combination (whenever we collected data on several instruments with different types of goniometers):

```
loop_
_diffrn_goniostat.id
_diffrn_goniostat.axis_1
_diffrn_goniostat.axis_2
_diffrn_goniostat.axis_3
1 omega-1 kappa-1 phi-1
2 omega-2 . .

loop_
_axis.id
_axis.type
_axis.equipment
_axis.depends_on
_axis.vector[1]
_axis.vector[2]
_axis.vector[3]
omega-1 rotation goniometer     .    1.0 0.0 0.0
kappa-1 rotation goniometer omega-1  0.64758809 0.0 -0.76199059
phi-1   rotation goniometer kappa-1  1.0 0.0 0.0
omega-2 rotation goniometer     .    0.0 -1.0 0.0
```

Finally for the cell:

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
1 150.67 . 150.67 . 111.48 .  90.00 .  90.00 . 120.00 .
2 150.61 . 150.61 . 111.46 .  90.00 .  90.00 . 120.00 .
```

At the moment the structure of the mmCIF categories seem to be mainly
concerned for describing the way from experiment to final
measurements. But if the deposition of unmerged data is considered
useful for (re-)analysing these measurements under the knowledge of
experimental conditions, a different hierarchy (as described above)
seems better.

Currently we have some existing categories/items, but most contain
diffrn_id pointing to _diffrn.id: I find it difficult to get my head
around how to “traverse” that structure the other way around ...

## TODO

- We should also look at v5-next (_pdbx_diffrn_unmerged_refln.scan_id and _pdbx_diffrn_scan): can/should some of those categories/items be re-used?
- Look into imgCIF etc for the hardware descriptions (instead of possibly re-inventing the wheel here)
