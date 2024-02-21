# Fine-tuning a classifier on the Activations from an LLM

We are aiming to train a small classifier on top of the activations of intermediate layers for the LLaMA-2 language model hosted on Vector's cluster. As an example we'll work on the IMDB sentiment classification task. It's a boolean task that asks the model to classify the sentiment associated with a short movie review. The dataset consists of 25,000 highly polar movie reviews for training, and another 25,000 for testing.

__NOTE__: We also have activations extracted from the final layer of the OPT-175B model. While this model is no currently hosted on the Vector, they still hold illustrative value and can be loaded in the `train_on_activations.ipynb` notebook.

Sample Review (Please forgive language):

"This movie sucked. It really was a waste of my life. The acting was atrocious, the plot completely implausible. Long, long story short, these people get "terrorized" by this pathetic "crazed killer", but completely fail to fight back in any manner. And this is after they take a raft on a camping trip, with no gear, and show up at a campsite that is already assembled and completely stocked with food and clothes and the daughters headphones. Additionally, after their boat goes missing, they panic that they're stuck in the woods, but then the daughters boyfriend just shows up and they apparently never consider that they could just hike out of the woods like he did to get to them. Like I said, this movie sucks. A complete joke. Don't let your girlfriend talk you into watching it."

Label: Negative

## VENV Installation

Both of the notebooks in this folder require certain dependencies. Before spinning up the notebooks on a GPU through the cluster, following the instructions in the top level [README](/README.md), make sure you source the `prompt_engineering` environment with the command

```bash
source /ssd003/projects/aieng/public/prompt_engineering/bin/activate
```

If you're running the notebooks launched through the Jupyter Hub, simply select `prompt_engineering` from the available kernels and you should be good to go.

If you want to create your own environment that you can modify, then you can do so by creating your own virtual environment with the command
```
python -m venv <name_of_venv>
```
then
```
source <name_of_venv>/bin/activate
```
and finally
```
pip install -r requirements.txt
```
where `requirements.txt` is housed at the top level of the repository

## Pickle Files

As part of the repository, we have included two sets of pkl files with precomputed activations for LLaMA-2 13B (and OPT-175B, see comment at the beginning of this README) in the `resources` folder. The files with names not containing `with_prompts` are those computed without preconditioning the language model with instruction and few-shot demonstration prompts, while the files with names that contain that string correspond to activations obtained using those components.

It takes a fair bit of time to compute the activations. So computing your own will take some patience. Alternatively, you can experiment with LLaMA-2 7B for faster activation production.

## Activation Computation

The first step is to compute activations for a set of training inputs and the same level of activations for testing inputs. Because the dataset is so large and the model is quite heavy, we'll just consider a few-shot set of training inputs (100 samples) and a randomly sampled set of points from the test dataset (300 samples).

These activations are stored in the resources folder and we'll train/test a classifier using the activations vectors in another notebook. Because the model is so expressive, this few-shot set of training examples should still perform fairly well on the test set. The notebook to compute these is `compute_activations.ipynb`.

We also want to consider whether training with a prompt as part of the input helps the downstream model perform the task more accurately. So we'll save activations with and without a few-shot prompt as part of the input. Finally, we consider which layer in the model houses the most information relevant to the completion of this task.

## Classifier Training

After pre-computing the activations with and without prompting, we train a simple DNN to perform sentiment classification on different training sets, including those generated with and without prompts as part of the input. We compare performance to determine whether the prompting helped produce better downstream performance. The notebook to do the training is `train_on_activations.ipynb`. We also consider which layers of the model contain the most "relevant" information for the task.
