## Notes on ALOS-2 data processing

### Ionospheric correction

1. Estimate ionospheric delays in isce2/alosStack.

2. Correct ionospheric delays in mintpy.

```bash
# est raw TS
smallbaselineApp.py ../JinshaAlos2A147F510.txt --end invert_network

# est iono TS
# check the `fig_ion` folder to locate the pairs with failed ionospheric estimation
modify_network.py inputs/ionStack.h5 --ex-date12 160715-200403 160923-210402 --update
reference_point.py inputs/ionStack.h5 -t smallbaselineApp.cfg --update
ifgram_inversion.py inputs/ionStack.h5 -t smallbaselineApp.cfg -w no --update

# phase corrections
diff.py timeseries.h5 ion.h5 -o timeseries_ion.h5 --update
tropo_pyaps3.py -f timeseries_ion.h5 -g inputs/geometryRadar.h5 --update
solid_earth_tides.py timeseries_ion_ERA5.h5 -g inputs/geometryRadar.h5 --update
remove_ramp.py timeseries_ion_ERA5_SET.h5 -m maskTempCoh.h5 --update
dem_error.py timeseries_ion_ERA5_SET_ramp.h5 -g inputs/geometryRadar.h5 -t smallbaselineApp.cfg --update
timeseries_rms.py timeseriesResidual.h5 -t smallbaselineApp.cfg
reference_date.py timeseries.h5 timeseries_*.h5 -t smallbaselineApp.cfg

# est time func
timeseries2velocity.py timeseries_ion_ERA5_SET_ramp_demErr.h5 -t smallbaselineApp.cfg --update

# post-processing
geocode.py inputs/geometryRadar.h5 timeseries_ion_ERA5_SET_ramp_demErr.h5 temporalCoherence.h5 avgSpatialCoh.h5 velocity.h5 -t smallbaselineApp.cfg -
-outdir ./geo --update
smallbaselineApp.py --plot
```
