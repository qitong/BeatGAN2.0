# BeatGAN - Generating Drum Loops via GANs

## Abstract
GANs have been used ubiquitously for generative image models. More recently, they have been applied to the task of [raw audio synthesis](https://arxiv.org/pdf/1802.04208.pdf) by Donahue et al with their WaveGAN architecture. Traditional audio generation has been done via HMMs, autoregressive models, or by applying image-based techniques (e.g. CNNs) to spectrograms (images of an audio's waveform in the time domain). Donahue et al demonstrated that by applying a 1D version of DCGAN with the same model size, one could generate high quality audio samples of human speech superior to any of the aforementioned techniques. The papers's goal was to generate speech, but the authors also applied their GAN to a small dataset of drum hits and was able to produce high quality drum samples. This project aims to explore the ability of the same model architecture to generate complex drums patterns, namely beats. In particular, using the architecture presented in the paper, a GAN is trained to generate the first bar of drum 4 bar drum patterns. The model is able to produce samples of nearly the same quality as those generated by the original WaveGAN paper. Furthermore, using a simple similarity metric and manual testing, it appears the majority of the GAN's outputs are legitimately new beats rather than clones of the training data.

## Generating Loops
WaveGAN's D and G use the same number of parameters as DCGAN, so the generator outputs vectors of shape _(16384, c)_, where _c_ is the number of audio channels. This was likely done because a large model would have required significantly more training data. 

44.1 khz is the standard sample rate for nearly all digital audio. Spotify, YouTube, etc all stream audio at this rate. However, at a sample rate of 44.1khz, the generator's output is only equal to .37 seconds of audio (16384/44100). This is far too short to contain any meaningful audio. To get a meaningful output size, I use a sample rate of 14.7khz (44.1/3). This also means that all I have to do to resample the audio is take every third data point. 

At 14.7khz the generator can produce about 1.11 seconds of audio. That's still to small to contain an entire four bar pattern. However, the vast majority of beat patterns follow an AAAB, AABB, or ABAB structure, meaning that knowing the first bar of the pattern means you have at least half of the full pattern. Luckily enough, at 120-125 bpm, the first bar of a 4/4 drum pattern is just under one second long! This means our goal can be to generate 4/4 loops at 120-125 bpm. 

## The Training Data
I trained the model using the [ULTIMATE DRUM LOOPS](splice.com/sounds/toolroom-records/ultimate-drum-loops) sample pack of 434 samples. Since we're only using the first bar, 434 samples means only 7 minutes of audio. The smallest dataset used in WaveGAN was 20 minutes of data (the rest were above an hour), so more data was needed. I wrote a white noise function to that shifts the original sample values (up or down) by <=.11%. I applied the white noise function five times to each original ULTIMATE DRUM LOOPS sample to get my training set of 2170 samples (35 minutes worth of audio). 

## Training the Model
I used the same model architecture and hyperparameters as the BeatGAN paper. The original paper ran the model for about 200k iterations for each dataset. For my dataset, this corresponded to about 6100 epochs, so I ran the model for that period of time to produce my final weights. These weights can be found in the **weights** folder at the top level. I used them for all output evaluation tasks. 

## Output Evaluation
One potential issue when training GANs is the GAN teaching itself to simply output the training data instead of outputting new values. To ensure BeatGAN's output samples were truly unique, I used a simple similarity score to measure the closeness of two audio waveforms (function compute_similarity_score in the code). For two vectors _A,B_ of the same length, _sum((A-B)^2))/sum(A^2)_ is the similarity of the vectors (lower score means more similar). The folder **similarity_metric_examples** at the top level provides some examples of generated outputs evaluated against this metric on the input data (each example has a README). As shown in the examples, the score corresponds pretty closely with the similarity of two audio files. 

After experimentation, I found a score of 0.1 or lower to be more or less indicative that two audio files were almost the same. Using the generator weights obtained from my 6100 epoch run (see above section), I found only 30% to 40% of the generated outputs were similar to an example in the training set. This means the majority of generated outputs are actually novel beats. 

## Further Work
The main extension to this work is finding a way to produce meaningfully long samples at a high quality sample rate (e.g. 44.1khz). This will undoubtedly require a larger model and thus more data. 

## Running the Model Yourself
Environment is described in the environment-gpu.yml. To run the model (once env is setup), put the training data into the ULTIMATE_DRUM_LOOPS folder (instructions provided in that folder's readme), then run the first six cells in beat_gan.ipynb (every cell up to and including the one that computes the similarity score). Make sure you clone the wavfile24.py module in the top level directory.  


