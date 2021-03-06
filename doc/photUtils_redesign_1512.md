# Bandpass class


## Member variables

### ```_wavelen```
A numpy array storing wavelength grid in nanometers.  This will be accessible through the ```@property``` ```wavelen```.

### ```_sb```
A numpy array storing the throughput curve as a function of ```wavelen```.  This will be the probability of a photon being observed.  This will be accessible through the ```@property``` ```sb```.

### ```_ab_norm```
This will be the normalization value applied when calculating AB fluxes.  It is the integral over ```wavelen``` of ```sb```/```wavelen``` multiplied by 3631 Janskies.  It will be set by ```__init___``` and reset whenever ```wavelen``` or ```sb``` change.

### ```_work_wavelen```
Initialized as ```None```.  Whenever ```wavelen``` needs to be resampled to match an ```Sed```, the resampled ```wavelen``` will be stored here.  That way, the original throughput information is not lost.

### ```_work_sb```
Initialized as ```None```.  Whenever ```wavelen``` needs to be resampled to match an ```Sed```, the resampled ```sb``` will be stored here.  That way, the original throughput information is not lost.

### ```_fill_value```
If integrating this ```Bandpass``` over an ```Sed``` with a region of non-overlap in wavelength (either the ```Bandpass``` does not cover the ```Sed``` or vice-versa), this value will be used to fill in the ```sb*flux``` (or whatever function is being integrated) in the region of non-overlap.  If ```fill_value``` is ```numpy.NaN```, then the resulting flux or magnitude for the whole ```Bandpass``` will be ```numpy.NaN```.  If ```fill_value``` is zero, then the resulting flux or magnitude will only include the region in wavelength space where the ```Bandpass``` and ```Sed``` do overlap; the non-overlapping region will be ignored.  ```fill_value``` will default to ```numpy.NaN```.  This will be accessible (and settable) through the ```@property``` ```fill_value```.

### ```_threshold```
If a region of non-overlap is present (see above), ```Bandpass``` will compare the absolute value of the existing function (either ```self._sb``` or a flux-like function from an ```Sed```) to the maximum value of that function.  If the resulting ratio is less than ```_threshold```, the region of non-overlap will be treated as unimportant.  This will be accessible (and settable) through the ```@property``` ```threshold```.


## Member methods

### ```__init___(self, wavelen, sb, fill_value=numpy.NaN, threshold=1.0e-10)```

#### Arguments
- ```wavelen``` -- a numpy array containing the wavelength grid in nanometers
- ```sb``` -- a numpy array containing the probability of a photon at a given wavelength being detected.
- ```fill_value``` -- see above
- ```threshold``` -- see above

#### Results
- Set ```self._wavelen = numpy.copy(wavelen)```
- Set ```self._sb = numpy.copy(sb)```
- Set ```self._fill_value = fill_value```
- Set ```self._threshold = threshold```
- Call ```self._calcABNorm()```

###```_calcABNorm(self)```
Calculate the AB flux normalization
#### Results
- Integrate ```3631*self._sb/self._wavelen``` over ```self._wavelen```
- Store the resulting value in ```self._ab_norm```


### ```__repr__(self)```
#### Results
- Return ```'Bandpass(wavelen=%s, sb=%s, fill_value=%s, threshold=%s)' % (repr(self.wavelen), repr(self.sb), repr(self.fill_value), repr(self.threshold))```

### ```readThroughput(filename)```
This will be a class method so that it can be called without instantiating ```Bandpass``` first.

#### Arguments
- ```filename``` -- a string denoting the name of the throughput file to be read

#### Results
- reads in ```filename```, expecting two columns: wavelength in nanometers and sb as a function of wavelength.
- returns an intantiation of ```Bandpass```

### ```__mul__(self, other)```

#### Arguments
- ```other``` -- another ```Bandpass```

