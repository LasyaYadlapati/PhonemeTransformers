# DSC 180 Report
## Task
In the past few weeks, we have worked on replicating [From Babble to Words](https://arxiv.org/pdf/2410.22906), by Goriely et. al. This repo is a fork of theirs, with minor changes for our own experimentation and to make it run properly on our cluster.

## Phoneme-based language models
Most language models are trained on *graphemes*, or written language(s). However, there is a marked disconnect between almost all languages' writing system and their speech system. In English, for example, ⟨c⟩ can make many different sounds depending on the context.

Thus motivates training directly on *phonemes*, symbols representing sounds. Such models have been used in analysis of phoneme distributions, lyric generation, and text-to-speech tools. Goriely et. al. attempt to *pretrain* a language model on phonemes for downstream language tasks, using a grapheme-to-phoneme conversion tool to transliterate directly from graphemes.

## Our work
We've mainly been replicating Goriely et. al.'s work, training our own models to evaluate on language understanding tasks. In the past week, we've focused on running hyperparameter sweeps on the model with wandb, adjusting hyperparameters such as learning rate and max training steps to find the optimal set. To do this, we've had to modify `train.py`, `setup.sh`, and `src/trainer.py`, as well as create `run_sweep.py` as an entry point for the sweeps. We've also had to change some of the configs in the `conf` directory, namely the tokenizers, to fit the updated IPA tokenizers. Our sweeps can be viewed at [https://wandb.ai/lemn-lab/phoneme-sweeps/sweeps](https://wandb.ai/lemn-lab/phoneme-sweeps/sweeps).

## Replication
The original README file from the repository has been kept unchanged in Original-README.md, where the authors have included instructions for running their code. To replicate what we did on a kubernetes cluster:

1. In `scripts/sweep.yaml`, define the configuration for your sweep. This repo uses a hydra config; check `conf/config.yaml` for available hyperparameters. These should be reformatted in `sweep.yaml`, for example:

```
trainer:
    lr: 1e-3
```
in `config.yaml` translates to the argument
```
trainer.lr:
```
in sweeps.yaml. More info [here](https://docs.wandb.ai/models/sweeps/define-sweep-configuration).

2. In `run_sweep.py`, change the `PROJECT_NAME` argument to a wandb project to which you want the sweep to write its results.

3. Locate `phoneme-sweep-job-example.yaml`. This is the job file for running a sweep. Make sure you put in your own Huggingface read and write tokens, as well as your own wand api token. Create a wandb project, and set the wandb namespace to your username, or the namespace in which you made the project in step 2. Finally, create a pvc with at least 4GiB of space, and mount the job to that pvc.

4. Navigate to this directory, then run `kubectl apply -f ./phoneme-sweep-job-example.yaml` to start the job.

Alternatively, to run locally, do steps 1 and 2, then run `python run_sweep.py`. 
