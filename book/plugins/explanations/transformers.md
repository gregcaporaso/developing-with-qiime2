(transformer-explanation)=
# Transformers
`Transformers` are functions for converting between formats and data types.
These transformers are typically defined along with the artifact classes that they are designed to work with, and [`q2-types`](https://github.com/qiime2/q2-types) provides a number of common types and associated transformers.
Plugins can define their own transformers.

## How are transformers used by a plugin?
`Transformers` are not called directly at any time within a plugin.
Transformations are handled by the QIIME 2 framework, as long as the appropriate `transformers` are registered.
The framework identifies the *input* `Artifact` source format for the transformation as the format registered to the `Artifact`'s {term}`artifact class`, and it identifies the transformation's destination type based on the functional annotation associated with the input in an `Action`'s registered function.
When *output* is generated by a `Method`, the framework identifies the source type for the transformation as the registered function's output annotation, and the destination format for the output `Artifact` from the output's artifact class defined in the `Action`'s registration.

For example, we can see how functional annotations define input and output formats in `q2_diversity.beta_phylogenetic`:

```python
def beta_phylogenetic(table: biom.Table,
                      phylogeny: skbio.TreeNode,
                      metric: str)-> skbio.DistanceMatrix:
```

This function requires `biom.Table` and `skbio.TreeNode` objects as input, and produces an `skbio.DistanceMatrix` object as output.

We can examine the first few lines of the `Action` registration for this function to determine the artifact classes of these input and output objects:

```python
plugin.pipelines.register_function(
    function=q2_diversity.beta_phylogenetic,
    inputs={'table': FeatureTable[Frequency],
            'phylogeny': Phylogeny[Rooted]},
    parameters={'metric': Str % Choices(beta.phylogenetic_metrics())},
    outputs=[('distance_matrix', DistanceMatrix)],
```


This pair of code examples illustrate that the `biom.Table` object used in `beta_phylogenetic` begins its life as a `FeatureTable[Frequency]` artifact, and the `skbio.TreeNode` comes from a `Phylogeny[Rooted]` artifact.
The output `skbio.DistanceMatrix` generated by `beta_phylogenetic` must be coerced to become a `DistanceMatrix` artifact.
The QIIME 2 framework takes care of all of those conversions for you, provided the appropriate transformers have been defined and registered ([which you can learn how to do here](howto-create-register-transformer)).