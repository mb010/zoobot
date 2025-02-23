.. _representations_guide:

Representations
===============

Representations are vectors that summarise your input data (here, galaxies).
In the context of Zoobot, our models learn to convert the image data (pixels) into representations before using those representations to make predictions.
You might like to extract the representations learned by a model - perhaps to use it directly for some new task, like a similarity search.
Here's how.

.. note:: 

    If you would like the representations of the trained GZ DECaLS model on DECaLS DR5 galaxies or on GZ2 galaxies, you can find them here (TODO Zenodo link).
    These were used for the morphology tools paper. If you need your own model and representations, read on.

Training a New Model
--------------------

Representations (in this context) are simply the activations of the model before the final dense layer(s).
Training a model (optimizing the weights) teaches it to create useful representations of input images.
Train exactly like you normally would. The representations may be more useful if you train on a broad multi-question task, like answering the GZ decision tree.
See :ref:`reproducing_decals` for a guide to training a new model.

We have published pretrained weights for models trained on GZ DECaLS - see :ref:`datanotes`. 
You could start with these and calculate representations on some new galaxies.
See ``make_predictions_loop.py`` for how to load the weights, and below for how to calculate representations.

You might also want to start from a pretrained model and use finetuning to get the best representation for your problem.
See ``finetune_advanced.py`` for an example. This adds some complexity, so we suggest trying with our pretrained weights first.

Extracting the Representation
-----------------------------

Extracting the representation is really just making a prediction, but without the final dense layers of the model.
``make_predictions_loop.py`` includes a working example for you to copy.

``make_predictions_loop.py`` can be used for three different kinds of predictions, depending on what you comment and uncomment.
To save the representations, uncomment the block marked "For saving the activations (representations)" and comment the others.
This configures the model like so:

- Defines a base model with no head (``include_top=False``) as we don't want the final dense layer for making volunteer predictions.
- Adds a new top with just ``GlobalAveragePooling2D``. This is the last layer we want to include. It just averages the first two axes of the previous (7x7)x1280-dim activations to 1280-dim.
- Groups the base model and new top with ``tf.keras.Sequential``
- Sets the ``label_cols`` to be as long as the dimension of the representation (e.g. 1280 for EfficientNet) rather than the usual answers (e.g. 34)

As always, remember to check ``run_name`` and any file paths.

``make_predictions_loop.py`` will then save the representations for each galaxy to files like {run_name}_{index}.csv.
These files are a bit awkward as they include lots of numbers like ``[[0.4, ...]]``.
Remove the brackets with ``predictions/reformat_predictions.py``.

Finally, compress the 1280-dim representation into a lower dimensionality using PCA with ``representations/compress_representations.py``.
The compressed representation is mathematically very similar (PCA should preserve most of the interesting variation) but much easier to work with.

