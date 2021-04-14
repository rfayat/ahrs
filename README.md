# AHRS: Attitude and Heading Reference Systems

![GitHub Workflow Status](https://img.shields.io/github/workflow/status/Mayitzin/ahrs/Build%20Python%20Package)
![docs](https://readthedocs.org/projects/ahrs/badge/?version=latest)
![PyPI - License](https://img.shields.io/pypi/l/ahrs)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/ahrs)
![PyPI](https://img.shields.io/pypi/v/ahrs)
![Codacy Badge](https://api.codacy.com/project/badge/Grade/bc366c601ed44e12b233218dd37cd32c)
![PyPI Downloads](https://pepy.tech/badge/ahrs)

AHRS is a collection of functions and algorithms in pure Python used to estimate the orientation of mobile systems.

Orginally, an [AHRS](https://en.wikipedia.org/wiki/Attitude_and_heading_reference_system) is a set of orthogonal sensors providing attitude information about an aircraft. This field has now expanded to smaller devices, like wearables, automated transportation and all kinds of systems in motion.

This package's focus is **fast prototyping**, **education**, **testing** and **modularity**. Performance is _NOT_ the main goal. For optimized implementations there are endless resources in C/C++ or Fortran.

AHRS is compatible with **Python 3.6** and newer.

## A few notes about this fork
This repository is a fork from [ahrs](https://github.com/Mayitzin/ahrs).

As mentioned by [@Mayitzin](https://github.com/Mayitzin/ahrs) above, the intended goal of this package is not performance. I nevertheless needed to try a range of attitude estimate filters on a few hundreds recordings of >100k samples and love the unique pure python api offered by this package. As I did not really need very high performance but the computation time got a bit long, I've decided to play a bit with [numba jit-classes](http://numba.pydata.org/numba-doc/dev/user/jitclass.html), which leverages JIT compilation to accelerate python code with relatively modest revamping of the code.

This fork therefore converts a few filters [Mahony](https://ahrs.readthedocs.io/en/latest/filters/mahony.html), [Madgwick](https://ahrs.readthedocs.io/en/latest/filters/madgwick.html) and [EKF](https://ahrs.readthedocs.io/en/latest/filters/ekf.html) for the moment, to numba jit-classes. A summary of the impact on computation time on my machine **for a time series of 100k samples** using only **IMU** data or also magnetometers (**MIMU**) are available below:

|  | Pure Python (IMU / MIMU) | NumbaClass (IMU / MIMU) | Δt (IMU / MIMU) | Overhead |
| --- | --- | --- | ---|---|
| **Madgwick** | 8.93s / 15.31s | 3.33s / 4.39s |-62% / -71% |9.17s|
| **Mahony** | 14.91s / 24.08s | 5.38s / 4.90s | -64% / -80% |21.34s|
| **EKF** | 73.26s / 134.95s | 11.91s / 11.72s | -83% / -91% | 32.33s|


The gains in computation time are significative but since numba compilation adds a relatively big overhead to the computation and given the spirit in which [@Mayitzin](https://github.com/Mayitzin/ahrs) made the package (only one dependency, focus on readability of the code...), I don't think it would really make sense to open a Pull Request to [ahrs](https://github.com/Mayitzin/ahrs) but I'm happy to share the code.

## Installation

The most recommended method is to install AHRS directly from this repository to get the latest version:

```shell
git clone https://github.com/Mayitzin/ahrs.git
cd ahrs
python setup.py install
```

Or using [pip](https://pip.pypa.io) for the stable releases:

```shell
pip install ahrs
```

AHRS depends on common packages [NumPy](https://numpy.org/) and [SciPy](https://www.scipy.org/). More packages are avoided, to reduce its third-party dependency.

## New in 0.3 (release candidate)

- [Type hints](https://www.python.org/dev/peps/pep-0484/) are added.
- NumPy is now the only third-party dependency.
- The **World Magnetic Model** ([WMM](https://www.ngdc.noaa.gov/geomag/WMM/DoDWMM.shtml)) is fully implemented. It can be used to estimate all magnetic field elements on any given place of Earth for dates between 2015 and 2025.

```python
>>> from ahrs.utils import WMM
>>> wmm = WMM(latitude=10.0, longitude=-20.0, height=10.5)
>>> wmm.magnetic_elements
{'X': 30499.640469609083, 'Y': -5230.267158472566, 'Z': -1716.633311360368,
'H': 30944.850352270452, 'F': 30992.427998627096, 'I': -3.1751692563622993,
'D': -9.73078560629778, 'GV': -9.73078560629778}
```

- The _ellipsoid model_ of the **World Geodetic System** ([WGS84](https://earth-info.nga.mil/GandG/update/index.php?dir=wgs84&action=wgs84)) is included. A full implementation of the **Earth Gravitational Model** ([EGM2008](https://earth-info.nga.mil/GandG/wgs84/gravitymod/egm2008/egm08_wgs84.html)) is _NOT_ available here, but the estimation of the main and derived parameters of the WGS84 using the ellipsoid model are implemented:

```python
>>> from ahrs.utils import WGS
>>> wgs = WGS()      # Creates an ellipsoid model, using Earth's characteristics by default
>>> wgs_properties = [x for x in dir(wgs) if not (hasattr(wgs.__getattribute__(x), '__call__') or x.startswith('__'))]
>>> for p in wgs_properties:
...     print('{:<{w}}  {}'.format(p, wgs.__getattribute__(p), w=len(max(wgs_properties, key=len))))
...
a                                          6378137.0
arithmetic_mean_radius                     6371008.771415059
aspect_ratio                               0.9966471893352525
atmosphere_gravitational_constant          343591934.4
authalic_sphere_radius                     6371007.1809182055
b                                          6356752.314245179
curvature_polar_radius                     6399593.625758493
dynamic_inertial_moment_about_X            8.007921777277886e+37
dynamic_inertial_moment_about_Y            8.008074799852911e+37
dynamic_inertial_moment_about_Z            8.03430094201443e+37
dynamical_form_factor                      0.0010826298213129219
equatorial_normal_gravity                  9.78032533590406
equivolumetric_sphere_radius               6371000.790009159
f                                          0.0033528106647474805
first_eccentricity_squared                 0.0066943799901413165
geometric_dynamic_ellipticity              0.003258100628533992
geometric_inertial_moment                  8.046726628049449e+37
geometric_inertial_moment_about_Z          8.073029370114392e+37
gm                                         398600441800000.0
gravitational_constant_without_atmosphere  398600098208065.6
is_geodetic                                True
linear_eccentricity                        521854.00842338527
mass                                       5.972186390142457e+24
mean_normal_gravity                        9.797643222256516
normal_gravity_constant                    0.0034497865068408447
normal_gravity_potential                   62636851.71456948
polar_normal_gravity                       9.832184937863065
second_degree_zonal_harmonic               -0.00048416677498482876
second_eccentricity_squared                0.006739496742276434
w                                          7.292115e-05
```

It can be used, for example, to estimate the normal gravity acceleration (in m/s^2) at any location on Earth.

```python
>>> wgs.normal_gravity(50.0, 1000.0)    # Normal gravity at latitude = 50.0 °, 1000 m above surface
9.807617683884756
```

Setting the fundamental parameters (`a`, `f`, `GM`, `w`) yields a different ellipsoid. For the moon, for instance, we build a new model:

```python
>>> moon_a = ahrs.MOON_EQUATOR_RADIUS
>>> moon_f = (ahrs.MOON_EQUATOR_RADIUS-ahrs.MOON_POLAR_RADIUS)/ahrs.MOON_EQUATOR_RADIUS
>>> moon_gm = ahrs.MOON_GM
>>> moon_w = ahrs.MOON_ROTATION
>>> moon = WGS(a=moon_a, f=moon_f, GM=moon_gm, w=moon_w)
>>> moon.normal_gravity(10.0, h=500.0)  # Gravity on moon at 10° N and 500 m above surface
1.6239259827292798
>>> moon.is_geodetic     # Only the Earth is geodetic
False
```

- The [International Gravity Formula](http://earth.geology.yale.edu/~ajs/1945A/360.pdf) and the EU's [WELMEC](https://www.welmec.org/documents/guides/2/) normal gravity reference system are also implemented.

```python
>>> ahrs.utils.international_gravity(50.0)       # Latitude = 50° N
9.810786421572386
>>> ahrs.utils.welmec_gravity(50.0, 500.0)       # Latitude = 50° N,   height above sea = 500 m
9.809152687885897
```

- New class `DCM` (derived from `numpy.ndarray`) for orientation/rotation representations as 3x3 Direction Cosine Matrices.

```python
>>> from ahrs import DCM
>>> R = DCM(x=10.0, y=20.0, z=30.0)
>>> type(R)
<class 'ahrs.common.dcm.DCM'>
>>> R.view()
DCM([[ 0.81379768 -0.46984631  0.34202014],
     [ 0.54383814  0.82317294 -0.16317591],
     [-0.20487413  0.31879578  0.92541658]])
>>> R.inv     # or R.I
array([[ 0.81379768  0.54383814 -0.20487413]
       [-0.46984631  0.82317294  0.31879578]
       [ 0.34202014 -0.16317591  0.92541658]])
>>> R.log
array([0.26026043, 0.29531805, 0.5473806 ])
>>> R.to_axisangle()        # Axis in 3D NumPy array, and angle as radians
(array([0.38601658, 0.43801381, 0.81187135]), 0.6742208510527136)
>>> R.to_quaternion()
array([0.94371436, 0.12767944, 0.14487813, 0.26853582])
>>> R.to_quaternion(method='itzhack', version=2)
array([ 0.94371436, -0.12767944, -0.14487813, -0.26853582])
```

- New class `QuaternionArray` to simultaneously handle an array with more quaternions at once.

```python
>>> Q = QuaternionArray(np.random.random((3, 4))-0.5)
>>> Q.view()
QuaternionArray([[ 0.31638467,  0.59313477, -0.62538687, -0.39621099],
                 [ 0.24973118, -0.37958194, -0.67851278, -0.57721079],
                 [-0.44643469,  0.17200957, -0.72678553,  0.49284031]])
>>> Q.w
array([ 0.31638467,  0.24973118, -0.44643469])
>>> Q.to_DCM()
array([[[-0.09618377, -0.49116723, -0.86573866],
        [-0.99258756, -0.017584  ,  0.1202528 ],
        [-0.07428738,  0.8708878 , -0.48583519]],

       [[-0.58710377,  0.80339746,  0.09930598],
        [ 0.22680733,  0.04549051,  0.97287669],
        [ 0.77708918,  0.5937029 , -0.20892408]],

       [[-0.54221755,  0.19001389,  0.81847104],
        [-0.69007015,  0.45504228, -0.56279633],
        [-0.47937805, -0.86996048, -0.115609  ]]])
>>> Q.conjugate()
array([[ 0.31638467, -0.59313477,  0.62538687,  0.39621099],
       [ 0.24973118,  0.37958194,  0.67851278,  0.57721079],
       [-0.44643469, -0.17200957,  0.72678553, -0.49284031]])
>>> Q.average()
array([ 0.19537239,  0.17826049, -0.87872408, -0.39736232])
```

- New submodule `frames` to represent the position of an object in different reference frames.
- [Metrics](https://ahrs.readthedocs.io/en/latest/metrics.html) for rotations in 3D spaces using quaternions and direction cosine matrices.
- New operations, properties and methods for class `Quaternion` (now also derived from `numpy.ndarray`)
- A whole bunch of [new constant values](https://ahrs.readthedocs.io/en/latest/constants.html) (mainly for Geodesy) accessed from the top level of the package.
- Docstrings are improved with further explanations, references and equations whenever possible.

## More Attitude Estimators

One of the biggest improvements in this version is the addition of many new attitude estimation algorithms.

All estimators are refactored to be consistent to the original articles describing them. They have in-code references to the original equations, so that you can follow the original articles along with the code.

Implemented attitude estimators are:

| Algorithm     | Gyroscope | Accelerometer | Magnetometer |
|---------------|:---------:|:-------------:|:------------:|
| AQUA          | YES       | YES           | Optional     |
| Complementary | YES       | YES           | Optional     |
| Davenport's   | NO        | YES           | YES          |
| EKF           | YES       | YES           | YES          |
| FAMC          | NO        | YES           | YES          |
| FLAE          | NO        | YES           | YES          |
| Fourati       | YES       | YES           | YES          |
| FQA           | NO        | YES           | Optional     |
| Integration   | YES       | NO            | NO           |
| Madgwick      | YES       | YES           | Optional     |
| Mahony        | YES       | YES           | Optional     |
| OLEQ          | NO        | YES           | YES          |
| QUEST         | NO        | YES           | YES          |
| ROLEQ         | NO        | YES           | YES          |
| SAAM          | NO        | YES           | YES          |
| Tilt          | NO        | YES           | Optional     |
| TRIAD         | NO        | YES           | YES          |

More Estimators are still a *Work In Progress*, or *planned* to be added in the future.

| Algorithm | Gyroscope | Accelerometer | Magnetometer | Status  |
|-----------|:---------:|:-------------:|:------------:|:-------:|
| ESOQ      | NO        | YES           | YES          | WIP     |
| ESOQ-2    | NO        | YES           | YES          | WIP     |
| FKF       | NO        | YES           | YES          | WIP     |
| FCF       | NO        | YES           | YES          | Planned |
| FOAM      | NO        | YES           | YES          | Planned |
| GDA-LKF   | YES       | YES           | YES          | Planned |
| MAGYQ     | YES       | YES           | YES          | Planned |
| QRAUKF    | YES       | YES           | YES          | Planned |
| REQUEST   | NO        | YES           | YES          | Planned |
| Sabatini  | YES       | YES           | YES          | Planned |
| SOLEQ     | NO        | YES           | YES          | Planned |

To use the sensor data to estimate the attitude simply pass the data to a desired estimator, and it will automatically estimate the quaternions with the given parameters.

```python
>>> attitude = ahrs.filters.Madgwick(acc=acc_data, gyr=gyro_data)
>>> attitude.Q.shape
(6959, 4)
```

Some algorithms allow a finer tuning of its estimation with different parameters. Check their documentation to see what can be tuned.

```python
>>> attitude = ahrs.filters.Madgwick(acc=acc_data, gyr=gyro_data, mag=mag_data, gain=0.1, frequency=100.0)
```

Speaking of documentation...

## Documentation

A comprehensive documentation, with examples, is now available in
[Read the Docs](https://ahrs.readthedocs.io).

## Note for future versions

`ahrs` moves away from plotting and data handling submodules to better focus in the algorithmic parts. Submodules `io` and `plot` are not built in the package anymore and, eventually, will be entirely removed from the base code.

This way you can also choose your favorite libraries for data loading and visualization. This also means, getting rid of its dependency on `matplotlib` too.
