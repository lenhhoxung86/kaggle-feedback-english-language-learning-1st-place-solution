# Feedback Prize - English Language Learning - 1st Place Solution

It's 1st place solution to Kaggle competition: https://www.kaggle.com/competitions/feedback-prize-english-language-learning

This repo contains the code we used to train the models, while the solution writeup is available here: https://www.kaggle.com/competitions/feedback-prize-english-language-learning/discussion/369457

### HARDWARE: (The following specs were used to create the original solution)

Almost all models were trained using Paperspace Free A6000 Machine.

* OS: Ubuntu 20.04.4 LTS
* CPU: Intel Xeon Gold 5315Y @3.2 GHz, 8 cores
* RAM: 44Gi 
* GPU: 1 x NVIDIA RTX A6000 (49140MiB)


### SOFTWARE (python packages are detailed separately in `requirements.txt`):

* Python 3.9.13
* CUDA 11.6
* nvidia drivers v510.73.05

### Training (Yevhenii Part)

* Download training data from https://www.kaggle.com/competitions/feedback-prize-english-language-learning
* Download OOFs, previous competition data, and pseudo labels from https://www.kaggle.com/datasets/evgeniimaslov2/feedback3-additional-data
* Extract data to `./data` directory (directory structure can be checked in `yevhenii_directory_structure.txt`). Pseudo labels and oofs also can be generated by running the scripts below
* To train models from `src` folder run: 
  * `train_first_step.sh` - for models from model2 to model50, this script will do the following: 
    * Pretrain model on pseudo labels (some models used pseudo labels only from the previous model and some from the ensemble). In case it's model2 this step is skipped (there are no pseudo labels yet). You can check the exact parameters and data that were used to train a model on this step in the `config` directory, for example,  `model15_pretraining_training_config.yaml`
    * Fine-tune model on true labels. You can check the exact parameters and data that were used to train a model on this step in the `config` directory, for example,  `model15_training_config.yaml`
    * Make model OOF
    * Generate pseudo labels
    * Make model-wise weighted ensemble to get better pseudo labels
  * `train_second_step.sh` - same loop as in first_step, but with few additions:
    * Pseudo labels from Rohit's models were added. Download or train Rohit's models and run `rohit_pseudo.sh`
    * Column-wise weighted ensemble was used
    * Some models are training on concatenated train+pseudo in one step
    * model71 instead of true labels used this competition data pseudo labels

Note: train_first_step.sh and train_second_step.sh scripts will rewrite existing OOFs and pseudo labels files, and rename serialized copies of the models

You also can train a model by running `python train.py` with arguments:
* `config_name` - name of config from `CONFIGS_DIR_PATH` directory specified in SETTING.json
* `run_id` - model_id, should be the same across folds. For example, I used `modelN_pretrain` for the pretraining step and `modelN` for finetuning step
* `debug` - if True, the script will select only 50 samples for training and validation dataframes
* `use_wandb` - if True, the script will log metrics to Weights and Biases
* `fold` - fold to train

To get pseudo labels and OOFs I used `python inference.py` with arguments:
* `model_dir_path` - path to model directory, ../models/model2 for example
* `mode`:
  * oofs - script will load dataframe from TRAIN_CSV_PATH (specified in SETTINGS.json) and save oof to OOFS_DIR_PATH
  * prev_pseudolabels - script will load dataframe PREVIOUS_DATA_PSEUDOLABELS_DIR_PATH and save pseudolabels for previous competition data to f'{PREVIOUS_DATA_PSEUDOLABELS_DIR_PATH}/{model_id}_pseudolabels' dir
  * curr_pseudolabels - script will load dataframe CURRENT_DATA_PSEUDOLABELS_DIR_PATH and make pseudolabels for this competition data to f'{CURRENT_DATA_PSEUDOLABELS_DIR_PATH}/{model_id}_pseudolabels' dir
  * submission - script will load dataframe TEST_CSV_PATH and make save submission to SUBMISSIONS_DIR_PATH
* `debug` - if True, script will select only 50 samples for training and validation dataframes

To find models weights in the ensemble I used `./notebooks/find_ensemble_weights.ipynb` notebook.
* All weight used are stored in `ensemble_weights` dictionary in `./src/make_pseudolabels_ensemble.py`
* `./src/make_pseudolabels_ensemble.py` will load pseudo labels from `PREVIOUS_DATA_PSEUDOLABELS_DIR_PATH` directory (or `CURRENT_DATA_PSEUDOLABELS_DIR_PATH` in case if `ensemble_id` = `current_train_ensemble_4434`) and save ensembled pseudolabels in the same directory, in `ensemble_id` subfolder

Note: Ensemble weights mentioned above were used only to get pseudo labels, you can read how final inference weights were found in Rohit's section
 
### Training (Rohit Part)

* Downlaod additional training data from https://www.kaggle.com/datasets/rohitsingh9990/training-data and extract it to `./data` directory. and from https://www.kaggle.com/datasets/rohitsingh9990/fb3-pl-models and extract it to `./data/pseudolabels` directory.
* To retrain models used for Pseudo labeling move to `src_rohit` directory and run: `./train_pl.sh` this will rewrite pseudo models in `./data/pseudolabels` directory. [Skip this step if you don't wish to retrain pseudo models]

> Files Description - 
  * `train_5folds.csv` - train file with fold column added using MultilabelStratifiedKFold
  * `train_feedback1.csv` - FeedBack First Comp. train file
  * `fb3_pl_fold0.csv` to `fb3_pl_fold4.csv` - FB1 comp. pseudo labels data
  * `pl_train.csv` - Pseudo Labels generated on this comp. train data [train.csv]
  * `train_pl_df.csv` - New train labels generated by taking Average of train.csv and pl_train.csv targets.

Generating Pseudo Labels: 
-  To generate fb3_pl_fold0.csv to fb3_pl_fold4.csv -> run: `./src_rohit/notebooks/generate_fb1_pls.ipynb`
-  To generate  pl_train.csv and train_pl_df.csv -> run: `./src_rohit/notebooks/generat_train_pls.ipynb`

To train models: 
* To train final models, move to `src_rohit` directory and run: `./src_rohit/train.sh`. 
* For 3 embedding models, run: https://www.kaggle.com/code/rohitsingh9990/rapids-svr-embeddings/data?scriptVersionId=112660664, download and extract output files to `./models`

Ensemble Weights:
* to get ensemble weights run: `./notebooks/weight-tuning-optuna.ipynb`


### Inference

Final inference kernel is available here: https://www.kaggle.com/code/rohitsingh9990/merged-submission-01?scriptVersionId=111953356
