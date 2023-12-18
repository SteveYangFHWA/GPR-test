## Chapter 3. Data Processing for the Mississippi Bridge Data

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Here we process GPR data from the actual Mississippi bridge to locate rebar configuration. The bridge name is I-10 over CEDAR LAKE ROAD, built-in 1970. The bridge type is a Prestressed Concrete Girder/Beam. The length of the bridge is 242.50 and the width is 59.40 ft, respectively.


<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/657d0502-d540-45d3-a4e5-3d9c5dfbcaa7" alt="image">
</p>

Figure 11. Bridge location map of I-10 over Cedar Lake Road in Mississippi.

### Step 1. Outlier removal via interquartile range (IQR) method

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;With the same `read_csv` function from the previous chapter, we are making 2 Pandas data frame: GPR data and configuration. We noticed that there are some outlier values on the GPR data, so we removed them by the interquartile range (IQR) method. The IQR method is a statistical technique used to identify and remove outliers from a dataset. It involves calculating the range between the first quartile (Q1) and the third quartile (Q3) of the data distribution. Outliers are then identified and removed based on a specified multiplier of the IQR. This method is effective in addressing skewed or non-normally distributed data by focusing on the central portion of the dataset. A more detailed explanation is in our code description of the `Interquartile_Range`.


<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/56ae9f68-382e-4067-845b-175ed992b692" alt="image">
</p>

Figure 12. Outlier control with IQR method. The df_1 is the raw data and the IQR_df_1 is the processed data. The red box shows how the outlier value changed.

### Step 2. Gain
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;We observed that the GPR signal isn't sufficiently clear for processing (see Figure 14 (a) and (c)), likely because the first peak amplitude significantly outweighs other signals. This disparity could be attributed to the GPR settings or signal attenuation. To address this issue, we employ a gain function to better highlight the reflected signal. We introduce two methods, namely power gain and exponential gain, <a href="https://emanuelhuber.github.io/RGPR/02_RGPR_tutorial_basic-GPR-data-processing/">[12]</a> to enhance the clarity of the reflected signal. The power gain function is defined as follows,

$$
x_g(t) = x(t) \cdot t^ \alpha.
$$

The signal $x(t)$ is multiplied by $t^ \alpha$, where $\alpha$ is the exponent. In the default case, $\alpha$ is set to 1. The effect of the power gain is to amplify the signal based on a power-law relationship with time $t$. This can be useful for emphasizing specific features of the signal.
