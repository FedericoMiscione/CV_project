# CV_project

## Useful links

Link to the dataset:
- CVUSA dataset (including Segformer segmentations) : https://drive.google.com/drive/folders/1niw0ksJ6dmfh-tuNEJT6goMMt6UdVutQ?usp=drive_link
- Synthetic:  
   train : https://drive.google.com/drive/folders/1UmWqBHypT-DTawTsHfGVkaFhXbUE0kax?usp=drive_link  
   validation : https://drive.google.com/drive/folders/118I4j4FuPAESZrsqnrGibK-s6vReQAWU?usp=drive_link  
   test : https://drive.google.com/drive/folders/18wTdbQSXv9G7t4VkiJzRgJSQnADmTu9a?usp=drive_link
- Synthetic segmentation : https://drive.google.com/drive/folders/1WAP0ewobiz-Oyvmu2U-ZWOgO9fjS9Nrh?usp=drive_link

Link to the checkpoints:
https://drive.google.com/drive/folders/19icjbUajA_qN_7tSlNzgruTBwbQi3Lbi?usp=drive_link

## Ground to Aerial reconstruction and segmentation

The code uses a subset of the CVUSA dataset, with a streetview image provided as input of a transformer-based model that reconstruct an aerial view of the input. The model is based on the SwinTransformer model pretrained as encoder and a deconvolutional net as decoder. The image provided as input is an RGB image concatenated with the related Depth Map and Segmentation (RGBDS image in this report). The training has been conducted using the related RGBDS from the satellite view.  
The generated image has been segmented using a UNET-resnet18 model pretrained on the ImageNet dataset and fine-tuned on the produced segmentation from the original satellite images of the dataset.

In order to obtain better results, the segmentations of streetview and satellite images have been produced using a pretrained SegFormer model, focusing on the following classes, with the corresponding value for the pretrained model: 

- Group 1:  
          1: building  
          25: house  
          48: skyscraper
- Group 2:  
          2: sky  
- Group 3:  
          4: tree  
          17: plant  
- Group 4:  
          6: road  
          11: sidewalk  
          52: path  
- Group 5:  
          9: grass  
          13: earth  
          29: field  
          94: land  
- Group 6:  
          21: water  
          26: sea  
          60: river  
          128: lake  
- Group 7:  
          46: sand

## Localization task ground-aerial:

The second task consists of geolocalizing the correct satellite image through features extracted from streetview, synthetic, segmentation of the synthetic and the aerial view. 
The task has been performed through a model composed by two subnetworks:
- **JointFeatureLearningNet**, in which two parallel VGG16 encoders, one for the ground image and one shared encoder for aerial, segmentation, synthetic views and aerial. Then the features are concatenated and passed through a 1x1 convolutional fusion layer. 
- **FeatureFusionNet**, in which performs global average pooling and reduces feature dimension to 256 via a linear layer.

**GeoLocalizationNet** is the model that combines the above modules.

The model has been trained using a *WeightedSoftMarginTripletLoss* that uses two terms: triplet loss between anchor–positive and anchor–negative, auxiliary loss between synthetic and positive embeddings
This encourages the model to both match true geo-locations and learn modality-invariant features.

In order to evaluate the final model, the following metrics have been used:  
Recall@K: Checks if the correct match is among the top-K nearest neighbors in embedding space.  
Top-K image matches are visualized during the validation.
