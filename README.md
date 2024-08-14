# NCEP_DATA
Script used to Extract Weather Variables from the NCEP GFS 0.25 Degree Global Forecast Grids from 2020-2024 for areas around Bangkok.

# Project Background
This program was created for the BDI (Big Data Institute Thailand) EnviLink project. The purpose of this project is to scrape data from the **[NCEP GFS 0.25 Degree Global Forecast Grids Historical Archive](https://rda.ucar.edu/datasets/ds084.1/)** from coordinates nearest to the 83 Air Quality Stations in Bangkok. Once the data has been scraped, the data can be used to train Machine Learning models to determine the Feature Importance relating to PM2.5 pollution in Bangkok. The main variables the project is interested in are Ventilation Rates, Planetary Boundary Layer Height, Gust, Wind Direction, and Temperature, at certain times of a specific day, month, and year. 

# NCEP/.grib2 Overview
NCEP uses  **.grib2** file format to store their global forecast model data. As stated on their website, the data is recalculated every 6 hours [0:00,6:00,12:00,18:00] and also includes prediction every 3 hours from hour 0 to 384 hours in the future from the last recalculation. See this [link](https://rda.ucar.edu/datasets/d084001/#) for more information on the NCEP data.

Due to the size of the **.grib2** file, downloading all the files to be stored in a local computer is not suitable. Therefore, this program downloads files either in chunks or one by one (depending on preference and computing power) and deletes them after it has been used, leaving only **.idx** files behind which are much smaller. The core of the program uses **xarray** to load files with the **cfgrib** engine, however, it has been noted that **cfgrib** has certain limitations, and **pygrib** can run better in certain contexts. However, in this project, using **cfgrib** is the most efficient and easiest way to access the data. 


# Program Scripts

There are three scripts which is needed to run the web scraper.

 1. **MULTI_DRIVER_FOURs.ipynb**
 2. **MULTI_FUNC_FOURs.ipynb**
 3. **file_deleter.ipynb**

# Installation
1. Download the jupyter notebook scripts and save it in a directory that you will use for the project. 
2. Each notebook should already have the list of libraries to install via the **pip** command. However, there may be dependencies that you may need to download first. Listed below are the libraries used by the scripts. You can check what installations are missing by seeing the output if an error occurs. 

	    pip install --upgrade bottleneck
	    pip install cfgrib==0.9.13.0
		pip install ipynb
		pip install pandas
		pip install requests
		pip install xarray
		pip install multiprocess
		pip install python-csv

# Changing the current working directory

Make sure to change the working directory to the one you will be using in your project. If you are using **MacOS** and your documents or desktop is saved to **iCloud**, set your working directory to somewhere that is not uploaded to iCloud like downloads. NOTE: If you save it to a directory that is uploaded to iCloud, it will take up significant computer space while it is being uploaded to iCloud.

Change the project directory in the **MULTI_DRIVER_FOURs.ipynb** in the second cell.

    # Set the directory
    
	desired_directory = '/Users/trak/Downloads/TEMPOOL'
	
	os.chdir(desired_directory)
	
	print("Current Working Directory: ", os.getcwd())

Make sure to do the same for the **MULTI_FUNC_FOURs.ipynb** in the process_file function

    def process_file(file,VR,P,G,PR,T,i,meta,U10,U20,U30,U40,U50,U80,U100,V10,V20,V30,V40,V50,V80,V100,pix,latitude,longitude):
	    # Downloading each file
	    filename = os.path.basename(file)
	    save_dir = '/Users/trak/Downloads/TEMPOOL/'
	    file_path = save_dir + filename
	    if not os.path.exists(file_path):
	        print('Downloading', file)
	        req = requests.get(file, allow_redirects=True)
	        with open(filename, 'wb') as f:
	            f.write(req.content)
	    
	    # Selecting File
	    # Assuming `save_dir` is predefined and accessible
	    save_dir = '/Users/trak/Downloads/TEMPOOL/'
	    file_path = save_dir + filename
    
   

# Program Setup

### Define Date Range

Once the working directory has been set, the next step is to set the range of dd/mm/yr to extract.  

### Set lat,lon pairs

The **lat_l** list is paired with **lon_l** of the same index, becoming a lat,lon pair. Add lat,lon values to both list  to create a point to extract data. 

    # The top lat corresponds to the bottom lon at the same index. Add as needed
	lat_l= [13.75,13.75,14,13.75]
	lon_l= [100.5,100.75,100.5,100.25]


# Changing/Adding Variables to Pull

To change the variables you want to pull, please refer to the NCEP website at the top of this document to see the dataset's list of variables. You will need to edit both the **MULTI_FUNC_FOURs** and the **MULTI_DRIVER_FOURs** scripts.

In the **MULTI_FUNC_FOURs**, add the variable name to this line.

    def process_file(file,VR,P,G,PR,T,i,meta,U10,U20,U30,U40,U50,U80,U100,V10,V20,V30,V40,V50,V80,V100,pix,latitude,longitude):


Open the dataset on the **typeOfLevel** that your variable is located in: 

     ds = xr.open_dataset(file_path, engine="cfgrib",filter_by_keys={'typeOfLevel': 'heightAboveGround','level':10})

The code above is used to open the dataset. You need to specify which **typeOfLevel** you wish to access. All the variables are categorized in a certain **typeOfLevel**. 

    vrate = ds["VRATE"].sel(latitude=latitude, longitude=longitude, method='nearest')
This next line of code is used to access the specific variable within the level. You will also need to specify which latitude, and longitude, you wish to see the values for. 

Finally, below this code, add your variable in the format following the other variables above. This adds the variable values to the list.

    # Add each value to the list
    VR[i] = str(vrate.values)
    P[i] = str(pblh.values)
    G[i] = str(gust.values)
    PR[i] = str(prate.values)
    T[i] = str(temp.values)
    U10[i] = str(u10.values)
    V10[i] = str(v10.values)
    U20[i] = str(u20.values)
    V20[i] = str(v20.values)
    U30[i] = str(u30.values)
    V30[i] = str(v30.values)
    U40[i] = str(u40.values)
    V40[i] = str(v40.values)
    U50[i] = str(u50.values)
    V50[i] = str(v50.values)
    U80[i] = str(u80.values)
    V80[i] = str(v80.values)
    U100[i] = str(u100.values)
    V100[i] = str(v100.values)
    meta[i] = str(filename)

In the **MULTI_DRIVER_FOURs**, you will need to create a manger list in order to retain order within the different processes. 

    with mp.Manager() as manager:
        VR = manager.dict()
        P = manager.dict()
        G = manager.dict()
        meta = manager.dict()
        u10 = manager.dict()
        u20 = manager.dict()
        u30 = manager.dict()
        u40 = manager.dict()
        u50 = manager.dict()
        u80 = manager.dict()
        u100 = manager.dict()
        v10 = manager.dict()
        v20 = manager.dict()
        v30 = manager.dict()
        v40 = manager.dict()
        v50 = manager.dict()
        v80 = manager.dict()
        v100 = manager.dict()
        PR = manager.dict()
        T = manager.dict()

Below this code, set your variable to **manager.dict()**. 

    p = mp.Process(target=wrapper_process_file, args=(file, VR, P, G,PR,T, idx, meta,u10,u20,u30,u40,u50,u80,u100,v10,v20,v30,v40,v50,v80,v100,pix,lat,lon))

Add your variable to the row headers.

    fields = ['Date', 'Time', 'VRATE', 'PBLH', 'WIND_SPEED(GUST)','PRATE','Temp','pixelID','lat','lon','u10','v10','u20','v20','u30','v30','u40','v40','u50','v50','u80','v80','u100','v100', 'file']

Add your variable to the **args**. Also, add the variables with (.get(idx)) to the line below.

    rows.append([dmy, hour, VR.get(idx), P.get(idx), G.get(idx),PR.get(idx),T.get(idx),pix,lat,lon,u10.get(idx),u20.get(idx),u30.get(idx),u40.get(idx),u50.get(idx),u80.get(idx),u100.get(idx),v10.get(idx),v20.get(idx),v30.get(idx),v40.get(idx),v50.get(idx),v80.get(idx),v100.get(idx),meta.get(idx)])

Finally, clear the variable values by adding the variable to this line.

    del VR, P, G, meta,u10,u20,u30,u40,u50,u80,u100,v10,v20,v30,v40,v50,v80,v100,PR,T


# Changing Chunk size/Semaphore size

!!! It is **not recommended** to adjust the chunk_size at this moment as with the way the code is written in the current version, each **multiprocess** process will download the same files at the same time, significantly slowing down runtime. Updates in the future may be added to fix this issue.

However, you can still change the chunk size if you want to explore with optimization. You can increase how many files are downloaded at a time by in Cell 6 of **MULTI_DRIVER_FOURs**.

    # Process files in chunks
    chunk_size = 1  # Adjust this based on memory constraints

You can also change the Semaphore number in the **MULTI_FUNC_FOURs** as well.

# Tips on Speeding Scraping Time

Even with **Multiprocess**, it can still take a long time to load the files. Running copies of the **MULTI_DRIVER_FOURs.ipynb** can help speed up run time by making each copy run at the same time for a different chunk of date ranges. For example, **MULTI_DRIVER_FOURs(1)** may run from **Jan 1, 2020** to **May 31, 2020**. **MULTI_DRIVER_FOURs(2)** can run from **June 1, 2020** to **December 31, 2020**. As the file does not take up too much CPU and RAM space, many copies can be run at the same time. The main limiting factor is your **network speed** which determines how fast you can download files from the internet. 

# file_deleter

The file_deleter is used to delete the file after it has been used. Run this file after you have started running the other two files in order to save computer space. Adjust the **time.sleep(#)** value as appropriate to your computing speed and network speed. 

If you run multiple copies of the **MULTI_DRIVER.ipynb**, make sure to also create another copy of the **file_deleter.ipynb** as well. 
