# OpenADMET PXR Challenge - Model Report
# Overview
The Pregnane-X Receptor (PXR) is a nuclear receptor that regulates the expression of drug metabolizing enzymes. Activation of PXR by small molecule ligands can increase enzyme levels, potentially leading to adverse drug-drug interactions and toxicity, making it an important antitarget in drug discovery and a major determinant of the adsorption, distribution, metabolism, excretion, and toxicity (ADMET) profile of new drug candidates. The OpenADMET organization has opened a challenge focused on the prediction of PXR activation by small molecules. The challenge participants are tasked with training quantitative structure-activity relationship (QSAR) models to predict the pEC<sub>50</sub> values of a blind test set containing 513 PXR ligands. The preliminary results place me among the top 50 participants, with models achieving coefficient of determination (R<sup>2</sup>) values between 0.5 and 0.6.
# Data preparation
The training set was composed of three datasets provided by OpenADMET: the original training dataset (n=4139), the crudes dataset (n=347) and the semipure dataset (n=90). The original training set was the first to be released with the beginning of the challenge. The other two datasets were released in a second time. The test set was composed by 513 compounds. RDKit was used to standardised the compounds' SMILES of all datasets as follows. Salts were stripped of their smaller fragment, SMILES were neutralised and the canonical SMILES and InChIKey were generated. Duplicates were aggregated by calculating the mean of their pEC<sub>50</sub> values. There was only one duplicated compound in the original training set. Two compounds were removed because they were outside the (min 4, max 75) number of heavy atoms limits and one more was removed because it contained a non standard element. The training datasets were concatenated resulting in a total of 5085 compounds. The test set compounds were process in the same way and none of them was removed with this data preparation workflow. Finally, the training data was split into 5 parts for cross-validation using the Tanimoto similarity splitter provided by the deepchem package.
# Molecular descriptors
I represented the compounds with a combination of 2D and 3D features:
1. Morgan fingerprints with radius 2 and 2048 bits to represent the local chemical environment.
2. VolSurf descriptors (n=122) to represent the global interaction and physchem environment.
I aggregated these two descriptor groups to for the X matrix which I used for training the model.
# Model definition
The model was a CatBoost regressor whose parameters I tuned to fit the training of a X matrix composed by both catecorical and continuous features and avoid overfitting: learning_rate=0.03, depth=6, l2_leaf_reg=3, random_strength=1, bagging_temperature=1, rsm=0.5. 
# Training & evaluation
I performed cross-validation with early stopping (early_stopping_rounds=50) and 3000 iterations (number of trees) to block the trainig at the most optimal point. When aggregating the 5 folds to calculate the performance scores, I also averaged the number of iterations for the final model training. The following table reports the cross-validation scores mean absolute error (MAE) and R<sup>2</sup>.

| MAE (mean) | MAE (std) | R<sup>2</sup> (mean) | R<sup>2</sup> (std) |
|----------|----------|----------|----------|
| 0.5342 | 0.0261 | 0.5563 | 0.0661 |


# Results analysis

# Conclusion

# Acknowledgments
- OpenADMET for organising the challenge
- [Molecular Discovery Ltd](https://www.moldiscovery.com/) for providing the programs MoKa and VolSurf.
