# iSpace 

**Xiangning Chu (xiangning.chu@colorado.edu**)

iSpace is a Python package that aims to provide access to a few well-known models to the space science comminuty. It is still under active development. The models include:

- Geopack
- Tsyganenko magnetospheric models (T89, T96, T01, and T04)
- Substorm current wedge inversion (under development)
- AACGM (under development)

## Installation

To install iSpace, simply download the Github repository and run the setup.py

```
git clone https://github.com/xnchu/iSpace.git
python setup.py install
```

The iSpace package contains a lot of Fortran codes, which are compiled into a single dynamic library module file using Numpy. It turns out to be difficult to compile the files across different platforms without proper setup, which requires a lot of efforts. Therefore, the iSpace package is pre-compiled under a number of popular platforms (Windows 64, Mac M1 arm, Mac Intel x86_64, and Linux 64). The corresponding module files (.pyd under Windows, and .so under Mac and Linux) are stored under /iSpace/libs/. The setup.py simply copy the corresponding module file to the folder of python packages (site-packages), so that iSpace can be imported globally. 



## Magnetospheric magnetic field model

The Earth's magnetospheric field model and [Geopack](https://geo.phys.spbu.ru/~tsyganenko/modeling.html), developed by N. A. Tsyganenko, is widely used in the space science community. The iSpace package provides an intereface to grant easy access to Geopack. The iSpace package do not calim any rights to these routines in Geopack. 

[The IDL version of Geopack](https://ampere.jhuapl.edu/tools/) is developed by H. Korth, and it is widely used with [SPEDAS](http://themis.ssl.berkeley.edu/software.shtml). 

### Example

Example1: Calculate the magnetic field using T96 model. 

```python
iyear = 2010 # year
idoy = 40 # day of year
ihour = 5 # hour
imin = 8 # minute
isec = 30 # second
xgsm = -6.15 # x in GSM coordinate (Re)
ygsm = 2.00 # y in GSM coordinate (Re)
zgsm = 1.50 # z in GSM coordinate (Re)
parmod = np.array([3.0, -20.0, 3.0, -5.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]) # The 10 parameters for T96 models. 
ispace.geopack_recalc(iyear,idoy,ihour,imin,isec)
bxgsm, bygsm, bzgsm = ispace.geopack_t96(parmod, xgsm, ygsm, zgsm)
print('The magnetic field (nT) in GSM coordinates:', bxgsm, bygsm, bzgsm)
```

 Example2: Trace the magnetic field lines from a specific point. 

```python
iyear = 2010 # year
idoy = 40 # day of year
ihour = 5 # hour
imin = 8 # minute
isec = 30 # second
xgsm = -6.15 # x in GSM coordinate (Re)
ygsm = 2.00 # y in GSM coordinate (Re)
zgsm = 1.50 # z in GSM coordinate (Re)
dir1 = 1.0 # tracing direction 
          # 1: antiparallel to the magnetic field, toward the southern hemisphere
          # -1: parallel to the magnetic field, toward the northern hemisphere
rlim = 40.0 # maximum radial distance (Re)
r0 = 110.0/6378.0+1.0 # Radius of the end point (Re). E.g., the ionosphere radius is (110.0/6378.0+1.0)=1.0174
iopt = 3.0 # The KP input for T89 model
parmod = np.array([3.0, -20.0, 3.0, -5.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]) # The 10 parameters for T96, T01, T04 models. 
exname = 'T96' # The external field model name (T89, T96, T01, T04)
inname = 'IGRF_GSM' # The internal field model name (IGRF_GSM, DIP)
ispace.geopack_recalc(iyear, idoy, ihour, imin, isec)
xfs, yfs, zfs, xxs, yys, zzs, Ls = ispace.geopack_trace(xgsm, ygsm, zgsm, dir1, rlim, r0, iopt, parmod, exname, inname)
xxs = xxs[0:Ls]
yys = yys[0:Ls]
zzs = zzs[0:Ls]
xfn, yfn, zfn, xxn, yyn, zzn, Ln = ispace.geopack_trace(xgsm, ygsm, zgsm, -dir1, rlim, r0, iopt, parmod, exname, inname)
xxn = xxn[0:Ln]
yyn = yyn[0:Ln]
zzn = zzn[0:Ln]
# xf, yf, zf: the footpoint of the field line, n for northern hemisphere, s for southern hemisphere
# xx, yy, zz: the coordinates of the field line
# L: the length of the field line. The rest of the arrays are padded with zeros in fortran.
print('Footpoint of the field line (Re) in the southern hemisphere:', xfs, yfs, zfs)
print('Footpoint of the field line (Re) in the northern hemisphere:', xfn, yfn, zfn)
```

### Testing

All the functions in iSpace.geopack is tested against the IDL Geopack for 1000 times. The results from IDL geopack is saved in an IDL sav file at ./iSpace/tests/test_geopack.sav

Test script is available at ./iSpace/tests/test_geopack_to_sav.py

All tests passed. 



## Inversion technique for Substorm Current Wedge (SCW)

The substorm current wedge is one of the most important currents during magnetospheric substorms. I have developed an inversion technique to obtain the SCW parameters, e.g., intensity and location of the field-aligned currents (FAC) using ground-based magnetic field data at Earth's midlatitudes. It is under development and will be published soon. 



## References:



Chu, X., et al. (2014), Development and validation of inversion technique for substorm current wedge using ground magnetic field data, *J. Geophys. Res. Space Physics*, 119, 1909– 1924, doi:[10.1002/2013JA019185](https://doi.org/10.1002/2013JA019185).



Tsyganenko, N. A., A Magnetospheric Magnetic Field Model With a Warped Tail Current Sheet, Planet. Space Sci., 37, 5-20, 1989. Doing: [10.1016/0032-0633(89)90066-4](https://doi.org/10.1016/0032-0633(89)90066-4)

 

Tsyganenko, N. A. (1995), Modeling the Earth's magnetospheric magnetic field confined within a realistic magnetopause, *J. Geophys. Res.*, 100( A4), 5599–5612, doi:[10.1029/94JA03193](https://doi.org/10.1029/94JA03193).

 

Tsyganenko, N. A., and Peredo, M. (1994), Analytical models of the magnetic field of disk-shaped current sheets, *J. Geophys. Res.*, 99( A1), 199– 205, doi:[10.1029/93JA02768](https://doi.org/10.1029/93JA02768).

 

Tsyganenko, N. A., A model of the magnetosphere with a dawn-dusk asymmetry, 1, Mathematical structure, *J. Geophys. Res.*, 107( A8), doi:[10.1029/2001JA000219](https://doi.org/10.1029/2001JA000219), 2002.

 

Tsyganenko, N. A., A model of the near magnetosphere with a dawn-dusk asymmetry, 2, Parameterization and fitting to observations, *J. Geophys. Res.*, 107( A8), doi:[10.1029/2001JA000220](https://doi.org/10.1029/2001JA000220), 2002.

 

Tsyganenko, N. A., Singer, H. J., and Kasper, J. C. (2003), Storm-time distortion of the inner magnetosphere: How severe can it get? *J. Geophys. Res.*, 108, 1209, doi:[10.1029/2002JA009808](https://doi.org/10.1029/2002JA009808), A5.

 

Tsyganenko, N. A., and Sitnov, M. I. (2005), Modeling the dynamics of the inner magnetosphere during strong geomagnetic storms, *J. Geophys. Res.*, 110, A03208, doi:[10.1029/2004JA010798](https://doi.org/10.1029/2004JA010798).

 

