=====================================
Pruning
=====================================

Pruning is a technique for reducing the size of a neural network by removing unimportant weights. The idea is to train a model, identify the unimportant weights, and then remove them. The resulting model will be smaller and faster, but will have the same accuracy as the original model.

.. image:: https://shrinkbench.github.io/diagram.svg
   :alt: Diagram of the pruning process
   :align: center

To learn more about what neural network pruning is, see this excellent `blog post <https://towardsdatascience.com/neural-network-pruning-101-af816aaea61>`_ by `Hugo Tessier <https://medium.com/@hugo.tessier>`_.

Source code for the pruning API can be found in the ``pruning`` package.

-----------------------
Pruning Strategies
-----------------------

There are many different ways to prune a neural network. In the ``pruning.strategies`` package, we provide implementations of some of the most common pruning strategies. These include:

^^^^^^^^^^^^^^^^^^^^
Global Pruning
^^^^^^^^^^^^^^^^^^^^
- **Random**: Randomly zeroes out a fraction of weights across the entire network.
- **Global Magnitude**: Prunes the least important weights across the entire network based on their absolute values.
- **Global Magnitude-Gradient**: Prunes weights globally based on a combination of their absolute values and gradients.
- **Global Activation-Magnitude**: Prunes weights globally based on a combination of their absolute values and corresponding activations.

^^^^^^^^^^^^^^^^^^^^
Layerwise Pruning
^^^^^^^^^^^^^^^^^^^^
- **Layerwise Magnitude**: Prunes the least important weights within each individual layer based on their absolute values.
- **Layerwise Magnitude-Gradient**: Prunes weights within each layer based on a combination of their absolute values and gradients.
- **Layerwise Activation-Magnitude**: Prunes weights within each layer based on a combination of their absolute values and corresponding activations.

^^^^^^^^^^^^^^^^^^^^
Channel Pruning
^^^^^^^^^^^^^^^^^^^^
- **Weight Norm Channel**: Prunes channels in Conv2D and optionally Linear layers based on the norm of their weights.
- **Activation Norm Channel**: Prunes channels based on the norm of their output activations, offering a dynamic pruning strategy.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Implementing Your Own Pruning Strategy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To implement a new pruning strategy, you need to extend one or more base classes from ``LayerPruning`` or ``VisionPruning``. Then, you either need to implement ``layer_masks`` or ``model_masks`` depending on whether you want to prune layerwise or globally. For example, see the following implementations of global and layerwise magnitude pruning:

.. code-block:: python

    class GlobalMagWeight(VisionPruning):

        def model_masks(self):
            importances = map_importances(np.abs, self.params())
            flat_importances = flatten_importances(importances)
            threshold = fraction_threshold(flat_importances, self.fraction)
            masks = importance_masks(importances, threshold)
            return masks


    class LayerMagWeight(LayerPruning, VisionPruning):

        def layer_masks(self, module):
            params = self.module_params(module)
            importances = {param: np.abs(value) for param, value in params.items()}
            masks = {param: fraction_mask(importances[param], self.fraction)
                    for param, value in params.items() if value is not None}
            return masks

----------------------------
Pruning Strategies Explained
----------------------------

Here is a brief overview of the pruning strategies available:

- **GlobalMagWeight (Global Magnitude Weight Pruning)**: Prunes the least important weights across the entire network based on their absolute values.
- **LayerMagWeight (Layerwise Magnitude Weight Pruning)**: Prunes weights within individual layers based on their absolute values.
- **GlobalMagGrad (Global Magnitude-Gradient Pruning)**: Prunes weights across the entire network based on a combination of their absolute values and gradients.
- **LayerMagGrad (Layerwise Magnitude-Gradient Pruning)**: Prunes weights within layers based on a combination of their absolute values and gradients.
- **GlobalMagAct (Global Activation-Magnitude Pruning)**: Prunes weights across the network based on a combination of their absolute values and corresponding activations.
- **LayerMagAct (Layerwise Activation-Magnitude Pruning)**: Prunes weights within layers based on a combination of their absolute values and corresponding activations.
- **RandomPruning**: Randomly zeroes out a fraction of weights across the entire network.


-------------------------
Using the Pruning Package
-------------------------

The pruning package allows for easy application of different pruning strategies to neural network models.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Listing Available Pruning Strategies
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the `list_pruning_strategies` function to list all available pruning strategies:

.. code-block:: python

    from visualml.optimization import list_pruning_strategies

    strategies = list_pruning_strategies()
    print(strategies)

    output: ['GlobalMagWeight', 'LayerMagWeight', 'GlobalMagGrad', 'LayerMagGrad', 'GlobalMagAct', 'LayerMagAct', 'RandomPruning']

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Applying Pruning to a Model
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To prune a model, use the `prune` function by specifying the model, strategy, and compression ratio:

.. code-block:: python

    from visualml.optimization import prune

    model = ...
    pruned_model = prune(model, strategy='RandomPruning', compression=1)


