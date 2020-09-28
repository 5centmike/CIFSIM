# CIFSIM
Compartmentalization by Independent Forces to Simulate Interaction Maps (CIFSIM)

To enable comparison across cell types and experiments we first binned all epigenetic signals into quantiles at a 100 kb resolution. For each normalized signal, we then learned, using a Maximum Likelihood Estimation approach, an attraction-repulsion relationship for each pair of quantiles. This attraction-repulsion mapping effectively represents the average enrichment or depletion between all bins with the corresponding level of signal. The model then predicts the number of reads at each bin by summing the attraction-repulsion scores for each signal and multiplying by a constant distance factor to account for the power law decay of genomic interactions. 

We model the compartmentalization of the genome as the independent contributions of individual 1D epigenetic signals and use a machine learning method Maximum Likelihood Estimation (MLE) to learn the relationship between the signals and interaction frequencies in the Hi-C map. In this way we hope to quantify the nuclear forces driving compartmentalization. Our model treats each epigenetic signal as an independent but additive effect on Hi-C interaction frequency according to the equation:

![alt text](https://github.com/5centmike/CIFSIM/Esignals.png?raw=true)

Where the expected interaction frequency E between any two genomic loci i and j is the sum of each signal’s weight effect. Using MLE we find the optimal values of each signal’s weights such that the expected result E is as close to the observed value as possible. This produces a trained model that can reproduce the 3D organization of the genome from 1D epigenetic signals by quantifying the expected contributions of each correlated compartmentalizing force.

We use the Maximum Likelihood Estimation approach to optimize the values of a vector 𝛃 where each entry in 𝛃 corresponds to an entry in the attraction-repulsion maps such that for all possible pairs of quantiles for each of the three chromatin signals H3K27ac, H3K27me3, and H3K9me3, there is a corresponding weight in 𝛃. We then construct a sparse matrix X where each row corresponds to an interaction bin in the flattened Hi-C matrix and each column a weight in 𝛃. Each row in X is zeros except in the 3 columns corresponding to the entries in 𝛃 that describe the pair of genomic bins’ signal quantiles. If we take an observed Hi-C map which has been normalized for distance by dividing by the expected value at each distance, which we term y, the model’s approximation of the normalized observed map is then:



Treating the normalized interaction frequency as a normal distribution we derive a likelihood function the log of which we will maximize to optimize the weights of 𝛃 in a Maximum Likelihood Estimation approach:



Where N is the total number of unique interacting bins in the linearized Hi-C matrix excluding each genomic bin’s interactions with itself. To optimize the parameters of 𝛃 we iteratively solve starting with initial values of 0 for all weights in 𝛃. We use the Newton-Raphson method to update the weights by gradient descent. With each iteration we account for one final feature of the Hi-C matrix. Due to the colocalization of compartmentalizing features along the chromosome the frequency of intra-compartmental interactions is enriched at short ranges. Two genomic bins within a megabase are more likely to be enriched for compartmental interactions than two bins many megabases apart. This bias leads to aberrant distance normalization, and so the distance normalization is updated after each iteration to account for the average compartmental enrichment the model predicts at each distance. The distance normalization is divided by the average enrichment so that it more accurately reflects the true effect of distance on interaction frequency.

