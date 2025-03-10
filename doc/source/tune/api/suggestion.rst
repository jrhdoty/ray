.. _tune-search-alg:

Tune Search Algorithms (tune.search)
====================================

Tune's Search Algorithms are wrappers around open-source optimization libraries for efficient hyperparameter selection.
Each library has a specific way of defining the search space - please refer to their documentation for more details.
Tune will automatically convert search spaces passed to ``Tuner`` to the library format in most cases.

You can utilize these search algorithms as follows:

.. code-block:: python

    from ray import train, tune
    from ray.train import RunConfig
    from ray.tune.search.optuna import OptunaSearch

    def train_fn(config):
        # This objective function is just for demonstration purposes
        train.report({"loss": config["param"]})

    tuner = tune.Tuner(
        train_fn,
        tune_config=tune.TuneConfig(
            search_alg=OptunaSearch(),
            num_samples=100,
            metric="loss",
            mode="min",
        ),
        param_space={"param": tune.uniform(0, 1)},
    )
    results = tuner.fit()


Saving and Restoring Tune Search Algorithms
-------------------------------------------

.. TODO: what to do about this section? It doesn't really belong here and is not worth its own guide.
.. TODO: at least check that this pseudo-code runs.

Certain search algorithms have ``save/restore`` implemented,
allowing reuse of searchers that are fitted on the results of multiple tuning runs.

.. code-block:: python

    search_alg = HyperOptSearch()

    tuner_1 = tune.Tuner(
        train_fn,
        tune_config=tune.TuneConfig(search_alg=search_alg)
    )
    results_1 = tuner_1.fit()

    search_alg.save("./my-checkpoint.pkl")

    # Restore the saved state onto another search algorithm,
    # in a new tuning script

    search_alg2 = HyperOptSearch()
    search_alg2.restore("./my-checkpoint.pkl")

    tuner_2 = tune.Tuner(
        train_fn,
        tune_config=tune.TuneConfig(search_alg=search_alg2)
    )
    results_2 = tuner_2.fit()

Tune automatically saves searcher state inside the current experiment folder during tuning.
See ``Result logdir: ...`` in the output logs for this location.

Note that if you have two Tune runs with the same experiment folder,
the previous state checkpoint will be overwritten. You can
avoid this by making sure ``RunConfig(name=...)`` is set to a unique
identifier:

.. code-block:: python

    search_alg = HyperOptSearch()
    tuner_1 = tune.Tuner(
        train_fn,
        tune_config=tune.TuneConfig(
            num_samples=5,
            search_alg=search_alg,
        ),
        run_config=RunConfig(
            name="my-experiment-1",
            storage_path="~/my_results",
        )
    )
    results = tuner_1.fit()

    search_alg2 = HyperOptSearch()
    search_alg2.restore_from_dir(
      os.path.join("~/my_results", "my-experiment-1")
    )

.. _tune-basicvariant:

Random search and grid search (tune.search.basic_variant.BasicVariantGenerator)
-------------------------------------------------------------------------------

The default and most basic way to do hyperparameter search is via random and grid search.
Ray Tune does this through the :class:`BasicVariantGenerator <ray.tune.search.basic_variant.BasicVariantGenerator>`
class that generates trial variants given a search space definition.

The :class:`BasicVariantGenerator <ray.tune.search.basic_variant.BasicVariantGenerator>` is used per
default if no search algorithm is passed to
:func:`Tuner <ray.tune.Tuner>`.

.. currentmodule:: ray.tune.search

.. autosummary::
    :nosignatures:
    :toctree: doc/

    basic_variant.BasicVariantGenerator

.. _tune-ax:

Ax (tune.search.ax.AxSearch)
----------------------------

.. autosummary::
    :nosignatures:
    :toctree: doc/

    ax.AxSearch

.. _bayesopt:

Bayesian Optimization (tune.search.bayesopt.BayesOptSearch)
-----------------------------------------------------------

.. autosummary::
    :nosignatures:
    :toctree: doc/

    bayesopt.BayesOptSearch

