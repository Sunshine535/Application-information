# ACL2024
## STSPL-SSC: Semi-Supervised Few-Shot Short Text Clustering with Semantic text similarity Optimized Pseudo-Labels

### Abstract

This study introduces the Semantic Textual Similarity Pseudo-Label Semi-Supervised Clustering (STSPL-SSC) framework. The STSPL-SSC framework is designed to tackle the prevalent issue of scarce labeled data by combining a Semantic Textual Similarity Pseudo-Label Generation process with a Robust Contrastive Learning module. The process begins with employing k-means clustering on embeddings for initial pseudo-Label allocation. Then we use a Semantic Text Similarity-enhanced module to supervise the secondary clustering of pseudo-labels using labeled data to better align with the real clustering centers. Subsequently, an Adaptive Optimal Transport (AOT) approach fine-tunes the pseudo-labels. Finally, a Robust Contrastive Learning module is employed to foster the learning of classification and instance-level distinctions, aiding clusters to better separate. Experiments conducted on multiple real-world datasets demonstrate that with just one label per class, clustering performance can be significantly improved, outperforming state-of-the-art models with an increase of 1-6% in both accuracy and normalized mutual information, approaching the results of fully-labeled classification.

Keywords: Semi-supervised learning，Semantic Textual Similarity，Short text clustering
