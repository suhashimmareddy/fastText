---
id: autotune
title: Automatic hyperparameter optimization
---

As we saw in [the tutorial](/docs/en/supervised-tutorial.html#more-epochs-and-larger-learning-rate), finding the best hyperparameters is crucial for building efficient models. However, searching the best hyperparameters manually is difficult. Parameters are dependent and the effect of each parameter vary from one dataset to another.

FastText's autotune feature allows you to find automatically the best hyperparameters for your dataset.

# How to use it

In order to activate hyperparameter optimization, we must provide a validation file with the `-autotune-validation` argument.

For example, using the same data as our [tutorial example](/docs/en/supervised-tutorial.html#our-first-classifier), the autotune can be used in the following way:

```sh
>> ./fasttext supervised -input cooking.train -output model_cooking -autotune-validation cooking.valid
```

Then, fastText will search the hyperparameters that gives the best f1-score on `cooking.valid` file:
```sh
Progress: 100.0% Trials:   27 Best score:  0.406763 ETA:   0h 0m 0s
```

Now we can test the obtained model with:
```sh
>> ./fasttext test model_cooking.bin data/cooking.valid
N       3000
P@1     0.666
R@1     0.288
```

By default, the search will take 5 minutes. You can set the timeout in seconds with the `-autotune-duration` argument. For example, if you want to set the limit to 10 minutes:

```sh
>> ./fasttext supervised -input cooking.train -output model_cooking -autotune-validation cooking.valid -autotune-duration 600
```

While autotuning, fastText displays the best f1-score found so far. If we decide to stop the tuning before the time limit, we can send one `SIGINT` signal (via `CTLR-C` for example). FastText will then finish the current training, and retrain with the best parameters found so far.



# Constrain model size

As you may know, fastText can compress the model with [quantization](/docs/en/cheatsheet.html#quantization). However, this compression task comes with its own [hyperparameters](/docs/en/options.html) (`-cutoff`, `-retrain`, `-qnorm`, `-qout`, `-dsub`) that have a consequence on the accuracy and the size of the final model.

Fortunately, autotune can also find the hyperparameters for this compression task while targeting the desired model size. To this end, we can set the `-autotune-modelsize` argument:

```sh
>> ./fasttext supervised -input cooking.train -output model_cooking -autotune-validation cooking.valid -autotune-modelsize 2M
```

This will produce a `.ftz` file with the best accuracy having the desired size:
```sh
>> ls -la model_cooking.ftz
-rw-r--r--. 1 celebio users 1990862 Aug 25 05:39 model_cooking.ftz
>> ./fasttext test model_cooking.ftz data/cooking.valid
N       3000
P@1     0.57
R@1     0.246
```


# How to set the optimization metric?

By default, autotune will test the validation file you provide, exactly the same way as `./fasttext test model_cooking.bin cooking.valid` and try to optimize to get the highest [f1-score](https://en.wikipedia.org/wiki/F1_score).

But, if we want to optimize the score of a specific label, say `__label__baking`, we can set the `-autotune-metric` argument:

```sh
>> ./fasttext supervised -input cooking.train -output model_cooking -autotune-validation cooking.valid -autotune-metric f1:__label__baking
```

This is equivalent to manually optimize the f1-score we get when we test with `./fasttext test-label model_cooking.bin cooking.valid | grep __label__baking` in command line.

Sometimes, you may be interested in predicting more than one label. For example, if you were optimizing the hyperparameters manually to get the best score to predict two labels, you would test with `./fasttext test model_cooking.bin cooking.valid 2`. You can also tell autotune to optimize the parameters by testing two labels with the `-autotune-predictions` argument.
