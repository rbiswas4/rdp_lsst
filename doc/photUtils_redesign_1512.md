#Bandpass class


##Member variables

###```_wavelen```
The wavelength grid in nanometers.  Stored as a numpy array.

###```_sb```
The throughput curve as a function of ```wavelen```.  This will be the probability of a photon being observed.  Stored as a numpy array.

###```_ab_norm```
This will be the normalization value applied when calculating AB fluxes.  It is the integral over ```wavelen`` of ```sb```/```wavelen```.  It will be set by ```__init___``` and reset whenever ```wavelen``` or ```sb``` change.

###```_work_wavelen```
Initialized as ```None```.  Whenever ```wavelen``` needs to be resampled to match an ```Sed```, the resampled ```wavelen``` will be stored here.  That way, the original throughput information is not lost.

###```_work_sb```
Initialized as ```None```.  Whenever ```wavelen``` needs to be resampled to match an ```Sed```, the resampled ```sb``` will be stored here.  That way, the original throughput information is not lost.

###```_fill_value```
If integrating this ```Bandpass``` over an ```Sed``` with a region of non-overlap in wavelength (either the ```Bandpass``` does not cover the ```Sed``` or vice-versa), what ```flux```-times-```sb``` value will be used in the non-overlapping regions.  If ```fill_value``` is ```numpy.NaN```, then the resulting flux or magnitude for the whole ```Bandpass``` will be ```numpy.NaN```.  If ```fill_value``` is zero, then the resulting flux or magnitude will only include the region in wavelength space where the ```Bandpass``` and ```Sed``` do overlap; the non-overlapping region will be ignored.  ```fill_value``` will default to ```numpy.NaN```.

###```_threshold```
The minimum value of ```sb``` that must be present at a value of ```wavelen``` for non-overlap to be considered a problem (i.e. if you are trying to integrate the ```Bandpass``` over an ```Sed``` with a different ```wavelen``` array).  Defaults to 10^-20


##Member methods

###```__init___(wavelen, sb)```

####Arguments
- ```wavelen``` -- a numpy array containing the wavelength grid in nanometers
- ```sb``` -- a numpy array containing the probability of a photon at a given wavelength being detected.

####Results
- automatically set ```_ab_norm```

###```readThroughput(filename)```
This will be a class method so that it can be called without instantiating ```Bandpass``` first.

####Arguments
-```filename``` -- a string denoting the name of the throughput file to be read

####Results
-reads in ```filename```, expecting two columns: wavelength in nanometers and sb as a function of wavelength.
-returns an intantiation of ```Bandpass```

###```__mul__(other)```

####Arguments
-```other``` -- another ```Bandpass```

####Results
-multiplies together two ```Bandpasses``` by resampling the ```Bandpasses``` onto the finest ```wavelen``` grid that can accommodate both and multiplying their ```sb``` arrays together
-Returns a new ```Bandpass``` instantiation that is the result of multiplying the two ```Bandpasses``` by each other.
-Raise an exception of there is a region of non-overlap in the two ```Bandpass```es ```wavelen``` arrays.  The non-overlap region must contain ```sb```>```_threshold``` to actually raise an exception.

#Sed class

##Member variables

##Member methods
