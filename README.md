# OpenADMET PXR Challenge - Model Report
# Overview
The Pregnane-X Receptor (PXR) is a nuclear receptor that regulates the expression of drug metabolizing enzymes. Activation of PXR by small molecule ligands can increase enzyme levels, potentially leading to adverse drug-drug interactions and toxicity, making it an important antitarget in drug discovery and a major determinant of the adsorption, distribution, metabolism, excretion, and toxicity (ADMET) profile of new drug candidates. The OpenADMET organization has opened a challenge focused on the prediction of PXR activation by small molecules. The challenge participants are tasked with training quantitative structure-activity relationship (QSAR) models to predict the pEC<sub>50</sub> values of a blind test set containing 513 PXR ligands. The preliminary results place me among the top 50 participants, with models achieving coefficient of determination (R<sup>2</sup>) values between 0.5 and 0.6.
# Data preparation
The training set was composed of three datasets provided by OpenADMET: the original training dataset (n=4139), the crudes dataset (n=347) and the semipure dataset (n=90). The original training set was the first to be released with the beginning of the challenge. The other two datasets were released in a second time. The test set was composed by 513 compounds of which, 253 had unblinded pEC<sub>50</sub> values and 260 had blinded pEC<sub>50</sub> values. RDKit was used to standardised the compounds' SMILES of all datasets as follows. Salts were stripped of their smaller fragment, SMILES were neutralised and the canonical SMILES and InChIKey were generated. Duplicates were aggregated by calculating the mean of their pEC<sub>50</sub> values. There was only one duplicated compound in the original training set. Two compounds were removed because they were outside the (min 4, max 75) number of heavy atoms limits and one more was removed because it contained a non standard element. The training datasets were concatenated resulting in a total of 4570 compounds. The test set compounds were process in the same way and none of them was removed with this data preparation workflow. Finally, the training data was split into 5 parts for cross-validation using the Tanimoto similarity splitter provided by the deepchem package.
# Molecular descriptors
I represented the compounds with a combination of 2D and 3D features:
1. Morgan fingerprints with radius 2 and 2048 bits to represent the local chemical environment.
2. VolSurf descriptors (n=122) to represent the global interaction and physchem environment.
I scaled the continuous VolSurf descriptors in the training set (by removing the mean and scaling to unit variance) and applied the scaling to the test set compounds. Afterwards, I aggregated these two descriptor groups to form the X matrix which I used for training the model.
# Model definition
The model is a consensus of two different regression models trained using the same descriptors but different algorithms:
- A CatBoost gradient boosting regressor configured for predicting continuous molecular properties. It builds an ensemble of up to 3,000 decision trees, sequentially fitting each tree to correct the errors of the previous ones. A relatively low learning rate (0.03) and moderate tree depth (6) allow the model to capture complex, non-linear relationships while reducing the risk of overfitting. Additional regularization parameters, including L2 leaf regularization, feature subsampling (rsm=0.5), and stochastic sampling (bagging_temperature=1), further improve generalization. Training is made more efficient through early stopping, which halts the boosting process if performance on the validation set does not improve for 50 consecutive iterations.
- A multimodal feed-forward neural network implemented with PyTorch. Separate neural network branches first learn latent representations from each input, reducing the Morgan fingerprints to a 256-dimensional embedding and the VolSurf descriptors to a 64-dimensional embedding. These embeddings are then concatenated and passed through a shared prediction head to produce a single continuous output for regression tasks. The architecture incorporates ReLU activations, batch normalization, and dropout layers to improve training stability, reduce overfitting, and enhance generalization.

The consensus predictins were genrated with the following formula: Y<sub>Consensus</sub> = Y<sub>CatBoost</sub> * 0.5 + Y<sub>NeurlaNetwork</sub> * 0.5
# Training & evaluation
For each model, I performed cross-validation with early stopping (early_stopping_rounds=50) and 3000 iterations (number of trees) to block the trainig at the most optimal point. When aggregating the 5 folds to calculate the performance scores, I also averaged the number of iterations for the final model training. The following table reports the cross-validation scores mean absolute error (MAE) and R<sup>2</sup>.

| Model | MAE (mean) | MAE (std) | R<sup>2</sup> (mean) | R<sup>2</sup> (std) |
|----------|----------|----------|----------|----------|
| CatBoost | 0.5342 | 0.0261 | 0.5563 | 0.0661 |
| Neural netwrork | 0.5564 | 0.02 | n.d. | n.d. |

I trained an intermediate CatBoost with the full training set (n=4570) and by setting iterations=1171 (the mean iterations across the 5 cross-validation folds). I used this model to predict the 253 unblinded test set compounds. The following table reports the prediction scores and the relative absolute error (RAE) which is used for ranking the challenge participants.

| Model | MAE | RAE | R<sup>2</sup> |
|----------|----------|----------|----------|
| CatBoost | 0.5064 | 0.6341 | 0.4905 |
| Neural netwrork | 0.5195 | 0.6505 | 0.4975 |
| Consensus | 0.4853 | 0.6077 | 0.5170 |

Finally, I concatenated the training set with the unblinded test set (for a total of 4823 compounds), I applied the same scaling to the VolSurf descriptors of the training and the blind test sets, trained a final model and predicted the blind test compounds. These predictions and the ones generated for the unblinded test set using the intermediate model were submitted for the final ranking.

# Results analysis
The scores relative to the prediction of the 253 unblinded compounds are indicative of a model with decent predictive power. The experimental vs predicted scatter plot in Figure 1 shows that the model struggles more in predicting the high and low pEC<sub>50</sub> values at the extremes of the activity distribution.
<figure>
  <img src="images/figure1" alt="Description" width="400">
  <figcaption>This is the visible caption text</figcaption>
</figure>
# Conclusion

# Acknowledgments
- OpenADMET for organising the challenge.
- [Molecular Discovery Ltd](https://www.moldiscovery.com/) for providing the programs MoKa and VolSurf.