#### Results
- multiplies together two ```Bandpasses``` by resampling the ```Bandpasses``` onto the finest ```wavelen``` grid that can accommodate both and multiplying their ```sb``` arrays together
- Returns a new ```Bandpass``` instantiation that is the result of multiplying the two ```Bandpasses``` by each other.
- Raise an exception if there is a region of non-overlap in the two ```Bandpass```es ```wavelen``` arrays.  The non-overlap region must contain ```abs(sb)/abs(sb.max())```>```self._threshold``` to actually raise an exception.
- Set ```self._work_wavelen``` and ```self._work_sb``` to ```None```.

### ```readThroughputList(fileNameList)```
This will be a class method so that it can be called without instantiating ```Bandpass``` first.

#### Arguments
- ```fileNameList``` -- a list of file names

#### Results
- Read each file in ```fileNameList```, expecting each file to contain two columns: wavelength in nanometers and sb as a function of wavelength.
- Multiply the resutling ```Bandpass```es together.
- Return the ```Bandpass``` that is the product of all of the ```Bandpass```es specified by the files in ```fileNameList```.

### ```_integrateWorkArrays(self, input_wavelen, input_fn)```
#### Arguments
- ```input_wavelen``` -- a numpy array defining a wavelength grid in nanometers
- ```input_fn``` -- a numpy array defining a function on ```input_wavelen```

#### Results
- Define ```local_max = min(self._wavelen.max(), input_wavelen.max())```
- Define ```local_min = max(self._wavelen.min(), input_wavelen.min())```
- Check to see if ```self._work_wavelen``` is a numpy array with the same grid spacing as ```input_wavelen``` on the interval [```min```, ```max```].  If not, resample ```self._wavelen``` and ```self._sb``` to meet that criterion and store the results in ```self._work_wavelen``` and ```self._work_sb```.
- Integrate ```input_fn*self._work_sb``` over ```self._work_wavelen```.  If there is a region of non-overlap between ```input_wavelen``` and ```self._work_wavelen```, check to see whether or not ```exists(abs(self._work_sb)/abs(self._work_sb.max()), abs(input_fn)/abs(input_fn.max()))>self._threshold``` in the region of non-overlap.  If ```True```, pad ```input_fn*self._work_sb``` with ```self._fill_value``` and raise a warning.  If ```False```, ignore the region of non-overlap (i.e. assume that ```self._work_sb*input_fn``` would be zero in the region of non-overlap, anyway).
- Return the result of the integration.

### ```calcFluxAB(self, inputSed)```
#### Arguments
- ```inputSed``` -- an instantiation of ```Sed```

#### Results
- Call ```self._integrateWorkArrays(inputSed._wavelen, inputSed._fnu/inputSed._wavelen)```.  Divide by ```_ab_norm```.  Return the result.  This will be the AB flux in units of ```maggies```

### ```calcMagAB(self, inputSed)```
#### Arguments
- ```inputSed``` -- an instantiation of ```Sed```

####Results
- Call ```calcFluxAB``` to get the AB flux of ```inputSed``` over this ```Bandpass```
- Return -2.5*log_10(flux_AB), which is the AB magnitude of ```inputSed``` in this ```Bandpass```.

### ```calcADU(self, inputSed, photParams)```
#### Arguments
- ```inputSed``` -- an instantiation of ```Sed```
- ```photParams``` -- an instantiation of the ```PhotometricParameters``` class carrying data about the photometric response of the instrument.

#### Results
- Call ```self._integrateWorkArrays(inputSed._wavelen, inputSed._fnu/inputSed._wavelen)```.  Mutliply by constants stored in ```photParams``` to convert to the number of ADU counts resulting from observing ```inputSed``` through the current ```Bandpass```.
- Return the calculated number of ADU.


### ```calcZeroPoint(self, photParams)```
Return the 'instrumental zero point', i.e. the magnitude of an F_nu-flat SED that produces one ADU per second.
#### Arguments
- ```photParams``` -- an instantiation of the ```PhotometricParameters``` class carrying data about the photometric response of the instrument.

