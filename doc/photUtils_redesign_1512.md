#Bandpass class


##Member variables

###```_wavelen```
The wavelength grid in nanometers.  Stored as a numpy array.

###```_sb```
The throughput curve as a function of ```wavelen```.  This will be the probability of a photon being observed.  Stored as a numpy array.

###```_ab_norm```
This will be the normalization value applied when calculating AB fluxes.  It is the integral over ```wavelen`` of ```sb```/```wavelen``` multiplied by 3631 Janskies.  It will be set by ```__init___``` and reset whenever ```wavelen``` or ```sb``` change.

###```_work_wavelen```
Initialized as ```None```.  Whenever ```wavelen``` needs to be resampled to match an ```Sed```, the resampled ```wavelen``` will be stored here.  That way, the original throughput information is not lost.

###```_work_sb```
Initialized as ```None```.  Whenever ```wavelen``` needs to be resampled to match an ```Sed```, the resampled ```sb``` will be stored here.  That way, the original throughput information is not lost.

###```_fill_value```
If integrating this ```Bandpass``` over an ```Sed``` with a region of non-overlap in wavelength (either the ```Bandpass``` does not cover the ```Sed``` or vice-versa), what ```flux```-times-```sb``` value will be used in the non-overlapping regions.  If ```fill_value``` is ```numpy.NaN```, then the resulting flux or magnitude for the whole ```Bandpass``` will be ```numpy.NaN```.  If ```fill_value``` is zero, then the resulting flux or magnitude will only include the region in wavelength space where the ```Bandpass``` and ```Sed``` do overlap; the non-overlapping region will be ignored.  ```fill_value``` will default to ```numpy.NaN```.

###```_threshold```
The minimum value of ```sb``` that must be present at a value of ```wavelen``` for non-overlap to be considered a problem (i.e. if you are trying to integrate the ```Bandpass``` over an ```Sed``` with a different ```wavelen``` array).  Defaults to 10^-20


##Member methods

###```__init___(self, wavelen, sb)```

####Arguments
- ```wavelen``` -- a numpy array containing the wavelength grid in nanometers
- ```sb``` -- a numpy array containing the probability of a photon at a given wavelength being detected.

####Results
- automatically set ```_ab_norm```

###```readThroughput(filename)```
This will be a class method so that it can be called without instantiating ```Bandpass``` first.

####Arguments
- ```filename``` -- a string denoting the name of the throughput file to be read

####Results
- reads in ```filename```, expecting two columns: wavelength in nanometers and sb as a function of wavelength.
- returns an intantiation of ```Bandpass```

###```__mul__(self, other)```

####Arguments
- ```other``` -- another ```Bandpass```

####Results
- multiplies together two ```Bandpasses``` by resampling the ```Bandpasses``` onto the finest ```wavelen``` grid that can accommodate both and multiplying their ```sb``` arrays together
- Returns a new ```Bandpass``` instantiation that is the result of multiplying the two ```Bandpasses``` by each other.
- Raise an exception of there is a region of non-overlap in the two ```Bandpass```es ```wavelen``` arrays.  The non-overlap region must contain ```sb```>```_threshold``` to actually raise an exception.
- Set ```self._work_wavelen``` and ```self._work_sb``` to ```None```.

###```readThroughputList(fileNameList)```
This will be a class method so that it can be called without instantiating ```Bandpass``` first.

####Arguments
- ```fileNameList``` -- a list of file names

####Results
- Read each file in ```fileNameList```, expecting each file to contain two columns: wavelength in nanometers and sb as a function of wavelength.
- Multiply the resutling ```Bandpass```es together.
- Return the ```Bandpass``` that is the product of all of the ```Bandpass```es specified by the files in ```fileNameList```.

###```_integrateWorkArrays(self, input_wavelen, input_fn)```
####Arguments
- ```input_wavelen``` -- a numpy array defining wavelength grid in nanometers
-```input_fn``` -- a numpy array defining a function on ```input_wavelen```

####Results
- Define ```local_max = min(self._wavelen.max(), input_wavelen.max())```
- Define ```local_min = max(self._wavelen.min(), input_wavelen.min())```
- Check to see if ```self._work_wavelen``` is a numpy array with the same grid spacing as ```input_wavelen``` on the interval [```min```, ```max```].  If not, resample ```self._wavelen``` and ```self._sb``` to meet that criterion and store the results in ```self._work_wavelen``` and ```self._work_sb```.
- Integrate ```input_fn*self._work_sb``` over ```self._work_wavelen```.  If there is a region of non-overlap between ```input_wavelen``` and ```self._work_wavelen```, check to see whether or not ```exists(self._work_sb, input_fn)>self._threshold``` in the region of non-overlap.  If ```True```, pad ```input_fn*self._work_sb``` with ```self._fill_value``` and raise a warning.  If ```False```, ignore the region of non-overlap (i.e. assume that ```self._work_sb*input_fn``` would be zero in the region of non-overlap, anyway).
- Return the result of the integration.

###```calcFluxAB(self, inputSed)```
####Arguments
- ```inputSed``` -- an instantiation of ```Sed```

####Results
- Call ```self._integrateWorkArrays(inputSed._wavelen, inputSed._fnu/inputSed._wavelen)```.  Divide by ```_ab_norm```.  Return the result.  This will be the AB flux in units of ```maggies```

###```calcMagAB(self, inputSed)```
####Arguments
- ```inputSed``` -- an instantiation of ```Sed```

####Results
- Call ```calcFluxAB``` to get the AB flux of ```inputSed``` over this ```Bandpass```
- Return -2.5*log_10(flux_AB), which is the AB magnitude of ```inputSed``` in this ```Bandpass```.

###```calcADU(self, inputSed, photParams)```
####Arguments
- ```inputSed``` -- an instantiation of ```Sed```
- ```photParams``` -- an instantiation of the ```PhotometricParameters``` class carrying data about the photometric response of the instrument.

####Results
- Call ```self._integrateWorkArrays(inputSed._wavelen, inputSed._fnu/inputSed._wavelen)```.  Mutliply by constants stored in ```photParams``` to convert to the number of ADU counts resulting from observing ```inputSed``` through the current ```Bandpass```.
- Return the calculated number of ADU.


###```calcZeroPoint(self, photParams)```
Return the 'instrumental zero point', i.e. the magnitude of an F_nu-flat SED that produces one ADU per second.
####Arguments
- ```photParams``` -- an instantiation of the ```PhotometricParameters``` class carrying data about the photometric response of the instrument.

####Results
- Instantiate ```flatSed```, an ```Sed``` with a flat ```fnu```.
- Call ```flatSed.normalizeADU(1, self)``` to normalize ```fnu``` to produce one ADU per second.
- Return ```self.calcMagAB(flatSed)```

#Sed class

##Member variables

##Member methods
