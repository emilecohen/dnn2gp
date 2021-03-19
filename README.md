This repository gathers the work on the implementation of the paper Approximate Inference Turns Deep Networks into Gaussian Processes, studied during the course of Bayesian Machine Learning provided by Rémi Bardenet and Julyan Arbel at Ecole Normale Supérieure Paris-Saclay (MVA Master).

*Work performed by Pierre Clavier, Emile Cohen, Julia Linhart*



# From Deep Neural Network to Gaussian Process (dnn2gp)
![dnn2gp schema](https://github.com/team-approx-bayes/dnn2gp/blob/master/figures/dnn2gp_schema.png)

This repository contains code to reproduce results for the paper *Approximate Inference Turns Deep Networks into Gaussian Processes*

### Computing and Visualizing Linear Model and GP

### Reproducing Model Selection Experiments

The results are readily available in directory `/results` and can be reproduced by running
```bash
python marglik.py --name of_choice
```

This will produce both toy and real world experiments and save the corresponding measurements
to the results directory with a new filename. The plots can then be generated by running
```bash
python marglik_plots.py --name original  # use our result files
python marglik_plots.py --name of_choice  # use your result files
```

### Computing and Visualizing Kernels and Predictive Distribution

We use pre-trained models that are saved in the `/models` directory and are trained on 
either CIFAR-10 or MNIST. The plots produced in our paper can be reproduced by running
```bash
python kernel_and_predictive.py
```
To reproduce the plots of kernel, predictive mean, and variance, run
```bash
python kernel_and_predictive_plots.py
``` 

### Computing Kernel and Predictive of Neural Network

#### 1. If you have* a trained neural network with weight-decay but no posterior approximation yet
Use the function `dnn2gp.compute_laplace(model, train_loader, prior_prec):` to compute a
Laplace approximation to the posterior distribution. Simply pass the pytorch model, iterator of
samples and targets, and the prior precision (equal to weight-decay parameter). We assume a 
classification task so we will use the neural network logits (pre-softmax) outputs and
assume you trained with softmax.

#### 2. Compute dnn2gp quantities (Jacobians and predictives)

To obtain Jacobians, GP predictive mean, labels, predictive variance, predictive noise, 
and predictive mean we can call `compute_dnn2gp_quantities(model, data_loader, post_prec=post_prec, limit=1000)`
with pytorch model that outputs logits and is trained with softmax and the corresponding training
data set loader. The posterior precision approximation needs to be passed, too. Otherwise,
only the Jacobians, GP predictive mean, and labels are returned.

Note that in correspondence to our paper, we use the reparameterization for these quantities.
Only the GP predictive mean corresponds to the dual model in the Theorems while the 
predictive mean and predictive variances are obtained *after* reparameterizing for 
the original data space.

#### 3. Compute the kernel and aggregate it

To compute the kernel, the Jacobians from 2. are required. Based on the Jacobians,
we can compute the kernel or some aggregation of it with `compute_kernel(Jacobians, agg_type='diag')`.
For most networks and data set sizes, using `limit=1000` in will be easily computable and does not
take much storage. The aggregation type `agg_type` denotes how we reduce the `NK x NK` kernel.
`'diag'` simply sums up across the diagonals, `'sum'` sums all elements of a `K x K`.


### Requirements

python 3.x with `requirements.txt`.
