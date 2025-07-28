# Helpful Tips


## Directory Structure

This is a more in depth explanation of the parts of the directory I wanted to highlight. 

Look at the simple_mid folder under *benchmark_eval/data/gt*. There are 3 items:

- **gt folder**: holds gt.txt, which contains ground truth values. The file should be in motchallenge format. 
- **img1 folder**: Holds the image frames of the video sequence. Should be taken straight from original video, no bounding boxes. Current videos are 10 seconds at 60 fps, so there are 600 images in each img1 folder. 
- **seqinfo.ini file**: holds video sequence information such as name, image directory name, frame rate, etc. 

Now look at the seqmaps folder under *benchmark_eval/data*. It contains:

- **vidseq_names.txt**: file specifying all video sequences you would like to include when running the HOTA metric. 

Lastly, look under *data/predictions/mot_challenge*. It contains:

- **simple_mid.txt**: file containing model output predictions. The file should be in motchallenge format. This will be directly compared with the gt.txt file. 

```
benchmark_eval   
│
└───data
    └───gt
    |   └───simple_mid
    |       └───gt    
    |       |       gt.txt
    |       └───img1
    |       |   |   simple_mid_1.jpg
    |       |   |   simple_mid_2.jpg
    |       |   |   ...
    |       |
    |       └───seqinfo.ini
    | 
    └───predictions
    |   └───mot_challenge
    |       └───simple_mid.txt
    |            
    └───seqmaps
        └───vidseq_names.txt

```


## Example Files

Here are some examples of what the contents of files should look like:

**seqinfo.ini file**:

```
[Sequence]
name= insert_vid_seq_name
imDir=img1
frameRate=60
seqLength=600
imWidth=1920
imHeight=1080
```

**vidseq_names.txt file**:

*make sure to include the word "name" at the top before listing the video sequences!

```
name
simple_mid
simple_ben
difficult_mid
difficult_ben
```

First 5 lines of **simple_mid.txt**:
```
1, 1, 1002.00, 610.50, 45.00, 58.50, 1, 1, -1
1, 2, 41.91, 207.94, 110.06, 67.13, 1, 1, -1
1, 3, 1134.75, 567.75, 42.00, 28.50, 1, 1, -1
10, 1, 1006.06, 614.45, 44.20, 56.91, 1, 1, -1
10, 2, 33.26, 200.92, 105.12, 64.41, 1, 1, -1
```

And the first 5 lines of its corresponding **gt.txt**:
```
1, 8, 441.00, 632.00, 21.00, 31.00, 1, 1, -1
1, 1, 1002.00, 610.50, 45.00, 58.50, 1, 1, -1
1, 2, 41.91, 207.94, 110.06, 67.13, 1, 1, -1
1, 3, 1134.75, 567.75, 42.00, 28.50, 1, 1, -1
10, 8, 431.00, 635.00, 21.00, 31.00, 1, 1, -1
```


