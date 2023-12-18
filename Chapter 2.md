## Chapter 2. Data Processing for the Experimental Lab Specimen

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This chapter focuses on elucidating the procedures employed for processing Ground Penetrating Radar (GPR) data obtained from our laboratory specimen. <a href="https://doi.org/10.1016/j.conbuildmat.2018.08.127">[3]</a> The structure under examination within our laboratory setting replicates a section of a concrete bridge with embedded reinforcement bars (rebar). Our GPR data processing involves implementing advanced techniques for time-zero correction and F-K migration, ensuring a precise representation of the rebar configuration within the specimen.

### Step 1. Read the saved CSV files
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;We need to read the saved CSV files to process further. Let’s use the read_csv function to define two Pandas data frames. After that, we set each row of config data as a local variable in Python.

```
user_directory = "C:/Your_directory"

df_1, df_2 = read_csv(user_directory)

for index, row in df_2.iterrows():

    variable_name = row['Unnamed: 0']
    
    locals()[variable_name] = row['config'] 
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Let’s investigate the CSV data in detail. The data frame 1 `df_1` is actual GPR data (512 Rows × 332 Cols). The 512 rows are the “depth” within the investigated underground subsurface (also considered as wavelet traveling time), and 332 columns are the number of scans, corresponding to a distinct scan instance where a radar wavelet is emitted and recorded by the antenna while the GPR machine traverses along the survey line.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The data frame 2 `df_2` is a configuration setting of GPR machine (24 Rows × 2 Cols). Here we discuss some of important parameters among them, but readers are redirected to the GPR manufacturer webpage for further details: GSSI SIR 3000 Manual [https://www.geophysical.com/wp-content/uploads/2017/10/GSSI-SIR-3000-Manual.pdf](https://www.geophysical.com/wp-content/uploads/2017/10/GSSI-SIR-3000-Manual.pdf). Here we discuss 7 important parameters in the configuration setting: `rh_nsamp`, `rhf_sps`, `rhf_spm`, `rhf_position`, `rhf_range`, `rhf_espr`, and `rhf_depth`.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The parameter `rh_nsamp` represents the number of samples in depth dimension (equivalent to the row dimension of GPR data 512). Imagine a grid interface of the underground subsurface to save the wavelet in discrete form (to save as data). The higher number of samples provide higher resolution, but larger size of the data. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The `rhf_sps` and `rhf_spm` refer to “scans-per-second” and “scans-per-meter”. Here scans mean the column dimension of the GPR data (332 in our data). Both of parameters indicate how many scans are obtained per unit time or distance. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The `rhf_position` and `rhf_range` are parameters for indicating position and range of the GPR scans in nano seconds (ns). For example, if `rhf_position = 0`, this means that the starting point for measuring positions in your GPR data is at the initial time when the radar pulse is sent. In other words, the measurement of positions starts from the moment the radar pulse is emitted. Similarly, if `rhf_range = 8`, it indicates the time it takes for the radar signals to travel to the subsurface and return takes 8 ns. By knowing the speed of electromagnetic waves and average dielectric constant, this time can be converted to a distance measurement.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The `rhf_espr` is average dielectric constant. It accounts for the varying properties of the subsurface materials. Different materials have different dielectric constants, and accurate knowledge of this parameter is crucial for correctly interpreting the travel time of radar signals and converting it into depth. Table 1 shows the relative dielectric constant for various materials. The `rhf_depth` is depth in meters, indicating how deep into the subsurface the radar signals penetrate underground.

Table 1. Relative Dielectric Permittivity for Different Materials. <a href="https://infotechnology.fhwa.dot.gov/bridge/">[4]</a>

| Material           | Dielectric Constant |
|--------------------|---------------------|
| Air                | 1                   |
| Clay               | 25-40               |
| Concrete           | 8-10                |
| Crushed base       | 6-8                 |
| Gravel             | 4-7                 |
| Hot mix asphalt    | 4-8                 |
| Ice                | 4                   |
| Insulation board   | 2-2.5               |
| Sand               | 4-6                 |
| Silt               | 16-30               |
| Silty sand         | 7-10                |
| Water (fresh)      | 81                  |

