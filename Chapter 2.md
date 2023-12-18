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


### Step 2. Time-zero correction

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Time-zero correction aligns multiple A-scans vertically. When the reflected signal is recorded in the receiver, several factors (thermal drift, electronic instability, cable length differences, or variations in antenna airgap) can cause inconsistent wavelet arrival time. <a href="https://books.google.com/books?hl=en&lr=&id=y__uIi-5RvgC&oi=fnd&pg=PP1&dq=Harry+M.+Jol+-+Ground+Penetrating+Radar+Theory+and+Applications-Elsevier+Science+(2009)&ots=kLraP5-XJq&sig=kNpAJ-oChLEdFUjOKMHZO6Dkxgo#v=onepage&q=Harry%20M.%20Jol%20-%20Ground%20Penetrating%20Radar%20Theory%20and%20Applications-Elsevier%20Science%20(2009)&f=false">[5]</a> Time-zero correction provides a more accurate depth calculation because it sets the top of the scan to a close approximation of the ground surface. <a href="https://www.geophysical.com/wp-content/uploads/2017/10/GSSI-RADAN-7-Manual.pdf">[6]</a> We assume the first positive peak of each A-scan represents the 0 m depth of the ground surface, which is called as “first positive peak” method. <a href="https://erdc-library.erdc.dren.mil/jspui/handle/11681/45621">[7]</a> <a href="https://ieeexplore.ieee.org/abstract/document/1343425">[8]</a>


<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/a1c507d7-7d12-44db-af33-67b73e1acb5b" alt="image">
</p>

Figure 3. Comparison of (a) scan-by-scan time-zero correction and (b) without time-zero correction on our GPR data.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;There are multiple methods of time-zero correction, one method is calculating the mean or median value of the first peak’s arrival time for the entire A-scans and align them based on the mean or median value.<a href="https://erdc-library.erdc.dren.mil/jspui/handle/11681/45621">[7]</a> However, this method is not robust since the time-zero position is not perfectly aligning with all the 1st positive peaks of the A-scans (see Figure 4). This method does not correspond with our assumption that the first positive peak of each A-scan represents the 0 m depth of the ground surface.


<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/55aa498f-96ec-4988-9015-60c859061797" alt="image">
</p>

Figure 4. Plots of multiple A-scans with mean value method. The red vertical line shows the time-zero index based on the mean value. Some of the 1st positive peaks of A-scans align with the time-zero, but some A-scans do not.  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The other method is “scan-by-scan” which is more reasonable and robust than the mean or median strategy. This method detects the 1st positive peaks of A-scans and processes them individually. However, since the location of the 1st peak is different from each other, the data length is also changed. For example, one A-scan has 1st positive peak at the 127th depth index, the other has the 130th index, and if we align the data based on the time-zero index, the starting and ending points of A-scans mismatch each other (See Figure 5). Thus, we cut out the data indices that are not in the common range. For example, if one of the A-scan ranges [-125, 386] and the other ranges [-135, 376], we are taking the data only in common so that the range of data indices becomes [-125, 376]. Figure 6 shows the result of our scan-by-scan time-zero correction after the index cut out.



<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/4d392493-33e3-4956-8122-d8ee96378281" alt="image">
</p>

Figure 5. Plots of multiple A-scans with scan-by-scan method. The red vertical line shows the time-zero index, and the 1st positive peak is aligned to the red line. This method results in misalignment at the starting and ending points of the A-scan profiles.


<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/dc315665-60d2-4397-8135-9c5b416359dc" alt="image">
</p>
Figure 6. Plots of multiple A-scans with scan-by-scan method, but after cutting out the data that is not in the common range. The red vertical line shows the time-zero index, and the 1st positive peak is aligned to the red line. The misalignment at the starting and the ending points of the A-scan profiles is removed.


### Step 3. Migration

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Migration process converts the hyperbolic signal into spatial locations of the subsurface object. The hyperbolas in the GPR B-scan data stem from the nature of its method. GPR uses electromagnetic wave pulse to detect buried objects, and the antenna is traversing over the survey line. When the antenna is directly above the object, the distance is at its minimum. As the antenna moves, the distance is getting longer. This changing distance results in a distance-time plot that resembles a hyperbola (Figure 7). 