.. _suggest-TuneBOHB:

BOHB (tune.search.bohb.TuneBOHB)
--------------------------------

BOHB (Bayesian Optimization HyperBand) is an algorithm that both terminates bad trials
and also uses Bayesian Optimization to improve the hyperparameter search.
It is available from the `HpBandSter library <https://github.com/automl/HpBandSter>`_.

Importantly, BOHB is intended to be paired with a specific scheduler class: :ref:`HyperBandForBOHB <tune-scheduler-bohb>`.

In order to use this search algorithm, you will need to install ``HpBandSter`` and ``ConfigSpace``:

.. code-block:: bash

    $ pip install hpbandster ConfigSpace

See the `BOHB paper <https://arxiv.org/abs/1807.01774>`_ for more details.

.. autosummary::
    :nosignatures:
    :toctree: doc/

    bohb.TuneBOHB

.. _tune-hebo:

HEBO (tune.search.hebo.HEBOSearch)
----------------------------------

.. autosummary::
    :nosignatures:
    :toctree: doc/

    hebo.HEBOSearch

.. _tune-hyperopt:

HyperOpt (tune.search.hyperopt.HyperOptSearch)
----------------------------------------------

.. autosummary::
    :nosignatures:
    :toctree: doc/

    hyperopt.HyperOptSearch

.. _tune-optuna:

Optuna (tune.search.optuna.OptunaSearch)
----------------------------------------

.. autosummary::
    :nosignatures:
    :toctree: doc/

    optuna.OptunaSearch

.. _skopt:

Scikit-Optimize (tune.search.skopt.SkOptSearch)
-----------------------------------------------

.. autosummary::
    :nosignatures:
    :toctree: doc/

    skopt.SkOptSearch

.. _zoopt:

ZOOpt (tune.search.zoopt.ZOOptSearch)
-------------------------------------

.. autosummary::
    :nosignatures:
    :toctree: doc/

    zoopt.ZOOptSearch

.. _repeater:

Repeated Evaluations (tune.search.Repeater)
-------------------------------------------

Use ``ray.tune.search.Repeater`` to average over multiple evaluations of the same
hyperparameter configurations. This is useful in cases where the evaluated
training procedure has high variance (i.e., in reinforcement learning).

By default, ``Repeater`` will take in a ``repeat`` parameter and a ``search_alg``.
The ``search_alg`` will suggest new configurations to try, and the ``Repeater``
will run ``repeat`` trials of the configuration. It will then average the
``search_alg.metric`` from the final results of each repeated trial.


.. warning:: It is recommended to not use ``Repeater`` with a TrialScheduler.
    Early termination can negatively affect the average reported metric.

.. autosummary::
    :nosignatures:
    :toctree: doc/

    Repeater

.. _limiter:

ConcurrencyLimiter (tune.search.ConcurrencyLimiter)
---------------------------------------------------

Use ``ray.tune.search.ConcurrencyLimiter`` to limit the amount of concurrency when using a search algorithm.
This is useful when a given optimization algorithm does not parallelize very well (like a naive Bayesian Optimization).

.. autosummary::
    :nosignatures:
    :toctree: doc/

    ConcurrencyLimiter

.. _byo-algo:

Custom Search Algorithms (tune.search.Searcher)
-----------------------------------------------

If you are interested in implementing or contributing a new Search Algorithm, provide the following interface:

.. autosummary::
    :nosignatures:
    :toctree: doc/

    Searcher

.. autosummary::
    :nosignatures:
    :toctree: doc/

    Searcher.suggest
    Searcher.save
    Searcher.restore
    Searcher.on_trial_result
    Searcher.on_trial_complete

If contributing, make sure to add test cases and an entry in the function described below.

.. _shim:

Shim Instantiation (tune.create_searcher)
-----------------------------------------
There is also a shim function that constructs the search algorithm based on the provided string.
This can be useful if the search algorithm you want to use changes often
(e.g., specifying the search algorithm via a CLI option or config file).

.. autosummary::
    :nosignatures:
    :toctree: doc/

    create_searcher
