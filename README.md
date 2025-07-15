# CV_project

## Useful links

Link to the dataset:
- CVUSA dataset (including Segformer segmentations) : https://drive.google.com/drive/folders/1niw0ksJ6dmfh-tuNEJT6goMMt6UdVutQ?usp=drive_link
- Synthetic:  train : https://drive.google.com/drive/folders/1UmWqBHypT-DTawTsHfGVkaFhXbUE0kax?usp=drive_link
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

### Geo-Localization using Cross-View and Feature Fusion with Triplet Learning:

This repository implements a deep learning pipeline for cross-view geo-localization, using a joint feature learning and feature fusion network, trained with a customized triplet loss function. The task is to match a ground-level image (e.g., from street view) with its corresponding aerial (satellite) image, incorporating segmentation and synthetic views as additional modalities.

**Dataset**:
The code uses the CVUSA dataset (or a subset), which contains:
●	Ground-level panorama images (ground)
●	Synthetic renderings (synthetic)
●	Segmented images (segmentation)
●	Aerial views (bingmap)

CSV files list the file paths for training, validation, and test samples.

**Training Pipeline**:
1. Dataset Preparation
The CVUSATripletDataset class constructs triplets:
●	Anchor: Ground-level image
●	Positive: Corresponding aerial view
●	Negative: Random (wrong) aerial view

It also includes synthetic and segmentation views for auxiliary supervision.
Images are resized and normalized using ImageNet statistics.

2. Model Architecture
●	a. JointFeatureLearningNet
Two parallel VGG16 encoders:
One for the ground image
One shared encoder for aerial, segmentation, and synthetic views
Features are concatenated and passed through a 1x1 convolutional fusion layer.

●	b. FeatureFusionNet
Performs global average pooling and reduces feature dimension to 256 via a linear layer.

●	c. GeoLocalizationNet
Combines the above modules.

Outputs embeddings for query (ground) and reference (aerial) images.

3. Loss Function
WeightedSoftMarginTripletLoss uses two terms:
●	Triplet loss between anchor–positive and anchor–negative
●	Auxiliary loss between synthetic and positive embeddings

This encourages the model to both match true geo-locations and learn modality-invariant features.

4. Training Loop
Standard training with Adam optimizer
Loss and recall metrics printed every epoch
Supports checkpoint loading/saving
Uses GPU if available

5. Evaluation Metrics
Recall@K: Checks if the correct match is among the top-K nearest neighbors in embedding space.
Top-K image matches are visualized.

Key Functions:
recall_at_k: Computes recall based on embedding distances.
show_top_k_images: Visualizes the retrieved top-K images per query.
count_parameters: Displays model size.

Example Output:
Recall@1, Recall@5, Recall@10 printed each epoch.
Side-by-side image retrieval visualization.