<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/8b35dccc-3b62-4752-b542-fcf9d42b1704" alt="image">
</p>

Figure 7. Schematic of GPR data acquisition process and hyperbola profile formation on the distance-time plot. Note that the distance is minimal when the object and antenna are vertically aligned. <a href="https://www.scirp.org/journal/paperinformation?paperid=76624">[9]</a>


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The hyperbola profiles in B-scan can lead to distortions in the radar image. Migration algorithms help correct these distortions, effectively relocating the reflected signals to their correct positions in the subsurface, resulting in a more accurate representation of the buried features. Here we specifically introduce Frequency-Wavenumber (F-K or Stolt) migration. This method has proven to be working well for the constant-velocity propagation media, <a href="https://ieeexplore.ieee.org/abstract/document/1221785">[10]</a> <a href="https://www.hindawi.com/journals/mpe/2014/280738/">[11]</a> which also fits well with our objective (bridge rebar configuration). 


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The F-K migration transforms the GPR B-scan into an artificially created wave map. In other words, this method specifically locates the object by reconstructing the waveforms at object locations. This is done by the Fourier Transform, converting the waves from the time-space domain to the frequency-wavenumber domain. To understand how this process works, we need the wave propagation equation in a media,

$$
\left( \frac{\partial^2}{\partial x^2} + \frac{\partial^2}{\partial z^2} - \frac{1}{v^2} \frac{\partial^2}{\partial t^2} \right) \phi(x, z, t) = 0,
$$

where $\phi\$ is the wave function (apparently GPR signals are electromagnetic waves), $x$ is the axis along the GPR survey line, $z$ is the axis along the depth, and $t$ is the time. If you are not familiar with the wave propagation equation in a media, you are redirected to the following YouTube video, which derives the wave equation from scratch with the guitar (acoustic wave): [https://www.youtube.com/watch?v=UXqUXYaRyGU&t=1684s&ab_channel=SteveBrunton](https://www.youtube.com/watch?v=UXqUXYaRyGU&t=1684s&ab_channel=SteveBrunton)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The next step is applying the Fourier transform to the wave function $\phi(x, z, t)\$,

$$
\phi(x, z, t) = \frac{1}{2\pi} \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} E(k_x, \omega) e^{-j(k_x x + k_z z - \omega t)} \ dk_x \ d\omega,
$$

where $E(k_x, \omega)$ is the Fourier domain for every possible combination of wave number $k_x$ and frequency $\omega$. The physical meaning of the equation is that the wave function can be expressed as a summation of various plane waves with different wavenumbers and frequencies. It is noteworthy that the $E(k_x, \omega)$ is time-independent, which will be used to reconstruct the waveform at the specific location of the object underground (The meaning of “migration” comes from this aspect).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;After the Fourier transform, we correlate our actual GPR data into the equation. Note that we receive GPR signal at $z = 0$, where the antenna locations are at the surface. Then the wave function becomes,

$$
\phi(x, z=0, t) = \frac{1}{2\pi} \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} E(k_x, \omega) e^{-j(k_x x - \omega t)} \ dk_x \ d\omega.
$$

This equation can transform the GPR signals into the frequency-wavenumber domain. At this point, it is good to mention again why we are doing all these hard things. Our objective is reconstructing the wave function at the specific location of the object in the subsurface, to remove the hyperbola profile of GPR data. To do this, we utilize the Fourier transform to convert our GPR data. This is because the frequency-wavenumber domain is independent of time, which makes it easier to reconstruct our wave function at the object position. Thus, to reconstruct the wave function, we need to know the $E(k_x, \omega)$. $E(k_x, \omega)$ is obtained by inverse Fourier transform and our GPR signal,

