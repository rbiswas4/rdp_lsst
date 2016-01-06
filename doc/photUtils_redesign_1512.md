#Bandpass class


##Member variables

###wavelen
The wavelength grid in nanometers.  Stored as a numpy array.

###sb
The throughput curve as a function of ```wavelen```.  This will be the probability of a photon being observed.  Stored as a numpy array.

###_work_wavelen
Initialized as ```None```.  Whenever ```wavelen``` needs to be resampled to match an ```Sed```, the resampled ```wavelen``` will be stored here.  That way, the original throughput information is not lost.

##Member methods


#Sed class

##Member variables

##Member methods
