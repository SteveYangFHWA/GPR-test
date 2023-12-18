# Case Study: GPR data processing for mapping rebar configuration on the bridge

## Chapter 1. Introduction

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Here in the FHWA CHARISMA Github repository, we provide a Python-based case study for processing raw GPR data to locate the rebar configuration of a concrete bridge. We discuss detailed step-by-step tutorials with actual case studies on 1) Experimental lab specimens and 2) Actual bridge (I-10 Mississippi). We first discuss the data (dimensions, physical meaning, and sizes), and then see how the data is being processed (i.e., gain, dewow, time-zero correction, and F-K migration) with visual plots and figures. Finally, we analyze the results to locate the rebar configuration.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ground Penetrating Radar (GPR) is a non-destructive evaluation technique to investigate objects or structures buried underground. The basic principle of GPR involves the transmission of high-frequency radio waves (10 MHz to 2.6 GHz) into the ground. When these waves encounter subsurface materials with different electrical properties, the waves are reflected to the surface. The GPR system records the time it takes for these reflections to return, and this information is used to create a profile of the subsurface.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;While the antenna is traveling the bridge horizontally (along the x-axis), the transmitter (Tx) of the antenna emits the electromagnetic wavelet and it penetrates through the ground (z-axis). The reflected signal from the object in the subsurface is recorded into the receiver (Rx) of the antenna. Thus, the recorded data contains multiple reflected wavelets with respect to the distance along x and z (note that z also indicates wave propagation time). Each reflected wavelet as a function of discrete time is called an “A-scan”. “B-scan” is obtained by stitching multiple A-scans together along the survey line (x) and assigning a color (black to white) to the wave amplitude. Since the B-scan provides 2D vertical information underground, it is commonly used to analyze the system or structure. “C-scan” refers to the horizontal view of the specific horizontal layer of the subsurface. Figure 1 helps us understand the definition of each type of scan from GPR.

<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/945457f6-3e45-46c8-8921-59d08ed28bdd" alt="image">
</p>

Figure 1. The definition of A-, B-, and C-scan. <a href="https://doi.org/10.1515/jag-2020-0004">[1]</a>



<p align="center">
  <img src="https://github.com/SteveYangFHWA/GPR-test/assets/154262555/2b3f8c0a-64dd-421f-a4e7-9340ef3bd8da" alt="image">
</p>

Figure 2. GPR data acquisition on the bridge (Left) and how GPR B-scan data looks like with respect to the actual rebar configuration (Right). The horizontal axis (x-axis) of the data corresponds to the distance along the survey line (moving direction of the GPR machine), and the vertical axis (z-axis) is the depth of the ground.  <a href="https://infotechnology.fhwa.dot.gov/bridge/">[2]</a>


### Step 1. Environment setup via Anaconda

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Anaconda is commonly used for managing Python packages in user-defined environments. In other words, it allows you to create isolated environments for different projects, each with its own set of dependencies, without interfering with the system-wide Python installation. This is particularly useful in data science and scientific computing where projects may have different requirements and dependencies. Here in First, download Anaconda in the provided URL.

URL to download conda: [https://www.anaconda.com/download](https://www.anaconda.com/download)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;We recommend to install our conda environment (`charisma-env`) to run the code properly. This environment has all the dependencies from our entire code. Download the `charisma-env.yml` file from our CHARISMA Github first, open your Anaconda Prompt, and go to the “Downloads” directory. 


To create charisma-env with conda, run the following command in your root environment:

`conda env create -f charisma-env.yml`

To activate the charisma-env in conda:

`conda activate charisma-env`

To verify that the new environment was installed correctly:

`conda env list`

After creating the charisma-env environment, install Jupyter Notebook or Spyder from Anaconda Navigator to use our CHARISMA Python package.


### Step 2. Convert DZT into CSV files

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GPR data is saved in DZT format (which is not accessible without specific software), so here we provide a data format conversion code to CSV format, which is more friendly to readers. The readdzt function in our Python code (`gprPreprocessing.py`) is in charge of opening, reading, and configuring the DZT data into the Pandas data frame, and the save_to_csv function exports the data frame into CSV format. Here the readers need to define the directory path of the downloaded DZT file and “user_directory” individually to save the CSV files in your storage system. Here we are saving two CSV files for data frame1 `df1` and data frame2 `df2` from one DZT file. The `df1` is for collected GPR data and `df2` is for the configuration settings of GPR.
```
data, hdr, df1, df2 = readdzt(/Your directory/FILE____488.DZT')
user_directory = "H:/GPR_work"
save_to_csv(df1, user_directory, 'data')
save_to_csv(df2, user_directory, 'config')
```


