# CCA-images-text
## This repo was cloned from https://github.com/marioyc/CCA-images-text 
The following marked section is a general explanation of the scripts that @marioyc coded and implemented including a reference to the publication which first implemented this framework.

---

Canonical Correlation Analysis for joint representations of images and text based on [1].

Finds a common representation space for images and tags. Uses the [MS COCO dataset](http://mscoco.org/dataset/#overview), in particularly the captions given for the training data.

* main/preprocess.py : computes tags for the training images and computes the features using the [VGG16](http://www.robots.ox.ac.uk/~vgg/research/very_deep/) network
* main/pca_cca.py : computes a PCA on the training data and then performs a CCA to find the projection matrices
* main/image_to_tags.py : finds the corresponding tags for the images on the validation data
* main/tag_to_image.py : finds the corresponding images taking as tags the categories of COCO

[1] Yunchao Gong, Qifa Ke, Michael Isard, Svetlana Lazebnik. A Multi-View Embedding Space for Modeling Internet Images, Tags, and their Semantics. International Journal of Computer Vision, Volume 106 Issue 2, January 2014, Pages 210-233.

---

## More detail regarding the code framework author Kevin Ryan github handle @kevjp
Main ammendments to code introduced in this repository are highlighted in bold

* main/preprocess.py - contains function count_words which takes the captions associated with each image from the MS COCO dataset (5 captions per image), removes stopwords and tokenizes all words for downstream analysis. It then generates a dictionary for each image and provdes a frequency count for each word over the 5 captions
Also contains the function calc_features which loads a word embedding framework from text8 (relates to a Wikipedia corpus captured in 2006). It also loads a pre-trained deep learning image classification model. **Added in argument use_model to enable code to have the utility to swap in an external model as opposed to the default image feature extractor which is a pre-trained VGG16 model. This allowed me to swap in the Room type/object multi-label classifier to explore its efficicacy in deefining meaningful feature vectors that separate well in the joint image-text embedding space.**
calc_features then sorts the words associated with each image into frequency order ignoring any that are not in thee text8 corpus. It extracts the image feature and also obtains the word vector representation of the top n (default is 2) most frequent words associated with the image. Can change this setting by providing the argument --tagsPerImage . The image features and the word vectors are stored in separate numpy matrices and both matrices are then stored in a single compressed file called trrain\\-features.npz using numpy_savez_compressed.
* main/pca_cca.py - Loads the matrices from the compressed numpy file geneerated by main/preprocess.py. Peforms PCA on the image features matrix to reduce its dimensionality (output of VGG network is 4096 length vector) to a 500 length vector. Then performs Canonical Correlation Analysis using the PCA generated image vectors and the associated word vectors for each image. The CCA derived bases that define the transformation required to transform the image feature vectors and the word veectors to the image-text embedding space are defined and saved along with the PCA model into a compressed numpy file called projections.npz.
* main/image_to_tags.py - contains single function called image_to_tags() which loads the bases veectors generated by main/pca_cca.py. Then it calls the function calc_testing_image_features() from feeatures.py and loads the imge paths, PCA model and the image basis vector into the function. calc_testing_image_features() then extracts features using eeither VGG16 **or the model loaded from the argument use_model that I added into the script. I had to add in a boolean setting as an argument to calc_testing_image_features() defined as VGG16_bool.** It then performs PCA on the resultant veectors and then takes the dot product of each of those PCA-derived features with the image basis vector in order to transform each image feature vector onto the image-text embedding space. The dot product of the word vector representations of all words in the glossary of terms is calculated with th word vector basis veector is calculated. A pairwise distance matrix (euclidean distance) is calculated between all transformed image feature vectors and all transformed word vectors is calculated. For each image the distance values are sorted into order to find the closest words to each image and the top 5 closest words to each image are recorded in a numpy arrary and saved as score_matrix.npz
* aws_install_ubuntu14.sh - bash script for configuring AWS EC2 instance environment. **added in the capacity to add in the Google/ADE20K handlabelled data set used to generate Room Type/object classfier (see https://github.com/kevjp/Rightmove)**
* main/mds_pca.py script takes in score matrix generated by main/image_to_tags.py script and runs function gen_scatter_multi_tag(). This function generates a pairwise distance matrix between the score vectors for each image and then performs MDS to transform the pairwise distance matrix to a 2 dimensional space preserving the distances between each image. The MDS is represented using a 2d scatter plot of the transformed data points. This code is completely new from the original repo by @marioyc

