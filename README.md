# Blank Classifier

## Data acquisition

The script to get data for the blank classifier is through `get_blank_classifier_data.py`

- This script runs in a python 2 environment present in the Master Server

Command:
```python
sudo python get_blank_classifier_data.py --thresh=500 --email_ids=aman.khullar@oniondev.com --start_date=2019-01-01 --end_date=2019-05-31 --output=/tmp/blank_classifier.xlsx
```

- Thresh is the number of items in the training data for each class. In the example above there will be 500 items for Accept class, 500 items for the Blank class

- Here the Blank class is '0' and Accept class is '1'

- The data generated from the above command is stored in the 'data' folder by the name `blank_classifier.xlsx`

- With respect to number of data points that should be used to train the classifier, there is no fixed rule but with around 17,000 items, we got a test set accuracy of 98% so probably using around 15,000 in the training set is a good idea.


### Download the audios and convert them into wav files

##### Activate the python 2 environment as in Master server
source activate <environment_name>
> NOTE: The requirements.txt has been created to install the dependencies

Download the audios of only the clean data as produced in the previous step and then convert them into wav files which are easier for feature extraction.

Download audio files:
```python
python download_audios.py --in_file="<in_file>.xlsx" --out_folder_mp3="<folder name>"
```
- Time taken to download 100 audios : 30.27.30 seconds (this step also depends on the stability of internet connection)
- Time taken to download 6000 audios : 1541.53 seconds

> Hardware specification: Processor: Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz, 16GB RAM

Convert mp3 files to wav files:
```python
python convert_to_wav.py --in_folder="<folder created in previous step>" --out_folder="<folder containing wav audios>"
```

- Time taken to Convert the audios to Wave : 9.36 seconds
- Time taken to Convert 6000 (actually 5964) audios to Wave : 833.69 seconds

### Audio features extraction

The 136 features are extracted for all the audio files using this script. Corresponding feature files in the form of numpy arrays are saved. The audio features are extracted using [PyAudioAnalysis library](https://github.com/tyiannak/pyAudioAnalysis/wiki/3.-Feature-Extraction). 4 values (25th percentile, 50th percentile, 75th percentile and 95th percentile) from the 34 features are extracted to get the 136 (34*4) features.

- We use a 50ms framesize with a 25ms stepsize
- Hence, for example if we have an audio of 100ms, there will be 3 frames and for a 1min audio there will be 2399 frames
- There will be 34 features for each frame
- Hence the feature matrix size will be 34*2399 for a 1 minute audio
- After that we take the 25th percentile, 50th percentile, 75th percentile and  95th percentile for each feature across all the frames
- We finally get a 34*4=136 feature vector for any audio file

Extract and store audio features - in both npy files and csv format:
```python
python get_features.py --in_file=data/blank_classifier.xlsx --auds_dir=audio_files_wav/ --out_file=data/blank_classifier_feats.csv --feats_dir=feats_dir
```

- Time taken to extract features of 50 audio files : 371.02 seconds and for 100 audio files : 741.04 seconds
- Time taken to extract features from 6000 audio files: approx 6 hours

### Make the train-test data split
```python
python create_train_test_splits.py --feats_csv=data/blank_classifier_feats.csv --train_csv=data/blank_classifier_feats_train.csv --test_csv=data/blank_classifier_feats_test.csv
```
This script randomizes the data and keeps 80% of data in the training set and 20% of data in the test set


### Train the blank classifier
We are using a Random Forest classifier to label an audio into the 'Blank' vs. 'Accept' class.

```python
python train_blank_classifier.py --feats_file=data/blank_classifier_feats_train.csv --model_name=blank_classifier_31_05_2021.pkl --scaler_name=blank_classifier_scaler.pkl
```

### Test the blank classifier
Get the Confusion Matrix and accuracy of the new model. You should compare this with the previous model on the same dataset and then decide if the new model should be used.
```python
python test_blank_classifier.py --feats_file=data/blank_classifier_feats_test.csv --model_name=models/blank_classifier_31_05_2021.pkl --scaler_name=models/blank_classifier_scaler.pkl
```


<!-- ('Accuracy on training set: ', 0.9603130240357741)
('Accuracy on test set: ', 0.931323283082077)
Time taken to train SVM gender classifier : 5.48421382904 seconds

Average Accuracy : 0.931
('F1_score', array([0.92531876, 0.93643411]))


27th May

('Accuracy on training set: ', 0.9575181665735047)
('Accuracy on test set: ', 0.9514237855946399)
Time taken to train SVM gender classifier : 5.38154792786 seconds
('Accuracies : ', array([0.95142379]))
Average Accuracy : 0.951
('F1_score', array([0.94974003, 0.95299838])) -->


