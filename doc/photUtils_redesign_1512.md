#Bandpass class


##Member variables

###wavelen
The wavelength grid in nanometers.  Stored as a numpy array.

###sb
The throughput curve as a function of ```wavelen```.  This will be the probability of a photon being observed.  Stored as a numpy array.

###_work_wavelen
Initialized as ```None```.  Whenever ```wavelen``` needs to be resampled to match an ```Sed```, the resampled ```wavelen``` will be stored here.  That way, the original throughput information is not lost.

###_work_sb
Initialized as ```None```.  Whenever ```wavelen``` needs to be resampled to match an ```Sed```, the resampled ```sb``` will be stored here.  That way, the original throughput information is not lost.

###fill_value
If integrating this ```Bandpass``` over an ```Sed``` with a region of non-overlap in wavelength (either the ```Bandpass``` does not cover the ```Sed``` or vice-versa), what ```flux```-times-```sb``` value will be used in the non-overlapping regions.  If ```fill_value``` is ```numpy.NaN```, then the resulting flux or magnitude for the whole ```Bandpass``` will be ```numpy.NaN```.  If ```fill_value``` is zero, then the resulting flux or magnitude will only include the region in wavelength space where the ```Bandpass``` and ```Sed``` do overlap; the non-overlapping region will be ignored.  ```fill_value``` will default to ```numpy.NaN```.

##Member methods


#Sed class

##Member variables

##Member methods
