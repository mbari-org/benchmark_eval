# Usage 

## How to annotate a video in Rectlabel

To annotate a video, you need images, labels, and xml folders. This step can be done anywhere outside or inside the directory, since we will be exporting the annotated files to benchmark_eval. 

- Go to File -> Convert video to image frames and put the resulting frames into an img folder. 
- Create an empty xml folder. 
- Go to **tracker_output.ipynb** and set the video sequence, model, and tracker you are using in the **setting variables cell**. Run the **defining functions cell**.
- Run **model + tracker** cell to get yolo labels. The model parameters can be adjusted here, i.e. conf, iou, and imgsz. If you are using Windows/Linux, change device='mps' to device='cpu' or device='cuda' (GPU). Tracker parameters can also be adjusted in the yaml file. 
- In rectlabel, go to export -> import yolo txt files. To open your files, select the images folder for the "Images folder" and xml folder for the "Annotations folder". 

Your file structure could look something like this:
```
videos  
│
└───simple_mid
|    └───images
|    └───labels  
|    └───xml
| 
└───simple_ben
| ...
```

Helpful rectlabel tips:

- Use move to edit boxes and create to create boxes
- Turn on "Show labels on boxes" and "Show checkboxes on labels table"
- Double click bounding box to edit
- Command + arrow key to move through frames faster


## How to get Ground Truth files
- Under *benchmark_eval/data/gt*, create a folder named after your vidseq_name and create an empty folder named gt inside. ie (*../simple_mid/gt*).
- After you have cleaned up detections in Rectlabel, go to Export -> Export yolo txt files, save them under the folder you just created. You can delete the names file that Rectlabel exports along with your labels (ie mbari452k.yaml)
- Go to **tracker_output.ipynb** and make sure you have the correct video sequence in the **setting variables cell**.   
- Run the **ground truth cell** to add the track id to the yolo txt files. This will look for the "labels" folder you exported to *benchmark_eval/data/gt/vidseq_name/gt* and create a folder called "labels_w_trackid". This folder will then be converted from yolo to motchallenge format. The final ground truth file will be called gt.txt and the temporary "labels" and "labels_w_trackid" files should be deleted. 

### What is the MOTChallenge format?

The MOTChallenge file format is required for the metric calculation in TrackEval. The files from Rectlabel are exported as yolo txt files, so they need to be converted. 

MOTChallenge format:

```
<frame>, <id>, <x>, <y>, <w>, <h>, <conf>, <class>, <vis>
```

Example line:

```
1, 1, 1002.00, 610.50, 45.00, 58.50, 1, 1, -1
```

This means that in frame 1, an object with a track id 1 has a bounding box with a top left corner at (1002.00, 610.50) and a width of 45.00 and height of 58.50. Trackeval does not require confidence, class, or vis (visibility), so for these purposes I set them manually to 1, 1 and -1, respectively. 


## How to get Model Output files
- Go to **tracker_output.ipynb** and double check the video sequence in the **setting variables cell**.
- Run the **model + tracker cell** to get the model detections and convert them from yolo to motchallenge format. The final txt file should end up in *benchmark_eval/data/predictions/mot_challenge*

*if an error occurs, double check the paths. There are folders being generated and folders that you add yourself, so its possible that things can get mixed up!


## How to get HOTA scores
- In the video sequence folders under *benchmark_eval/data/gt*, add a folder named after your video sequence. In this folder:
    - add a folder called img1 to hold all the video image frames (should be the original frames, no bounding boxes).
    - Create a seqinfo.ini file - the format is shown in [Tips](tips.md). The name should match the video sequence folder name, and the imDir should match the image directory name. Adjust frame rate, sequence length, image width, and image height accordingly. 
    - There should already be a gt folder from the "How to get Ground Truth files" step. 
- Next, edit the vidseq_names.txt under *benchmark_eval/data/seqmaps*. The format should have “name” at the top, and then all the video sequences you would like to get the HOTA values for. An example can be seen in [Tips](tips.md).
- Lastly, in the terminal, make sure you are in the benchmark_eval directory and type in this command:

```bash
python scripts/run_mot_challenge.py  
  --GT_FOLDER data/gt \
  --TRACKERS_FOLDER data/predictions \
  --SEQMAP_FILE data/seqmaps/vidseq_names.txt \
  --TRACKER_SUB_FOLDER ''
```

### What is HOTA?

Plain english: metric that measures how well trajectories of detections align, averaged over all trajectories, while also taking into account incorrect detections. 

- balances association and detection. Previously used MOTA score was biased toward detection and IDF1 score was biased toward association.
- Made up of 6 secondary metrics:
    - **DetA (detection accuracy)** - percentage of aligning detections
    - **AssA (association accuracy)** - average alignment between trajectories, averaged over all detections. Basically how well tracker maintains identity consistency
    - **DetRe (detection recall)** - how well tracker finds all ground truth detections
    - **DetPr (detection precision)** - how well tracker manages to NOT predict extra detections that arent there
    - **AssRe (association recall)** - how well trackers can avoid splitting same object into multiple shorter tracks
    - **AssPr (association precision)** - how well trackers avoid merging multiple objects into a single track
    - **LocA (localization accuracy)** - measures spatial accuracy (IoU) between gt/predicted bounding boxes
