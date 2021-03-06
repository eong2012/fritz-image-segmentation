# fritz-image-segmentation
A Core ML compatible implementation of semantic segmentation with ICNet in Keras.

# Training Data
The model is trained on the [ADE20K dataset](http://groups.csail.mit.edu/vision/datasets/ADE20K/) provided by MIT. You can download training and validation data with this command (note this is a 3.8GB download):

```
wget http://groups.csail.mit.edu/vision/datasets/ADE20K/ADE20K_2016_07_26.zip -P /tmp/
unzip /tmp/ADE20K_2016_07_26.zip
```

The dataset contains >20,000 images and corresponding segmentation masks. Masks asign one of 150 categories to each individual pixel of the image. A list of object classes is included in this repo: [objectInfo150.txt]()

# Training
The model can be trained using the `train.py` script. It's recommended you train choose less than 20 image labels to train on as performance degrades after this point. The full 150 class labels is too much. A whitelist of class labels can be passed via the command line in a pipe separated string. Note that class labels much match those in the `objectInfo150.txt` exactly. Examples of valid whitelists are:

```
"person|wall|floor, flooring"
"chair|wall|coffee table, cocktail table|ceiling|floor, flooring|bed|lamp|sofa, couch, lounge|windowpane, window"
```

You can also set the `whitelist-threshold` argument to specify the fraction of whitelisted labels that must appear in an image for it to be used in training. For example, if 10 labels are whitelisted and the threashold is set to 0.6, at least 6 of the 10 whitelisted labels must appear in the image for it to be included.

Training the model by running:

```
python train.py \
    -d /tmp/ADEChallengeData2016/ \
    -w "chair|wall|coffee table, cocktail table|ceiling|floor, flooring|bed|lamp|sofa, couch, lounge|windowpane, window" \
    -e 10 \
    -o /tmp/icnet.h5
```

By default, a model weights checkpoint is saved every epoch. Note that only weights are saved, not the full model. This is to make it easier to build models for training vs inference.

# Converting to Core ML
The resulting Keras model can be converted using the script provided. It uses the standard `coremltools` package, but removes the additional model output nodes used for training.

```
python convert_to_coreml.py /tmp/icnet.h5 /tmp/icnet.mlmodel
```

Once you've got your Core ML model, you can use [Fritz](www.fritz.ai) to integrate, deploy, and manage it in your app. For more tutorials on mobile machine learning, check out [Heartbeat](heartbeat.fritz.ai).

# Benchmarks
On a Google Cloud Compute GPU instance with a single K80, a single epoch containing roughly 1600 768x768 images takes 20 minutes. Average cross-categorical accuracy reached >80% after 12 hours. An additional 3 hours of training with a learning rate of 0.00001 increased accuracy to ~87%. Inferences with a 768x768 model can be made at 8-9fps on an iPhone X.

# Example - Living Room Objects

<img src="https://github.com/fritzlabs/fritz-image-segmentation/blob/master/examples/living_room.jpg?raw=true" width="300" height="200">
<img src="https://github.com/fritzlabs/fritz-image-segmentation/blob/master/examples/example_image_and_mask.png?raw=true" width="300" height="200">
<img src="https://github.com/fritzlabs/fritz-image-segmentation/blob/master/examples/example_pixel_probabilities.png?raw=true" width="500" height="500">

Download the [mlmodel](https://github.com/fritzlabs/fritz-image-segmentation/blob/master/examples/icnet_768x768_living_room.mlmodel).

# Additional resources

* [Original ICNet Implementation](https://github.com/hszhao/ICNet)
* [Keras-ICNet](https://github.com/aitorzip/Keras-ICNet)
* [ICNet-tensorflow](https://github.com/hellochick/ICNet-tensorflow)
