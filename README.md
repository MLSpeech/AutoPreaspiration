# AutoPreaspiration
A software package for automatic extraction of pre-aspiration from speech segments in audio files,  using a trainable algorithm.

The package is based on AutoVot project - [automatic extraction of voice onset time (VOT) from audio 
files (2014 Joseph Keshet, Morgan Sonderegger, Thea Knowles)](https://github.com/mlml/autovot). The core implemantion of AutoVot was used to train a new model for the task of pre-aspiration detection. Some of the cpp files were changed
to fit the new task. In addition, some of the wrapper python files were changed and many new scripts and 
tests were added (mainly python and bash scripts).

It works as follows:
* The user provides wav files containing one (or more) obstruents, and corresponding Praat TextGrids containing some information about roughly where the pre-aspiration should be (e.g. time index within the preceding phoneme and time index within the obstruent).
* A classifier is used to find the pre-aspiration, for each coded obstruent, and add a new tier to each TextGrid containing these measurements.
* The user can either use a pre-existing classifier, or (recommended) train a new one using a manually-labeled pre-aspirations from their own data.

***Note: A newer project for extracting pre-aspiration using Recurrent Neural Network based method can be found here: [BiRNN_AutoPreaspiration](https://github.com/Yanivs24/BiRNN_AutoPreaspiration)***

## Dependencies
In order to use AutoPreaspiration you'll need the following installed in addition to the source code provided here:
* [GCC, the GNU Compiler Collection](http://gcc.gnu.org/install/download.html)
* [Python (Version 2.7 or earlier)](https://www.python.org/download/releases/2.7.6>)
* If you're using Python version 2.6 or earlier, you will need to install the argparse module (which is installed by default in Python 2.7), e.g. by running `easy_install argparse` on the command line.
* If you're using Mac OS X you'll need to download GCC, as it isn't installed by default.  You can either:
	* Install [Xcode](http://itunes.apple.com/us/app/xcode/id497799835?ls=1&mt=12), then install Command Line Tools using the Components tab of the Downloads preferences panel.
	* Download the [Command Line Tools for Xcode](http://developer.apple.com/downloads) as a stand-alone package.

  You will need a registered Apple ID to download either package.
  
  
## Installation

The code to clone AutoPreaspiration is: 

	$ git clone https://github.com/Yanivs24/AutoPreaspiration.git
	

Alternatively, you can download the current version of AutoPreaspiration as a zip file, just press "Clone or download" -> "Download ZIP"


## Compiling
Clean and compile from the `code` directory:

	$ cd AutoPA/AutoPA/code
	$ make clean

Then, run:

	$ make

Final line of the output should be:

	[make] Compiling completed
    

Finally, add the path to `code` to your `experiments` path:

	$ cd ../../experiments
	$ export PATH=$PATH:/[YOUR PATH HERE]/AutoPA/AutoPA/bin
	
	For example:
	$ export PATH=$PATH:/home/yaniv/projects/AutoPA/AutoPA/bin

If not working out of the given `experiments` directory, you must add the path to your intended working directory.
***IMPORTANT: YOU MUST ADD THE PATH EVERY TIME YOU OPEN A NEW TERMINAL WINDOW***


## Setup
First, make sure your working directory is the experiments directory.

In order to work with a new data, the wav files should be converted to 16kHz mono (if this is already the case this step should be
skipped).
This can be done by typing:

	$ python python_scripts/format_wav_files.py DIRECTORY_PATH
	
***A prerequisite for this script is installing SoX utility***

When 'DIRECTORY_PATH' is the path of the directory containing the data (wav&TextGrids)
This will place all the formatted wav files and the corresponding TextGrids in: DIRECTORY_PATH/formated


## Usage

You can skip this and go to the [Simple Usage Mode](#simpleusage) for a simpler user mode (but with less options).

### AutoPreaspiration allows for two modes of feature extraction:

* **Mode 1 - Covert feature extraction:** The handling of feature extraction is hidden. When training a classifier using these features, a cross-validation set can be specified, or a random 20% of the training data will be used. The output consists of modified TextGrids with a tier containing Preaspiration prediction intervals.
* **Mode 2 - Features extracted to a known directory:** Training and decoding is done after feature extraction. Features are extracted to a known directory once after which training and decoding may be done as needed. The output consists of the prediction performance summary. Mode 2 is recommended if you have a large quantity of data. 

***Note: All the different options of the following scripts can be viewed from the command line for each python program using the flag -h***

	For example:
	auto_pa_train.py -h
	
Some important optional arguments used by most of the scripts:

	--pa_tier PA_TIER     Name of the tier to extract pre-aspiration from (default: pa)
	--pa_mark PA_MARK     Only intervals on the pa_tier with this mark value
        	              (e.g. "pre") are used for training, or
                	      "*" for any string (this is the default)
	--window_min WINDOW_MIN
                              Left boundary of the window (in msec) relative to the
                              pre-aspiration interval's right boundary. Usually should be
                              negative, that is, before the pre-aspiration interval's left
                              boundary. (default: -50)
	--window_max WINDOW_MAX
          	              Right boundary of the window (in msec) relative to the
                 	      pre-aspiration interval's right boundary. Usually should be
                              positive, that is, after the pre-aspiration interval's right
                              boundary. (default: 60)



### Feature extraction and training

#### Mode 1:
##### *Train a classifier to automatically measure Preaspiration, using manually annotated Preaspirations in a set of textgrids and corresponding wav files.*
###### Usage: auto\_pa\_train.py [OPTIONS] wav\_list textgrid\_list model_filename 

#### Mode 2: 
##### *Extract acoustic features for AutoPreaspiration. To be used before auto_pa_train_after_fe.py or auto_pa_decode_after_fe.py*
###### Usage: auto\_pa\_extract\_features.py [OPTIONS] textgrid\_list wav\_list input\_filename features\_filename labels\_filename features_dir

##### *Then, we can train a classifier to automatically measure Preaspiration, using manually annotated Preaspirations for which features have already been extracted using auto_pa_extract_features.py, resulting in a set of feature files and labels.*
###### Usage: auto\_pa\_train\_after\_fe.py [OPTIONS] features\_filename labels\_filename model\_filename

### DECODING

#### Mode 1
##### *Use an existing classifier to measure Preaspiration for stops in a set of textgrids and corresponding wav files.*
###### Usage: auto\_pa\_decode.py [OPTIONS] wav\_filename textgrid\_filename model\_filename

                        
#### Mode 2
##### *Decoding when features have already been extracted*
###### Usage: auto_pa_decode_after_fe.py [OPTIONS] features_filename labels_filename model_filename

### Check Performance
##### *Compute various measures of performance given a set of labeled Preaspirations and predicted Preaspirations for the same stops, optionally writing information for each stop to a CSV file.*
###### Usage: auto_pa_performance.py [OPTIONS] labeled_textgrid_list predicted_textgrid_list labeled_pa_tier predicted_pa_tier [OPTIONS]


<a name="simpleusage"/>

## Simple Usage Mode
The bash scripits in the directory 'experiments/' wraps the logic of the project with a vey simple user interface.

### Mini Setup
For this mode, the examples should be filtered using:

	$ python python_scripts/filter_examples.py DIRECTORY_PATH/formated/
  
This script is responsible for two things:
1) Deletes all the examples without a 'pre' mark in their last tier in the TextGrid file, or with
   'pre' mark but without windows big enough for the algorithm.
2) If the last tier containing 'pre', it changes its name to 'bell' for uniformity.

### Training 
To train a model using all the files (wav and corresponding TextGrid) in a directory type:

	$ ./train_model.sh DIRECTORY_PATH
  
For example:	

	$ ./train_model.sh data/abe24_abe18_plosives/formated/
  
### Decoding
To decode all the examples in a directory using a pre-trained model type:

	$ ./decode_dir.sh DIRECTORY_PATH
  
Note: This can be used only after training a model