#### Results
- Instantiate ```flatSed```, an ```Sed``` with a flat ```fnu```.
- Calculate ```adu0 = self.calcAdu(flatSed)```
- Instantiate ```new_sed = Sed(wavelen=flatSed.wavelen, fnu=flatSed.fnu/adu0)```
- Return ```self.calcMagAB(new_sed)```

# Sed class

## Member variables

### ```_wavelen```
A numpy array storing the wavelength grid in nanometers.   This will be accessible through the ```@property``` ```wavelen```.

### ```_flambda```
A numpy array storing F_lambda in erg/cm^2/s/nm.  This will be accessible through the ```@property``` ```flambda```.

### ```_fnu```
A numpy array storing F_nu in Jansky.  This will be accessible through the ```@property``` ```flambda```.


## Member methods

### ```__init___(self, wavelen=None, flambda=None, fnu=None)```

#### Arguments
- ```wavelen``` -- a numpy array specifying the wavelength grid in nanometers
- ```flambda``` -- a numpy array specifying F_lambda in ergs/cm^2/s/nm
- ```fnu``` -- a numpy array specifying F_nu in Janskys

#### Results
- Set ```self._wavelen = None```, ```self._flambda = None```, ```self._fnu = None```.
- If ```flambda``` and ```fnu``` are both specified, raise an exception.
- If ```wavelen``` is specified but both ```flambda``` and ```fnu``` are not specified, raise an exception.
- If either ```flambda``` or ```fnu``` is specified but ```wavelen``` is not specified, raise an exception.
- If ```wavelen``` is specified and ```flambda``` is specified, set ```self._wavelen = numpy.copy(wavelen)```, ```self._flambda = numpy.copy(flambda)``` and call ```self._calculateFnu()```.
- If ```wavelen``` is specified and ```fnu``` is specified, set ```self._wavelen = numpy.copy(wavelen)```, ```self._fnu = numpy.copy(fnu)```, and call ```self._calculateFlambda()```.


### ```__repr___(self)```
#### Results
- Return ```'Sed(wavelen=%s, flambda=%s)' % (repr(self.wavelen), repr(self.flambda))```

### ```readSedFlambda(fileName)```
This will be a class method so that it can be called without instantiating Sed first.
#### Arguments
- ```fileName``` -- The name of a text file containging two columns: the wavelength in nanometers and F_lambda in ergs/cm^2/s/nm

#### Results
- Open the specified file and read in ```wavelen``` and ```flambda```.
- Return ```Sed(wavelen=wavelen, flambda=flambda)```.

### ```readSedFnu(fileName)```
This will be a class method so that it can be called without instantiating Sed first.
#### Arguments
- ```fileName``` -- The name of a text file containing two columns: the wavelength in nanometers and F_nu in Janskys.

#### Results
- Open the specified file and read in ```wavelen``` and ```fnu```.
- Return ```Sed(wavelen=wavelen, fnu=fnu)```.

### ```_calculateFnu(self)```
#### Results
- Calculate ```self._fnu``` from ```self._wavelen``` and ```self._flambda```.

### ```_calculateFlambda(self)```
#### Results
- Calculate ```self._flambda``` from ```self._wavelen``` and ```self._fnu```.

### ```flatFnuSED()```
This will be a class method so you do not have to instantiate ```Sed``` before calling.
#### Results
- Create ```wavelen``` and ```fnu``` corresponding to a flat F_nu SED
- Return ```Sed(wavelen=wavelen, fnu=fnu)```

### ```_mutiplyBySED(self, other)```
Multipy the SED by another SED
#### Arguments
- ```other``` -- another Sed

#### Results
- Create  ```f1 = numpy.copy(self.flambda)```, ```f2 = numpy.copy(other.flambda)```.
- Create ```new_wavelen```: a numpy array representing the finest wavelength grid that can accommodate the overlap between ```self.wavelen``` and ```other.wavelen```.
- Resample ```f1``` and ```f2``` onto ```new_wavelen```.
- Return ```Sed(wavelen=new_wavelen, flambda=f1*f2)```

