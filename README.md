# Dynamic Routing Between Capsules

A barebones CUDA-enabled PyTorch implementation of the CapsNet architecture in the paper "Dynamic Routing Between Capsules" by [Kenta Iwasaki](https://github.com/iwasaki-kenta) on behalf of Gram.AI.

Training for the model is done using [TorchNet](https://github.com/pytorch/tnt), with MNIST dataset loading and preprocessing done with [TorchVision](https://github.com/pytorch/vision).

## Description

> A capsule is a group of neurons whose activity vector represents the instantiation parameters of a specific type of entity such as an object or object part. We use the length of the activity vector to represent the probability that the entity exists and its orientation to represent the instantiation paramters. Active capsules at one level make predictions, via transformation matrices, for the instantiation parameters of higher-level capsules. When multiple predictions agree, a higher level capsule becomes active. We show that a discrimininatively trained, multi-layer capsule system achieves state-of-the-art performance on MNIST and is considerably better than a convolutional net at recognizing highly overlapping digits. To achieve these results we use an iterative routing-by-agreement mechanism: A lower-level capsule prefers to send its output to higher level capsules whose activity vectors have a big scalar product with the prediction coming from the lower-level capsule.

Paper written by Sara Sabour, Nicholas Frosst, and Geoffrey E. Hinton. For more information, please check out the paper [here](https://arxiv.org/abs/1710.09829).

__Note__: Affine-transformations for the data augmentation stage have not been implemented yet. This implementation only provides an efficient implementation for the dynamic routing, example CapsNet architecture, and squashing functions mentioned in the paper.

## Requirements

* Python 3
* PyTorch
* TorchVision
* TorchNet
* TQDM

## Usage

**Step 1** Configure network architecture to your preferences inside `capsule_network.py`.

```python
self.conv1 = nn.Conv2d(in_channels=1, out_channels=256, kernel_size=9, stride=1)

self.primary_capsules = CapsuleLayer(num_capsules=8, num_route_nodes=-1, in_channels=256, out_channels=32, kernel_size=9, stride=2)

self.digit_capsules = CapsuleLayer(num_capsules=10, num_route_nodes=32 * 6 * 6, in_channels=8, out_channels=16)

self.decoder = nn.Sequential(
	nn.Linear(16, 512),
	nn.ReLU(inplace=True),
	nn.Linear(512, 1024),
	nn.ReLU(inplace=True),
	nn.Linear(1024, 784),
	nn.Sigmoid()
)
```

**Step 2** Adjust the number of training epochs and and batch sizes inside `capsule_network.py`.

```python
engine.train(h, get_iterator(True), maxepoch=30, optimizer=optimizer)
```

```python
def get_iterator(mode):
	dataset = MNIST(root='./data', download=True, train=mode)
	data = getattr(dataset, 'train_data' if mode else 'test_data')
	labels = getattr(dataset, 'train_labels' if mode else 'test_labels')
	tensor_dataset = tnt.dataset.TensorDataset([data, labels])

	return tensor_dataset.parallel(batch_size=100, num_workers=4, shuffle=mode)
```

**Step 3** Start training. The MNIST dataset will be downloaded if you do not already have it in the same directory the script is run in.

```console
$ python3 capsule-networks.py
```

## Benchmarks

Haven't fully benchmarked the entire model yet; not enough computing power available. Accuracy so far at the time of writing (7th epoch) is capped at 99.28%.

![Training progress.](media/Progress.png)

Default PyTorch Adam optimizer hyperparameters were used with no learning rate scheduling. Epochs with batch size of 100 takes ~3 minutes on a Razer Blade w/ GTX 1050. 

## TODO

* Affine transformations for the data augmentation stage.
* Loss & accuracy charting.
* Model parameter serialization.
* Decoder reconstruction plotting.

## Credits

Primarily referenced these two TensorFlow and Keras implementations:
1. [Keras implementation by @XifengGuo](https://github.com/XifengGuo/CapsNet-Keras)
2. [TensorFlow implementation by @naturomics](https://github.com/naturomics/CapsNet-Tensorflow)

Major thanks to [@InnerPeace-Wu](https://github.com/InnerPeace-Wu) for a [discussion on the dynamic routing procedure](https://github.com/XifengGuo/CapsNet-Keras/issues/1) outlined in the paper.

## Contact/Support

Gram.AI is currently heavily developing a wide number of AI models to be either open-sourced or released for free the community, and hence cannot guarantee complete support for this work.

If any issues come up with the usage of this implementation however, or if you would like to contribute in any way, please feel free to send an e-mail to [kenta@gram.ai](kenta@gram.ai) or open a new GitHub issue on this repository.