$$
E(k_x, \omega) = \frac{1}{2\pi} \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} \phi(x, z=0, t) e^{-j(k_x x - \omega t)} \ dx \ dt.
$$

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Now, let's formulate an equation for constructing our $E(k_x, \omega)$ based on the GPR data. To simplify our task and streamline our efforts, we'll rely on a straightforward assumption known as Exploding Source Modeling (ESM). Essentially, ESM involves assuming that the reflected GPR wave originates directly from the object itself. This allows us to represent our reconstructed signal (the migrated image) as  $\phi(x, z, t = 0)$ since $t = 0$ means the initial waveform at the object. It's important to note that this assumption is valid only when the data is time-zeroed, as our consideration of the time frame extends from the object to the surface. In other words, the assumption for time-zero correction applies solely to the timeframe between the object and the surface.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Another crucial aspect of ESM is the elimination of the $-\omega t$ in the equation. This modification enables us to employ the Fast Fourier Transform for wavefunction reconstruction. By removing the time-dependent term from the exponential, the equation becomes more numerically manageable. Here is the summarized logic of F-K migration; first, we express our GPR signal into the frequency-wavenumber domain, and this converted domain is used to reconstruct the waveform at the object locations with a very simple assumption that the wave is coming from the object. Then our final mathematical representation for F-K migration is as follows,

$$
\phi(x, z, t=0) = \frac{1}{2\pi} \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} E(k_x, \omega) e^{-j(k_x x + k_z z)} \ dk_x \ d\omega.
$$

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;In our Python code, we discretize the wave function at the surface (GPR signal) $\phi(x, z = 0, t)$ as a matrix, and then solve the equations above numerically (through fast Fourier transform) to reconstruct the wave function at $t = 0$, $\phi(x, z, t = 0)$ based on $E(k_x, \omega)$. Figure 8 shows the migration results from the mean, and scan-by-scan time zero correction, respectively. Instead of the hyperbola profiles in the raw GPR B-scan data, there are some points with high amplitude (white dots in Figure 8). These points indicate the rebar locations underground. 


<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/1809f71f-222b-4a95-b5a7-f86900ad3877" alt="image">
</p>

Figure 8. F-K migration results from (a) mean-time zero correction and (b) scan-by-scan time zero correction. 


### Step 4. Pinpoint Rebars

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;It is noteworthy that the size of the white points in Figure 8 is too large compared to the rebar diameter, so we estimate the rebar location with the K-means clustering method. K-means clustering is a popular unsupervised machine learning algorithm used for partitioning a dataset into distinct, non-overlapping clusters. The goal is to group similar data points and separate dissimilar ones. Interesting features in K-means clustering are that 1) we can specify the number of clusters we want to observe, and 2) we can point out the centroid of each cluster. We take advantage of these features to pinpoint the rebar location.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Our code normalizes the signal amplitude of the migrated data frame from -1 to 1, gets all the data points where the amplitudes are above 0.7. This allows us to get all the white data points to form a cluster around the white locations. Since we know the number of white points, we apply the K-means clustering algorithm to identify the (x, z) coordinates of each centroid of the cluster. Figure 9 shows the estimated rebar location from the mean, and scan-by-scan time zero correction, respectively. Figure 10 shows the differences in these two cases, with the root mean squared error (RMSE) value of 0.101 inches. Note that the initial rebar locations differ around 0.2 to 0.3 inches.


<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/669a777b-e284-4d1c-bc0d-4226c0668ad7" alt="image">
</p>

Figure 9. Estimated rebar location from (a) mean and (b) scan-by-scan time-zero correction.


<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/005e0767-0cbd-472d-b0b0-d817fa0f341c" alt="image">
</p>

Figure 10. Rebar location difference between mean time-zero and scan-by-scan time-zero correction.


### Step 5. Discussion 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;With the precise rebar configuration identified in our lab specimen through time-zero correction and F-K migration techniques, we have established a solid foundation for our data analysis. The remarkably clean nature of the lab specimen data has allowed us to bypass the need for additional processing steps like gain or dewow adjustments.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;As we move forward, a crucial step in validating the robustness of our methodology is to apply it to GPR data acquired from an actual bridge. Real-world scenarios often present unique challenges that may not be fully replicated in a controlled laboratory environment. To ensure the reliability and applicability of our method, the next chapter shows how we process GPR data collected from the bridge structure.