### ```_multiplyByConstant(self, cc)```
Multiply the SED by normalizing constant
#### Arguments
- ```cc``` -- a normalizing constant

#### Results
- If not ```isinstance(cc, int) or isinstance(cc, float) or isinstance(cc, numpy.float)``` raise an exception.
- Return ```Sed(wavelen=self.wavelen, flambda=cc*self.flambda)```

### ```__mul__(self, other)```
Multply the SED either by a normalizing constant or by another Sed
#### Arguments
- ```other``` -- either a normalizing constant or another Sed

#### Results
- If ```isinstance(other, Sed)``` call ```self._multiplyBySED(other)```
- If not ```isinstance(other, Sed)``` call ```self._multiplyByConstant(other)```

### ```normalizeCurrentSed(self, mm, bp)```
Normalize the current SED to have AB mangitude ```mm``` in Bandpass ```bp```.
#### Arguments
- ```mm``` -- a float.  The magnitude value desired in the ```Bandpass``` ```bp```.
- ```bp``` -- a ```Bandpass```

#### Results
- Call ```m0 = bp.calcMagAB(self)``` to find the magnitude of the current SED in ```bp```.
- Set ```self._flambda *= numpy.power(10.0, -0.4*(mm-m0))```
- Call ```self._calculateFnu()```

### ```getNormalizedSed(self, mm, bp)```
Return an SED that is identical to the current SED, except normalized to have magnitude ```mm``` in ```Bandpass``` ```bp```.
#### Arguments
- ```mm``` -- a float.  The magnitude value desired in the ```Bandpass``` ```bp```.
- ```bp``` -- a ```Bandpass```

#### Results
- Call ```m0 = bp.calcMagAB(self)``` to find the magnitude of the current SED in ```bp```.
- Return ```Sed(wavelen=self.wavelen, flambda=self.flambda*numpy.power(10.0, -0.4*(mm-m0)))```


### ```applyRedshift(self, zz, K_correct=True)```
Return an ```Sed``` corresponding to the current ```Sed``` redshifted by ```zz```
#### Arguments
- ```zz``` -- a float representing the redshift
- ```K_correct``` -- a boolean indicating whether or not to apply the K correction

#### Results
- If ```zz>0``` and ```K_correct``` is ```True``` return ```Sed(wavelen=self.wavelen*(1.0+zz), flambda=self.flambda/(1.0+zz))```
- If ```zz>0``` and ```K_correct``` is ```False``` return ```Sed(wavelen=wavelen.self.wavelen*(1.0+zz), flambda=self.flambda)```
- If ```zz<0``` and ```K_correct``` is ```True``` return ```Sed(wavelen=self.wavelen/(1.0-zz), flambda=self.flambda*(1.0-zz))```
- If ```zz<0``` and ```K_correct``` is ```False``` return ```Sed(wavelen=self.wavelen/(1.0-zz), flambda=self.flambda)```

# Use Cases:

## Unnormalized Bandpasses
Very often the throughput files are unnormalized, and each observation is accompanied with a 'zero point' and a magnitude system (usually the same). The zero point is used to set the normalization. The `BandPass.fluxAB` function would be equal to the flux with zeropoint 0, and magsys 'ab'. Can we generalize that function to take in these additional optional parameters?

Also, as I understand it, an unnormalized ` -Sb` would still give correct results. So, maybe it should be defined as being proportional to the probability?
 
## Lists of Similarly sampled Seds with different observing conditions
Consider finding the flux/ flux uncertainty of the same object (transient or not) in a list of observations. If the object is a static object its Sed will be the same, but the bands might be different, and the fivesigmadepth is different leading to different values of flux uncertainty. For transients like SN, the seds will be different, but sampled in the same way. Is there a recommeded way of using these procedures so that the checks/work-arrays can be done once for such SEDs ?

## List of potentially differently sampled Seds with the same observing conditions:
As in an instance Catalog. Here the thing that is the same is the fivesigma depth. 
