# Set up the conda environment

conda create -n ss python=3.9 cmake=3.14.0 -y
conda activate ss



#Install dependences (habitat-lab/habitat-sim)
git clone https://github.com/facebookresearch/habitat-sim.git
cd habitat-sim
git checkout v0.1.7
python setup.py install --headless --audio

git clone https://github.com/facebookresearch/habitat-lab.git
cd habitat-lab
git checkout v0.1.7
pip install -e .

# Install SoundSpaces
git clone https://github.com/facebookresearch/sound-spaces.git
cd sound-spaces
git checkout 8b5902790ef659540c12fbc80b7228e17f81bb94
pip install -e .

cd sound-spaces
mkdir data && cd data
mkdir scene_datasets && cd scene_datasets
mkdir replica 

#Download Replica3D Datatset
git clone https://github.com/facebookresearch/Replica-Dataset.git
cd Replica-Dataset
./download.sh ~/sound-spaces/data/scene_datasets/replica 

# Download additional data to sound-spaces/data folder
wget http://dl.fbaipublicfiles.com/SoundSpaces/binaural_rirs.tar && tar xvf binaural_rirs.tar
wget http://dl.fbaipublicfiles.com/SoundSpaces/metadata.tar.xz && tar xvf metadata.tar.xz
wget http://dl.fbaipublicfiles.com/SoundSpaces/sounds.tar.xz && tar xvf sounds.tar.xz
wget http://dl.fbaipublicfiles.com/SoundSpaces/datasets.tar.xz && tar xvf datasets.tar.xz
wget http://dl.fbaipublicfiles.com/SoundSpaces/pretrained_weights.tar.xz && tar xvf pretrained_weights.tar.xz

# Scene Observations for Replica Datatset (Go to sound-spaces directory)
python scripts/cache_observations.py

# Test using pre-trained model
python ss_baselines/av_nav/run.py --run-type eval --exp-config ss_baselines/av_nav/config/audionav/replica/test_telephone/audiogoal_depth.yaml EVAL_CKPT_PATH_DIR data/pretrained_weights/audionav/av_nav/replica/heard.pth 

# Demo Video for Pre-Trained Model 

python ss_baselines/av_nav/run.py --run-type eval --exp-config ss_baselines/av_nav/config/audionav/replica/test_telephone/audiogoal_depth.yaml --model-dir data/models/replica/audiogoal_depth EVAL_CKPT_PATH_DIR data/pretrained_weights/audionav/av_nav/replica/heard.pth VIDEO_OPTION [\"disk\"] TASK_CONFIG.SIMULATOR.USE_RENDERED_OBSERVATIONS False TASK_CONFIG.TASK.SENSORS [\"POINTGOAL_WITH_GPS_COMPASS_SENSOR\",\"SPECTROGRAM_SENSOR\",\"AUDIOGOAL_SENSOR\"] SENSORS [\"RGB_SENSOR\",\"DEPTH_SENSOR\"] EXTRA_RGB True TASK_CONFIG.SIMULATOR.CONTINUOUS_VIEW_CHANGE True DISPLAY_RESOLUTION 512 TEST_EPISODE_COUNT 1

# Train Model from Scratch
python ss_baselines/av_nav/run.py --exp-config ss_baselines/av_nav/config/audionav/replica/train_telephone/audiogoal_depth.yaml --model-dir data/models/replica/audiogoal_depth

# Test Model from Scratch (XXX is the checkpoint number you want to evaluate)
python ss_baselines/av_nav/run.py --run-type eval --exp-config ss_baselines/av_nav/config/audionav/replica/test_telephone/audiogoal_depth.yaml --model-dir data/models/replica/audiogoal_depth EVAL_CKPT_PATH_DIR data/models/replica/audiogoal_depth/data/ckpt.XXX.pth

# Demo Video for the new trained model (XXX is the checkpoint number you want to evaluate)
python ss_baselines/av_nav/run.py --run-type eval --exp-config ss_baselines/av_nav/config/audionav/replica/test_telephone/audiogoal_depth.yaml --model-dir data/models/replica/audiogoal_depth EVAL_CKPT_PATH_DIR data/models/replica/audiogoal_depth/data/ckpt.XXX.pth VIDEO_OPTION [\"disk\"] TASK_CONFIG.SIMULATOR.USE_RENDERED_OBSERVATIONS False TASK_CONFIG.TASK.SENSORS [\"POINTGOAL_WITH_GPS_COMPASS_SENSOR\",\"SPECTROGRAM_SENSOR\",\"AUDIOGOAL_SENSOR\"] SENSORS [\"RGB_SENSOR\",\"DEPTH_SENSOR\"] EXTRA_RGB True TASK_CONFIG.SIMULATOR.CONTINUOUS_VIEW_CHANGE True DISPLAY_RESOLUTION 512 TEST_EPISODE_COUNT 1



