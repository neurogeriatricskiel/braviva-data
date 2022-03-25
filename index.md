## Data organization

The overall theme of the BraViva study is the:
> *Quantitative assessment of motor fluctuations and bradykinesia in Parkinson’s disease using wearables*.

On these pages we describe the organization of the BraViva dataset. For the time being, we focus only on data from the [Movisens Move 4](https://docs.movisens.com/Sensors/Move4/) sensors.

### Folder structure
```
.
├── BraViva/
│   ├── rawdata/
│   │   ├── sub-COKI70030/
│   │   │   └── ses-T2/
│   │   │       └── motion/
│   │   │           ├── sub-COKI70030_ses-T2_run-20200228.mat
│   │   │           ├── sub-COKI70030_ses-T2_run-20200229.mat
│   │   │           ├── ...
│   │   │           └── sub-COKI70030_ses-T2_run-20200306.mat
│   ├── derivatives/
│   │   ├── sub-COKI70030/
│   │   │   └── ses-T2/
│   │   │       ├── motion/
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200228_channels.tsv
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200228_motion.mat
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200228_channels.tsv
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200229_motion.mat
│   │   │       │   ├── ...
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200306_channels.tsv
│   │   │       │   └── sub-COKI70030_ses-T2_run-20200306_motion.mat
│   │   │       ├── pressure/
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200228_channels.tsv
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200228_pressure.mat
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200229_channels.tsv
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200229_pressure.mat
│   │   │       │   ├── ...
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200306_channels.tsv
│   │   │       │   └── sub-COKI70030_ses-T2_run-20200306_pressure.mat
│   │   │       ├── temperature/
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200228_channels.tsv
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200228_temperature.mat
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200229_channels.tsv
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200229_temperature.mat
│   │   │       │   ├── ...
│   │   │       │   ├── sub-COKI70030_ses-T2_run-20200306_channels.tsv
│   │   │       │   └── sub-COKI70030_ses-T2_run-20200306_temperature.mat
│   │   │       └── sub-COKI70030_ses-T2_scans.tsv
└── ...
```
Obviouly, this structure is preserved for other subjects as well.

#### Raw data
In the `rawdata/` folder, the corresponding `.mat` files contain a single variable namely `data`. `data` is a MATLAB struct that contains all relevant data and the corresponding meta-data.
```matlab
>>> load('./BraViva/rawdata/sub-COKI70030/sess-T2/motion/sub-COKI70030_ses-T2_run-2020028.mat', 'data')
>>> data
```
| tracked_point | data       | acq_time_start
| ------------- | ---------- | --------------
| ankle         | 1x4 struct | 1x1 datetime
| lowBack       | 1x4 struct | 1x1 datetime
| wrist         | 1x4 struct | 1x1 datetime

For each of the tracked points, the data collected with the corresponding IMU is saved in a struct itself:
```matlab
>>> data(1).data
```
| type          | unit         | sampling_frequency | data
| ------------- | ------------ | ------------------ | ----
| angularRate   | dps          | 64                 | Nx3 double
| acc           | g            | 64                 | Nx1 double
| press         | Pa           |  8                 | Nx1 double
| temp          | Grad Celsius |  1                 | Nx3 double

with $N$ number of timesteps. 

```matlab
>>> ts_init = data(1).acq_time_start; % initial timestamp
>>> ts = ts_init + seconds((0:size(data(1).data(1).data,1)-1)'/data(1).data(1).sampling_frequency); % timestamps
>>> figure; plot(ts, data(1).data(1).data); % plots the angular rate data of the ankle sensor
```

#### Derivatives
In the `derivatives/` folder, the data are organized more alike the [BIDS format](https://bids-standard.github.io/bids-starter-kit/).
The `sub-<label>_ses-T2_scans.tsv` file contains relevant information on the data files, and the initial timestamp:
| filename | acq_time_start
| -------- | --------------
| motion/sub-COKI12345_ses-T2_run-20190926_motion.mat | 26-Sep-2019 09:00:02
| pressure/sub-COKI12345_ses-T2_run-20190926_pressure.mat | 26-Sep-2019 09:00:02
| ... | ...

Then the `motion/sub-COKI12345_ses-T2_run-20190926_channes.tsv` file contains informations about the data, namely:
| name           | type   | component | tracked_point | units | sampling_frequency
| -------------- | ------ | --------- | ------------- | ----- | ------------------
| ankle_ACC_x    | ACC    | x         | ankle         | g     | 64
| ankle_ACC_y    | ACC    | y         | ankle         | g     | 64
| ...            | ...    | ...       | ...           | ...   | ...
| wrist_ANGVEL_z | ANGVEL | z         | wrist         | dps   | 64

and the data itself is stored in the `motion/sub-COKI12345_ses-T2_run-20190926_motion.mat` file, in a `NxD double` array called `motion`, with `N` number of time steps, and `D` number of channels:
```matlab
>>> load('motion/sub-COKI12345_ses-T2_run-20190926_motion.mat', 'motion');
>>> motion
```
