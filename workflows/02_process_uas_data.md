# Process CAREER UAS Data

This workflow outlines how to process CAREER UAS data using Metashape on CU Boulder's research computing. Note that CAREER transects can be time-consuming to process, but that Metashape will allow you to open up multiple instances of it and process in both (To open another Metashape window, go to Terminal, create a new Terminal tab, and then run "source ~/.metashape" and "metashape" again.

## Access CU Research Computing via OnDemand
* Go to CU OnDemand: https://ondemand.rc.colorado.edu/
* Sign in - If you do not have access to CURC, see below ("Getting CURC Access")
* Start desktop (top nav bar, “Interactive Apps”, “Core desktop”)
  * Set the number of hours you expect to use the desktop (over-estimate!)
  * Set the number of cores, either 8 or 16 usually
* Wait for the job to start. It will be ‘Queued’ for a little while, exact time totally depends on current demand.
* Open the Terminal and type the following commands:
  * source ~/.metashape
  * metashape
* Before the FIRST time you ever save anything with Metashape, run this line of code from the terminal:
  * echo "umask 0000" >> ~/.metashape
  * This will ensure that all of your Metashape files and directories have open permissions (the default settings are a pain)

### Getting CURC Access
If the user has a CU Identikey, they can just sign up for an account as normal at [https://rcamp.rc.colorado.edu/accounts/account-request/create/verify/ucb]. Once they have an account they can be added to any user groups/accounts on the system that they need to access Earthlab software/files. Have the Macrosystems project manager email Research Computing Help (rc-help@colorado.edu) and ask for them to have access to Earth Lab petalibrary files and Metashape.
 
If the user does not have an Identikey, you can request one through the CU POI or Research Affiliate process ([see Earth Lab's guide to this process](https://github.com/earthlab/earth-lab-operations/wiki/CU-Research-Affiliates-&-Persons-of-Interest-(POIs))). Once that process is finished, the user can sign up for a CURC account per the above instructions.

## Document that you are starting the process
* Before starting to process imagery, check the [data processing spreadsheet](https://docs.google.com/spreadsheets/d/1QifnM6ORmHZaS2IsCR-tbr5HOFIdyin8sbgU08rIpkE/edit?usp=sharing) to make sure that nobody has done the transect you are about to do. Write down that you are starting processing in by putting "YOUR_INITIALS in process" in the orthomosaic column of the transect you are working on.

## Process in Metashape

This section of the workflow describes how to process UAS data for the CAREER project. It is a more specific version of the [Earth Lab metashape workflow](https://github.com/earthlab/el-drones/blob/master/docs/03_post-mission_agisoft_metashape_workflow.md), which was developed by Megan Cattau from a guide posted in this [Agisoft forum post](https://www.agisoft.com/forum/index.php?topic=7851.0). Also see Megan Cattau's [UASWorkflows repo](https://github.com/mcattau/UASWorkflows).

### Steps
0. If you are operating on CU Boulder research computing resources, before the FIRST time you ever use Metashape, run this line of code from the terminal:

   echo "umask 0000" >> ~/.metashape
   
   This will ensure that all of your Metashape files and directories have open permissions (the default settings are a pain)
1. Open Metashape
   - To open metashape on  CURC, open a terminal and type the following code lines:
        - source ~/.metashape
        - metashape
2. Save project in the 'metashape' directory for your transect
    - When saving your file, keep the file naming structure as the plot or transect directory you are working in and add _METASHAPE to the end
3. Set preferences (Tools -> preferences)  
    - -> GPU-> Check box for GPU if there is one available, check box for Use CPU when performing…)

#### Splitting data into chunks
It may be useful to process the data in chunks in order to optimize processing time, especially for the dense cloud. In this case, you can add multiple chunks to the workspace
 - select "Add Chunk" button at the top of the workspace
 - choose as many chunks as makes sense for the size of your project

If you choose to process your data in chunks, then you will need to perform the following steps in the workflow on ALL of the chunks in your workspace separately. However, "Align Photos" and "Build Dense Cloud" can be automated using a batch process (workflow -> Batch Process)

4. Import photos (Workflow menu in top toolbar -> add folder--adding the entire folder is often easiest   
    - Select single camera system
    - If processing in chunks, distribute the photos between each of your chunks
5. Clean up photos 
    - Manually remove outlier photos (e.g.,  taken during take-off) in Model workspace
    - Check camera calibration (from EXIF files) (tools-> Camera calibration) exiftool
    - Do this for each chunk in your workspace  
6. Convert GPS coordinates of your geotagged images to match the coordinate system of your project. Use the "convert" tab under the "Reference" panel and select the coordinate system from the "coordinate System" under "Convert Reference" dialog box. Verify coordinate system of imagery by clicking on chunk in Workspace pane and looking in bottom left corner.
    - For all CAREER project imagery, convert coordinates to EPSG::26913.
    - Do this for each chunk in your workspace and double check that each chunk has correct coordinate system assigned at bottom left corner of workspace
7. Estimate image quality  
    - In Photos workspace, change view to detailed > select all photos > right-click > Estimate Image Quality... 
    - Disable all images that have an image quality below 0.7
    - Do this for each chunk in your workspace  
8. Align photos (workflow->) same as generating sparse point cloud  
    - High accuracy, generic preselection, reference preselection
    - Under advanced, check Adaptive model fitting
    - CAREER uses 40,000 & 10,000 for key and tie point limits
  If working with multiple chunks, you can use a batch process to automate alignment for all of your chunks (Workflow -> Batch Process -> Add -> Job Type: Align Photos, Apply to: All Chunks or Selection)
 - check "Save project after each step" and hit OK
9. Clean sparse point cloud (Model > GRADUAL SELECTION). Remove all points with high reprojection error (choose a value below 1, suggest using 0.5-0.8 ) and high reconstruction uncertainty (try to find the 'natural threshold' by moving the slider).
    - Do this for each chunk in your workspace
10. Adjust bounding box to be just at the boundary of the furthest out flight lines
    - Do this for each chunk in your workspace
11. Optimize Cameras (tools ->) to improve alignment accuracy  
    - Check all but bottom left one (Fit k4) when using DJI imagery. Don’t check advanced (Adaptive camera model fitting and est tie point covariance)
    - Do this for each chunk in your workspace  
    - Agisoft’s description of through correspondence (AIS): Optimize cameras - you refine the camera calibration parameter values based on the calculated values after the images are aligned.   
12. Build Dense point cloud (workflow ->)    
    - High quality
    - Aggressive depth filtering
    - check box for calculate point colors)
  If working with multiple chunks, you can use a batch process to automate dense cloud processing (workflow -> Batch Process -> Add -> Job Type: Build Dense Cloud, Apply to: All Chunks or Selection)
 - check "Save project after each step" and hit OK

  #### Merging chunks
After creating the dense cloud for each chunk, merge all of the chunks together (workflow -> Merge chunks)
Be sure that you build the DEM in the following step using the Dense Cloud from the Merged Chunks

13. Build DEM from dense cloud (workflow ->)   
    - For orthomosaic generation – faster than mesh (but mesh may be required for complex terrain)  
    - Geographic type, check that it’s correct projection  
    - Source data: dense cloud  
    - Interpolation: Extrapolated (interpolation – enabled would leave elevation values only for areas seen by a camera)  
    - Check resolution  
14. Build orthomosaic based on DEM (workflow ->)
    - DEM surface
    - Select if you want blending, otherwise Mosaic (default)
    - Enable hole filling
    - Check pixel size (can check meters button)
15.	Export results to the 'metashape/metashape_outputs' folder for your transect
    - Right click on the Dense Cloud and 'Export Dense Cloud...' Export the points with the appropriate transect name and then '_POINTCLOUD'. The filetype will be .laz (the default)
    - Since CAREER project transects point clouds are large, these will need to be tiled. In the dense cloud export dialogue (after you choose your file name), check the box next to "Split in blocks (m):" and then put in your block size. For CAREER processing, use 100x100m.
    - Right click on the orthomosaic and 'Export Orthomosaic...' Export the orthomosaic with the appropriate transect name and then '_ORTHO'. 
19. Generate the processing report to the 'metashape/metashape_outputs' folder for your project
    - File -> Export -> Generate Report
    - Keep all defaults, but re-name it to the appropriate transect name and then "_report". Click 'ok'



## Document that you've finished the process
* Make sure to mark down that the processing is complete on the [data processing spreadsheet](https://docs.google.com/spreadsheets/d/1QifnM6ORmHZaS2IsCR-tbr5HOFIdyin8sbgU08rIpkE/edit?usp=sharing) by putting your initials in the column of each output

