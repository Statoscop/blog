Title: Transfer Learning avec Keras : entraînement et évaluation d'un modèle de détection d'images
Author: Antoine
Date: '2024-02-14'
Slug: transfer-learning-keras
Category: Python, Deep learning
Tags: Python, Keras, CNN, Machine Learning, Data Science, Transfer Learning
Cover: images/cover_17.png
Summary: Création d'un modèle de détection d'images permettant de classifier des fruits et leur état de maturité.

[TOC]


L'une des petites révolutions que le Deep Learning a permis est le fait de pouvoir entraîner des modèles sur des __données très volumineuses__. En particulier, les modèles de détection d'images, les _Convolutional Neural Networks (CNN)_ ont pu se développer.  
Ces modèles s'appuient sur des couches profondes spécifiques : __les réseaux convolutifs__. Ceux-ci permettent de pré-traiter les images afin de repérer des structures spécifiques dans les images et de pouvoir ainsi correctement les classifier, en évitant le [sur-apprentissage](https://blog.statoscop.fr/le-dilemme-biais-variance-dans-la-modelisation-de-donnees.html#liens-avec-loverfitting-et-lunderfitting-dans-les-modeles-de-machine-learning). Ces couches de convolution sont suivis de réseaux de neurones classiques permettant d'obtenir la classification voulue.   
Reste que ces modèles doivent être entraînés sur __des quantités importantes de données__ avant d'être performants, et que cet entraînement peut être extrêmement __long et coûteux en ressources__, rendant quasi-impossible le fait de le réaliser sur son ordinateur en local. C'est là qu'intervient le __transfer learning__. Le principe est de partir d'un modèle pré-entraîné et de se contenter de le ré-entraîner ou d'en entraîner une partie sur ses données spécifiques. En particulier, __la partie convolutive du modèle n'est pas forcément spécifique à un thème donné__ et n'a souvent pas besoin d'être ré-entraîné. C'est comme si on se contentait de __spécialiser un modèle généraliste__. Cette méthode va nous permettre de gagner un temps précieux en entraînement et de __bénéficier de performances de modèles entraînés sur un grand nombre de données__.  
Voyons comment la mettre en oeuvre avec la __librairie Python `keras`__.


# Préparation des données  
Pour notre exemple, on va partir ici d'un [dataset Kaggle](https://www.kaggle.com/datasets/alihasnainch/fruits-dataset-for-classification/code) avec des images de différents fruits, pourris ou en bon état. Les fruits pris en compte sont les pêches, les fraises et les grenades. Pour chaque fruit, on sépare ceux qui sont pourris de ceux qui ne le sont pas. On a donc 6 classes en tout. Commençons par afficher certaines de ces images :  


```python
%matplotlib inline
import matplotlib.pyplot as plt
import numpy as np
import os
from PIL import Image

labels = []
images = []
for i in os.listdir("dataset"):
    images.append("dataset/"+i+"/"+os.listdir("dataset/"+i)[0])
    labels.append(i)

plt.figure()
f, axes = plt.subplots(2, 3)
for ax in range(len(axes.ravel())):
    subplot = axes.ravel()[ax]
    subplot.imshow(np.asarray(Image.open(images[ax])))
    subplot.xaxis.set_tick_params(labelbottom=False)
    subplot.yaxis.set_tick_params(labelleft=False)
    subplot.set_title(labels[ax])
plt.show()
```
    
![Pelican](../images/keras_tl/output_3_1.png)
    

Avec `keras`, **le dataset est créé directement à partir d'un dossier** où chaque sous-dossier comporte une classe, qui sera labellisée par le nom de celui-ci. De plus, la **partition entre données d'entraînement et de validation** peut être faite directement à ce moment là. On utilise pour cela la fonction `image_dataset_from_directory`, avec les paramètres correspondant à notre problème.  
On a aussi **créé en amont un dossier contenant un sous-ensemble de ces images** qui nous servira de **dataset de test** au moment de l'évaluation de notre modèle.


```python
IMG_SIZE = 256

from keras.utils import image_dataset_from_directory  
images_dataset = image_dataset_from_directory('dataset', # nom du dossier où se trouvent mes deux dossiers d'images
                                              batch_size = 64, # Nombre d'images chargées simultanément en mémoire lors de l'apprentissage
                                              image_size = (IMG_SIZE, IMG_SIZE), # Choix du taille de l'image
                                              validation_split = 0.30, # 30% des données en set de validation
                                              seed = 15, # reproductibilité du split aléatoire
                                              label_mode = "categorical", # problème de classif
                                              subset = "both") # on crée les deux sets d'entraînement et de validation
training_set = images_dataset[0]
validation_set = images_dataset[1]

# Set test 
test_set = image_dataset_from_directory('Set test', # nom du dossier où se trouvent mes deux dossiers d'images
                                        image_size = (IMG_SIZE, IMG_SIZE), # Choix de la taille de l'image
                                        label_mode = "categorical") # problème de classif
``` 

    Found 1325 files belonging to 6 classes.
    Using 928 files for training.
    Using 397 files for validation.
    Found 330 files belonging to 6 classes.


Nos images ont bien été chargées, nous pouvons vérifier que les labels ont été correctement attribuées : 


```python
training_set.class_names
```




    ['fraise fraîche',
     'fraise pourrie',
     'grenade fraîche',
     'grenade pourrie',
     'pêche fraîche',
     'pêche pourrie']



À noter qu'il est importer de correctement paramétrer la __taille du batch__ et les __dimensions de l'image__. Plus la taille du batch est importante, plus le modèle tournera rapidement, mais il pourrait être moins précis et surtout votre machine pourrait crasher par manque de RAM et/ou GPU. Pour la taille de l'image, de la même manière elle aura **un impact sur les performances du modèle** et sur la capacité de votre machine à encaisser l'entraînement.

# Entraînement du modèle  
Pour entraîner notre modèle, on commence par charger un des modèles de détection d'images disponibles dans `keras` puis on définit l'ensemble de notre réseau de neurones.  

## Chargement d'un modèle pré-entraîné  
De nombreux **modèles CNN pré-entraînés** peuvent être chargés depuis `keras`. Ils sont répertoriés sur [cette page](https://keras.io/api/applications/) de la documentation. Nous décidons pour notre exemple d'utiliser la __dernière version du modèle ResNet50__ :  


```python
# Chargement du modèle Xception
from keras.applications import ResNet50V2
classifier_resnet = ResNet50V2(weights='imagenet', include_top=False, input_shape=(IMG_SIZE, IMG_SIZE, 3))
```

Notez que l'on paramétrise `include_top=False` de manière à pouvoir choisir le format de nos images en entrée, ici en 256*256.  
Le modèle chargé a la structure suivante :  


```python
classifier_resnet.summary()  
```

```python


    Model: "resnet50v2"
    __________________________________________________________________________________________________
     Layer (type)                Output Shape                 Param #   Connected to                  
    ==================================================================================================
     input_1 (InputLayer)        [(None, 256, 256, 3)]        0         []                            
                                                                                                      
     conv1_pad (ZeroPadding2D)   (None, 262, 262, 3)          0         ['input_1[0][0]']             
                                                                                                      
     conv1_conv (Conv2D)         (None, 128, 128, 64)         9472      ['conv1_pad[0][0]']           
                                                                                                      
     pool1_pad (ZeroPadding2D)   (None, 130, 130, 64)         0         ['conv1_conv[0][0]']          
                                                                                                      
     pool1_pool (MaxPooling2D)   (None, 64, 64, 64)           0         ['pool1_pad[0][0]']           
                                                                                                      
     conv2_block1_preact_bn (Ba  (None, 64, 64, 64)           256       ['pool1_pool[0][0]']          
     tchNormalization)                                                                                
                                                                                                      
     conv2_block1_preact_relu (  (None, 64, 64, 64)           0         ['conv2_block1_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv2_block1_1_conv (Conv2  (None, 64, 64, 64)           4096      ['conv2_block1_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv2_block1_1_bn (BatchNo  (None, 64, 64, 64)           256       ['conv2_block1_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv2_block1_1_relu (Activ  (None, 64, 64, 64)           0         ['conv2_block1_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv2_block1_2_pad (ZeroPa  (None, 66, 66, 64)           0         ['conv2_block1_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv2_block1_2_conv (Conv2  (None, 64, 64, 64)           36864     ['conv2_block1_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv2_block1_2_bn (BatchNo  (None, 64, 64, 64)           256       ['conv2_block1_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv2_block1_2_relu (Activ  (None, 64, 64, 64)           0         ['conv2_block1_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv2_block1_0_conv (Conv2  (None, 64, 64, 256)          16640     ['conv2_block1_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv2_block1_3_conv (Conv2  (None, 64, 64, 256)          16640     ['conv2_block1_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv2_block1_out (Add)      (None, 64, 64, 256)          0         ['conv2_block1_0_conv[0][0]', 
                                                                         'conv2_block1_3_conv[0][0]'] 
                                                                                                      
     conv2_block2_preact_bn (Ba  (None, 64, 64, 256)          1024      ['conv2_block1_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv2_block2_preact_relu (  (None, 64, 64, 256)          0         ['conv2_block2_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv2_block2_1_conv (Conv2  (None, 64, 64, 64)           16384     ['conv2_block2_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv2_block2_1_bn (BatchNo  (None, 64, 64, 64)           256       ['conv2_block2_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv2_block2_1_relu (Activ  (None, 64, 64, 64)           0         ['conv2_block2_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv2_block2_2_pad (ZeroPa  (None, 66, 66, 64)           0         ['conv2_block2_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv2_block2_2_conv (Conv2  (None, 64, 64, 64)           36864     ['conv2_block2_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv2_block2_2_bn (BatchNo  (None, 64, 64, 64)           256       ['conv2_block2_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv2_block2_2_relu (Activ  (None, 64, 64, 64)           0         ['conv2_block2_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv2_block2_3_conv (Conv2  (None, 64, 64, 256)          16640     ['conv2_block2_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv2_block2_out (Add)      (None, 64, 64, 256)          0         ['conv2_block1_out[0][0]',    
                                                                         'conv2_block2_3_conv[0][0]'] 
                                                                                                      
     conv2_block3_preact_bn (Ba  (None, 64, 64, 256)          1024      ['conv2_block2_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv2_block3_preact_relu (  (None, 64, 64, 256)          0         ['conv2_block3_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv2_block3_1_conv (Conv2  (None, 64, 64, 64)           16384     ['conv2_block3_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv2_block3_1_bn (BatchNo  (None, 64, 64, 64)           256       ['conv2_block3_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv2_block3_1_relu (Activ  (None, 64, 64, 64)           0         ['conv2_block3_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv2_block3_2_pad (ZeroPa  (None, 66, 66, 64)           0         ['conv2_block3_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv2_block3_2_conv (Conv2  (None, 32, 32, 64)           36864     ['conv2_block3_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv2_block3_2_bn (BatchNo  (None, 32, 32, 64)           256       ['conv2_block3_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv2_block3_2_relu (Activ  (None, 32, 32, 64)           0         ['conv2_block3_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     max_pooling2d (MaxPooling2  (None, 32, 32, 256)          0         ['conv2_block2_out[0][0]']    
     D)                                                                                               
                                                                                                      
     conv2_block3_3_conv (Conv2  (None, 32, 32, 256)          16640     ['conv2_block3_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv2_block3_out (Add)      (None, 32, 32, 256)          0         ['max_pooling2d[0][0]',       
                                                                         'conv2_block3_3_conv[0][0]'] 
                                                                                                      
     conv3_block1_preact_bn (Ba  (None, 32, 32, 256)          1024      ['conv2_block3_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv3_block1_preact_relu (  (None, 32, 32, 256)          0         ['conv3_block1_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv3_block1_1_conv (Conv2  (None, 32, 32, 128)          32768     ['conv3_block1_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv3_block1_1_bn (BatchNo  (None, 32, 32, 128)          512       ['conv3_block1_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv3_block1_1_relu (Activ  (None, 32, 32, 128)          0         ['conv3_block1_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv3_block1_2_pad (ZeroPa  (None, 34, 34, 128)          0         ['conv3_block1_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv3_block1_2_conv (Conv2  (None, 32, 32, 128)          147456    ['conv3_block1_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv3_block1_2_bn (BatchNo  (None, 32, 32, 128)          512       ['conv3_block1_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv3_block1_2_relu (Activ  (None, 32, 32, 128)          0         ['conv3_block1_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv3_block1_0_conv (Conv2  (None, 32, 32, 512)          131584    ['conv3_block1_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv3_block1_3_conv (Conv2  (None, 32, 32, 512)          66048     ['conv3_block1_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv3_block1_out (Add)      (None, 32, 32, 512)          0         ['conv3_block1_0_conv[0][0]', 
                                                                         'conv3_block1_3_conv[0][0]'] 
                                                                                                      
     conv3_block2_preact_bn (Ba  (None, 32, 32, 512)          2048      ['conv3_block1_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv3_block2_preact_relu (  (None, 32, 32, 512)          0         ['conv3_block2_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv3_block2_1_conv (Conv2  (None, 32, 32, 128)          65536     ['conv3_block2_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv3_block2_1_bn (BatchNo  (None, 32, 32, 128)          512       ['conv3_block2_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv3_block2_1_relu (Activ  (None, 32, 32, 128)          0         ['conv3_block2_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv3_block2_2_pad (ZeroPa  (None, 34, 34, 128)          0         ['conv3_block2_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv3_block2_2_conv (Conv2  (None, 32, 32, 128)          147456    ['conv3_block2_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv3_block2_2_bn (BatchNo  (None, 32, 32, 128)          512       ['conv3_block2_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv3_block2_2_relu (Activ  (None, 32, 32, 128)          0         ['conv3_block2_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv3_block2_3_conv (Conv2  (None, 32, 32, 512)          66048     ['conv3_block2_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv3_block2_out (Add)      (None, 32, 32, 512)          0         ['conv3_block1_out[0][0]',    
                                                                         'conv3_block2_3_conv[0][0]'] 
                                                                                                      
     conv3_block3_preact_bn (Ba  (None, 32, 32, 512)          2048      ['conv3_block2_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv3_block3_preact_relu (  (None, 32, 32, 512)          0         ['conv3_block3_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv3_block3_1_conv (Conv2  (None, 32, 32, 128)          65536     ['conv3_block3_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv3_block3_1_bn (BatchNo  (None, 32, 32, 128)          512       ['conv3_block3_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv3_block3_1_relu (Activ  (None, 32, 32, 128)          0         ['conv3_block3_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv3_block3_2_pad (ZeroPa  (None, 34, 34, 128)          0         ['conv3_block3_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv3_block3_2_conv (Conv2  (None, 32, 32, 128)          147456    ['conv3_block3_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv3_block3_2_bn (BatchNo  (None, 32, 32, 128)          512       ['conv3_block3_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv3_block3_2_relu (Activ  (None, 32, 32, 128)          0         ['conv3_block3_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv3_block3_3_conv (Conv2  (None, 32, 32, 512)          66048     ['conv3_block3_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv3_block3_out (Add)      (None, 32, 32, 512)          0         ['conv3_block2_out[0][0]',    
                                                                         'conv3_block3_3_conv[0][0]'] 
                                                                                                      
     conv3_block4_preact_bn (Ba  (None, 32, 32, 512)          2048      ['conv3_block3_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv3_block4_preact_relu (  (None, 32, 32, 512)          0         ['conv3_block4_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv3_block4_1_conv (Conv2  (None, 32, 32, 128)          65536     ['conv3_block4_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv3_block4_1_bn (BatchNo  (None, 32, 32, 128)          512       ['conv3_block4_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv3_block4_1_relu (Activ  (None, 32, 32, 128)          0         ['conv3_block4_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv3_block4_2_pad (ZeroPa  (None, 34, 34, 128)          0         ['conv3_block4_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv3_block4_2_conv (Conv2  (None, 16, 16, 128)          147456    ['conv3_block4_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv3_block4_2_bn (BatchNo  (None, 16, 16, 128)          512       ['conv3_block4_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv3_block4_2_relu (Activ  (None, 16, 16, 128)          0         ['conv3_block4_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     max_pooling2d_1 (MaxPoolin  (None, 16, 16, 512)          0         ['conv3_block3_out[0][0]']    
     g2D)                                                                                             
                                                                                                      
     conv3_block4_3_conv (Conv2  (None, 16, 16, 512)          66048     ['conv3_block4_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv3_block4_out (Add)      (None, 16, 16, 512)          0         ['max_pooling2d_1[0][0]',     
                                                                         'conv3_block4_3_conv[0][0]'] 
                                                                                                      
     conv4_block1_preact_bn (Ba  (None, 16, 16, 512)          2048      ['conv3_block4_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv4_block1_preact_relu (  (None, 16, 16, 512)          0         ['conv4_block1_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv4_block1_1_conv (Conv2  (None, 16, 16, 256)          131072    ['conv4_block1_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv4_block1_1_bn (BatchNo  (None, 16, 16, 256)          1024      ['conv4_block1_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block1_1_relu (Activ  (None, 16, 16, 256)          0         ['conv4_block1_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv4_block1_2_pad (ZeroPa  (None, 18, 18, 256)          0         ['conv4_block1_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv4_block1_2_conv (Conv2  (None, 16, 16, 256)          589824    ['conv4_block1_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv4_block1_2_bn (BatchNo  (None, 16, 16, 256)          1024      ['conv4_block1_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block1_2_relu (Activ  (None, 16, 16, 256)          0         ['conv4_block1_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv4_block1_0_conv (Conv2  (None, 16, 16, 1024)         525312    ['conv4_block1_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv4_block1_3_conv (Conv2  (None, 16, 16, 1024)         263168    ['conv4_block1_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv4_block1_out (Add)      (None, 16, 16, 1024)         0         ['conv4_block1_0_conv[0][0]', 
                                                                         'conv4_block1_3_conv[0][0]'] 
                                                                                                      
     conv4_block2_preact_bn (Ba  (None, 16, 16, 1024)         4096      ['conv4_block1_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv4_block2_preact_relu (  (None, 16, 16, 1024)         0         ['conv4_block2_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv4_block2_1_conv (Conv2  (None, 16, 16, 256)          262144    ['conv4_block2_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv4_block2_1_bn (BatchNo  (None, 16, 16, 256)          1024      ['conv4_block2_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block2_1_relu (Activ  (None, 16, 16, 256)          0         ['conv4_block2_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv4_block2_2_pad (ZeroPa  (None, 18, 18, 256)          0         ['conv4_block2_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv4_block2_2_conv (Conv2  (None, 16, 16, 256)          589824    ['conv4_block2_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv4_block2_2_bn (BatchNo  (None, 16, 16, 256)          1024      ['conv4_block2_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block2_2_relu (Activ  (None, 16, 16, 256)          0         ['conv4_block2_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv4_block2_3_conv (Conv2  (None, 16, 16, 1024)         263168    ['conv4_block2_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv4_block2_out (Add)      (None, 16, 16, 1024)         0         ['conv4_block1_out[0][0]',    
                                                                         'conv4_block2_3_conv[0][0]'] 
                                                                                                      
     conv4_block3_preact_bn (Ba  (None, 16, 16, 1024)         4096      ['conv4_block2_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv4_block3_preact_relu (  (None, 16, 16, 1024)         0         ['conv4_block3_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv4_block3_1_conv (Conv2  (None, 16, 16, 256)          262144    ['conv4_block3_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv4_block3_1_bn (BatchNo  (None, 16, 16, 256)          1024      ['conv4_block3_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block3_1_relu (Activ  (None, 16, 16, 256)          0         ['conv4_block3_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv4_block3_2_pad (ZeroPa  (None, 18, 18, 256)          0         ['conv4_block3_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv4_block3_2_conv (Conv2  (None, 16, 16, 256)          589824    ['conv4_block3_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv4_block3_2_bn (BatchNo  (None, 16, 16, 256)          1024      ['conv4_block3_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block3_2_relu (Activ  (None, 16, 16, 256)          0         ['conv4_block3_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv4_block3_3_conv (Conv2  (None, 16, 16, 1024)         263168    ['conv4_block3_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv4_block3_out (Add)      (None, 16, 16, 1024)         0         ['conv4_block2_out[0][0]',    
                                                                         'conv4_block3_3_conv[0][0]'] 
                                                                                                      
     conv4_block4_preact_bn (Ba  (None, 16, 16, 1024)         4096      ['conv4_block3_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv4_block4_preact_relu (  (None, 16, 16, 1024)         0         ['conv4_block4_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv4_block4_1_conv (Conv2  (None, 16, 16, 256)          262144    ['conv4_block4_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv4_block4_1_bn (BatchNo  (None, 16, 16, 256)          1024      ['conv4_block4_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block4_1_relu (Activ  (None, 16, 16, 256)          0         ['conv4_block4_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv4_block4_2_pad (ZeroPa  (None, 18, 18, 256)          0         ['conv4_block4_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv4_block4_2_conv (Conv2  (None, 16, 16, 256)          589824    ['conv4_block4_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv4_block4_2_bn (BatchNo  (None, 16, 16, 256)          1024      ['conv4_block4_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block4_2_relu (Activ  (None, 16, 16, 256)          0         ['conv4_block4_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv4_block4_3_conv (Conv2  (None, 16, 16, 1024)         263168    ['conv4_block4_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv4_block4_out (Add)      (None, 16, 16, 1024)         0         ['conv4_block3_out[0][0]',    
                                                                         'conv4_block4_3_conv[0][0]'] 
                                                                                                      
     conv4_block5_preact_bn (Ba  (None, 16, 16, 1024)         4096      ['conv4_block4_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv4_block5_preact_relu (  (None, 16, 16, 1024)         0         ['conv4_block5_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv4_block5_1_conv (Conv2  (None, 16, 16, 256)          262144    ['conv4_block5_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv4_block5_1_bn (BatchNo  (None, 16, 16, 256)          1024      ['conv4_block5_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block5_1_relu (Activ  (None, 16, 16, 256)          0         ['conv4_block5_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv4_block5_2_pad (ZeroPa  (None, 18, 18, 256)          0         ['conv4_block5_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv4_block5_2_conv (Conv2  (None, 16, 16, 256)          589824    ['conv4_block5_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv4_block5_2_bn (BatchNo  (None, 16, 16, 256)          1024      ['conv4_block5_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block5_2_relu (Activ  (None, 16, 16, 256)          0         ['conv4_block5_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv4_block5_3_conv (Conv2  (None, 16, 16, 1024)         263168    ['conv4_block5_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv4_block5_out (Add)      (None, 16, 16, 1024)         0         ['conv4_block4_out[0][0]',    
                                                                         'conv4_block5_3_conv[0][0]'] 
                                                                                                      
     conv4_block6_preact_bn (Ba  (None, 16, 16, 1024)         4096      ['conv4_block5_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv4_block6_preact_relu (  (None, 16, 16, 1024)         0         ['conv4_block6_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv4_block6_1_conv (Conv2  (None, 16, 16, 256)          262144    ['conv4_block6_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv4_block6_1_bn (BatchNo  (None, 16, 16, 256)          1024      ['conv4_block6_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block6_1_relu (Activ  (None, 16, 16, 256)          0         ['conv4_block6_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv4_block6_2_pad (ZeroPa  (None, 18, 18, 256)          0         ['conv4_block6_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv4_block6_2_conv (Conv2  (None, 8, 8, 256)            589824    ['conv4_block6_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv4_block6_2_bn (BatchNo  (None, 8, 8, 256)            1024      ['conv4_block6_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv4_block6_2_relu (Activ  (None, 8, 8, 256)            0         ['conv4_block6_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     max_pooling2d_2 (MaxPoolin  (None, 8, 8, 1024)           0         ['conv4_block5_out[0][0]']    
     g2D)                                                                                             
                                                                                                      
     conv4_block6_3_conv (Conv2  (None, 8, 8, 1024)           263168    ['conv4_block6_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv4_block6_out (Add)      (None, 8, 8, 1024)           0         ['max_pooling2d_2[0][0]',     
                                                                         'conv4_block6_3_conv[0][0]'] 
                                                                                                      
     conv5_block1_preact_bn (Ba  (None, 8, 8, 1024)           4096      ['conv4_block6_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv5_block1_preact_relu (  (None, 8, 8, 1024)           0         ['conv5_block1_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv5_block1_1_conv (Conv2  (None, 8, 8, 512)            524288    ['conv5_block1_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv5_block1_1_bn (BatchNo  (None, 8, 8, 512)            2048      ['conv5_block1_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv5_block1_1_relu (Activ  (None, 8, 8, 512)            0         ['conv5_block1_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv5_block1_2_pad (ZeroPa  (None, 10, 10, 512)          0         ['conv5_block1_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv5_block1_2_conv (Conv2  (None, 8, 8, 512)            2359296   ['conv5_block1_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv5_block1_2_bn (BatchNo  (None, 8, 8, 512)            2048      ['conv5_block1_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv5_block1_2_relu (Activ  (None, 8, 8, 512)            0         ['conv5_block1_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv5_block1_0_conv (Conv2  (None, 8, 8, 2048)           2099200   ['conv5_block1_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv5_block1_3_conv (Conv2  (None, 8, 8, 2048)           1050624   ['conv5_block1_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv5_block1_out (Add)      (None, 8, 8, 2048)           0         ['conv5_block1_0_conv[0][0]', 
                                                                         'conv5_block1_3_conv[0][0]'] 
                                                                                                      
     conv5_block2_preact_bn (Ba  (None, 8, 8, 2048)           8192      ['conv5_block1_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv5_block2_preact_relu (  (None, 8, 8, 2048)           0         ['conv5_block2_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv5_block2_1_conv (Conv2  (None, 8, 8, 512)            1048576   ['conv5_block2_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv5_block2_1_bn (BatchNo  (None, 8, 8, 512)            2048      ['conv5_block2_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv5_block2_1_relu (Activ  (None, 8, 8, 512)            0         ['conv5_block2_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv5_block2_2_pad (ZeroPa  (None, 10, 10, 512)          0         ['conv5_block2_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv5_block2_2_conv (Conv2  (None, 8, 8, 512)            2359296   ['conv5_block2_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv5_block2_2_bn (BatchNo  (None, 8, 8, 512)            2048      ['conv5_block2_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv5_block2_2_relu (Activ  (None, 8, 8, 512)            0         ['conv5_block2_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv5_block2_3_conv (Conv2  (None, 8, 8, 2048)           1050624   ['conv5_block2_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv5_block2_out (Add)      (None, 8, 8, 2048)           0         ['conv5_block1_out[0][0]',    
                                                                         'conv5_block2_3_conv[0][0]'] 
                                                                                                      
     conv5_block3_preact_bn (Ba  (None, 8, 8, 2048)           8192      ['conv5_block2_out[0][0]']    
     tchNormalization)                                                                                
                                                                                                      
     conv5_block3_preact_relu (  (None, 8, 8, 2048)           0         ['conv5_block3_preact_bn[0][0]
     Activation)                                                        ']                            
                                                                                                      
     conv5_block3_1_conv (Conv2  (None, 8, 8, 512)            1048576   ['conv5_block3_preact_relu[0][
     D)                                                                 0]']                          
                                                                                                      
     conv5_block3_1_bn (BatchNo  (None, 8, 8, 512)            2048      ['conv5_block3_1_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv5_block3_1_relu (Activ  (None, 8, 8, 512)            0         ['conv5_block3_1_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv5_block3_2_pad (ZeroPa  (None, 10, 10, 512)          0         ['conv5_block3_1_relu[0][0]'] 
     dding2D)                                                                                         
                                                                                                      
     conv5_block3_2_conv (Conv2  (None, 8, 8, 512)            2359296   ['conv5_block3_2_pad[0][0]']  
     D)                                                                                               
                                                                                                      
     conv5_block3_2_bn (BatchNo  (None, 8, 8, 512)            2048      ['conv5_block3_2_conv[0][0]'] 
     rmalization)                                                                                     
                                                                                                      
     conv5_block3_2_relu (Activ  (None, 8, 8, 512)            0         ['conv5_block3_2_bn[0][0]']   
     ation)                                                                                           
                                                                                                      
     conv5_block3_3_conv (Conv2  (None, 8, 8, 2048)           1050624   ['conv5_block3_2_relu[0][0]'] 
     D)                                                                                               
                                                                                                      
     conv5_block3_out (Add)      (None, 8, 8, 2048)           0         ['conv5_block2_out[0][0]',    
                                                                         'conv5_block3_3_conv[0][0]'] 
                                                                                                      
     post_bn (BatchNormalizatio  (None, 8, 8, 2048)           8192      ['conv5_block3_out[0][0]']    
     n)                                                                                               
                                                                                                      
     post_relu (Activation)      (None, 8, 8, 2048)           0         ['post_bn[0][0]']             
                                                                                                      
    ==================================================================================================
    Total params: 23564800 (89.89 MB)
    Trainable params: 23519360 (89.72 MB)
    Non-trainable params: 45440 (177.50 KB)
    __________________________________________________________________________________________________
```  

On constate qu'il contient de nombreuses couches et donc de nombreux paramètres à entraîner. Pour optimiser le temps d'exécution, on décide de ne pas ré-entraîner ces paramètres :  


```python
# On définit les couches cachées comme non entraînables
#(on pourrait les réentrainer ou choisir de ne réentrainer que les premières par exemple)
for layer in classifier_resnet.layers:
    layer.trainable = False
```

## Structure de notre modèle  
Il ne nous reste plus qu'à **introduire ce bloc convolutif dans notre réseau de neurones**. On y ajoute en amont des couches permettant d'appliquer une mise à l'échelle uniforme (`Rescaling`) et des modifications aléatoires des images d'entraînement (`RandomRotation`, `RandomZoom`) permettant d'obtenir un **modèle plus généralisable**. En sortie du modèle ResNet, **on ajoute une couche `Flatten()`** qui sort un vecteur de données à partir de la dernière couche de convolution, puis **quelques couches de réseau de neurones** qui serviront à entraîner notre modèle sur nos données.  
On choisit ici d'ajouter 2 couches `Dense` de 256 et 128 neurones. Les nombres de couches et de neurones que vous allez implémenter dépendent de **la complexité du modèle souhaitée et de la taille des données d'entraînement dont vous disposez**. Il y a un arbitrage à faire entre [performances du modèle et risques de surapprentissage](https://blog.statoscop.fr/le-dilemme-biais-variance-dans-la-modelisation-de-donnees.html). En sortie, la dernière couche doit avoir un **nombre de neurones égal au nombre de classes** de votre problème et une **fonction d'activation** correspondant à un problème de classification (ici `softmax`).   


```python
from keras.models import Sequential
from keras.layers import Rescaling, RandomRotation, RandomFlip, RandomTranslation, RandomZoom, Flatten, Dropout, Dense, BatchNormalization
from keras.optimizers import Adam

# On définit l'ensemble de notre réseau
classifier_resnet_final = Sequential([
    Rescaling(1./255),
    RandomRotation((-0.2, 0.2)),
    RandomZoom((0, 0.3)),
    classifier_resnet,
    Flatten(),
    Dense(256, activation='relu'),
    Dense(128, activation='relu'),
    Dense(6, activation='softmax')
])

# On compile avec l'algorithme d'optimisation, la fonction de perte et la métrique en sortie : 
classifier_resnet_final.compile(optimizer='adam', loss='categorical_crossentropy', metrics=['accuracy'])
```

Enfin, il ne nous reste plus qu'à lancer **l'entraînement de notre modèle sur nos données**. En amont, on crée un `ModelCheckPoint` qui va permettre de sauvegarder sur notre disque le modèle le plus performant :  


```python
# On fait un modèle check point
from keras.callbacks import ModelCheckpoint
model_checkpoint_callback = ModelCheckpoint(
    filepath="model/best_{epoch:02d}.keras",
    save_weights_only=False,
    save_best_only = True,
    monitor='val_accuracy',
    mode='max')
```

On lance finalement l'entraînement, en choisissant __20 epochs__. Les époques sont **le nombre de fois où chaque point sera vu par le modèle**. La taille du __batch__ correspond au nombre d'images dans chaque lot de données qui sera envoyé dans le modèle avant mise à jour de ses paramètres. Elle a été définie dans la fonction `image_dataset_from_directory` et est de 64 images. Cela signifie que ma machine doit pouvoir supporter de charger 64 images en mémoire. Pour chaque époque, il y a donc X étapes, ou passages de batch, permettant que l'ensemble des images des données d'entraînement soit vu par le modèle.


```python
# On lance l'entraînement 
fitting_resnet = classifier_resnet_final.fit(
    training_set,
    epochs = 20,
    validation_data = validation_set,
    callbacks=[model_checkpoint_callback]) 
```

```python  

    Epoch 1/20
    15/15 [==============================] - 104s 7s/step - loss: 10.5564 - accuracy: 0.5216 - val_loss: 2.7596 - val_accuracy: 0.7935
    Epoch 2/20
    15/15 [==============================] - 98s 7s/step - loss: 2.3802 - accuracy: 0.8050 - val_loss: 2.0154 - val_accuracy: 0.8312
    Epoch 3/20
    15/15 [==============================] - 99s 7s/step - loss: 1.3172 - accuracy: 0.8588 - val_loss: 1.5961 - val_accuracy: 0.8564
    Epoch 4/20
    15/15 [==============================] - 100s 7s/step - loss: 0.7892 - accuracy: 0.8879 - val_loss: 1.9057 - val_accuracy: 0.8363
    Epoch 5/20
    15/15 [==============================] - 99s 7s/step - loss: 0.8162 - accuracy: 0.8944 - val_loss: 1.4213 - val_accuracy: 0.8539
    Epoch 6/20
    15/15 [==============================] - 98s 7s/step - loss: 0.6900 - accuracy: 0.8922 - val_loss: 1.1035 - val_accuracy: 0.8866
    Epoch 7/20
    15/15 [==============================] - 97s 7s/step - loss: 0.4349 - accuracy: 0.9397 - val_loss: 1.2567 - val_accuracy: 0.8514
    Epoch 8/20
    15/15 [==============================] - 97s 7s/step - loss: 0.4405 - accuracy: 0.9461 - val_loss: 1.1115 - val_accuracy: 0.8866
    Epoch 9/20
    15/15 [==============================] - 97s 7s/step - loss: 0.5066 - accuracy: 0.9321 - val_loss: 1.3761 - val_accuracy: 0.8816
    Epoch 10/20
    15/15 [==============================] - 100s 7s/step - loss: 0.3270 - accuracy: 0.9461 - val_loss: 1.4117 - val_accuracy: 0.8967
    Epoch 11/20
    15/15 [==============================] - 97s 7s/step - loss: 0.4974 - accuracy: 0.9429 - val_loss: 1.1139 - val_accuracy: 0.8866
    Epoch 12/20
    15/15 [==============================] - 96s 7s/step - loss: 0.2116 - accuracy: 0.9558 - val_loss: 1.3811 - val_accuracy: 0.8665
    Epoch 13/20
    15/15 [==============================] - 97s 7s/step - loss: 0.2452 - accuracy: 0.9666 - val_loss: 1.0028 - val_accuracy: 0.8866
    Epoch 14/20
    15/15 [==============================] - 97s 7s/step - loss: 0.1808 - accuracy: 0.9677 - val_loss: 1.3891 - val_accuracy: 0.8438
    Epoch 15/20
    15/15 [==============================] - 99s 7s/step - loss: 0.2319 - accuracy: 0.9580 - val_loss: 1.0152 - val_accuracy: 0.9093
    Epoch 16/20
    15/15 [==============================] - 98s 7s/step - loss: 0.2255 - accuracy: 0.9644 - val_loss: 1.1827 - val_accuracy: 0.8917
    Epoch 17/20
    15/15 [==============================] - 97s 7s/step - loss: 0.1173 - accuracy: 0.9720 - val_loss: 1.3951 - val_accuracy: 0.8791
    Epoch 18/20
    15/15 [==============================] - 97s 7s/step - loss: 0.2391 - accuracy: 0.9612 - val_loss: 1.2992 - val_accuracy: 0.8816
    Epoch 19/20
    15/15 [==============================] - 98s 7s/step - loss: 0.1443 - accuracy: 0.9688 - val_loss: 1.4597 - val_accuracy: 0.8791
    Epoch 20/20
    15/15 [==============================] - 96s 7s/step - loss: 0.2531 - accuracy: 0.9688 - val_loss: 1.3889 - val_accuracy: 0.8967
```

Le modèle finalement sélectionné est celui à la 15e époque. On constate en effet que l'accuracy n'augmente plus que sur les données d'entraînement au delà. 

# Évaluation et utilisation du modèle  
Une fois le modèle entraîné, il faut évaluer ses performances avant de l'utiliser pour réaliser de nouvelles prédictions.  

## Évaluation du modèle avec `scikit-learn` 

On utilise enfin notre set de test, importé au début de cet article en même temps que les sets d'entraînement de validation. En effet, si le modèle n'a pas été entraîné à proprement parler sur le set de validation, ce dernier a __servi à sélectionner le modèle final durant l'entraînement__ et n'est donc pas adapté pour en fournir une évaluation non biaisée. On évalue donc maintenant les performances du modèle finalement sélectionné, qui avait obtenu une accuracy de 91% sur les données de validation, avec les données test. 
On propose ici une **méthode pour utiliser les fonctions `classification_report` et `confusion_matrix` de `scikit-learn`** sur un modèle `keras`.  


```python
import numpy as np
from sklearn.metrics import classification_report
from keras.saving import load_model

modele_resnet = load_model("model/best_15.keras")

# on parcourt le set de test et on enregistre pour chaque élément
# la prédiction et la vraie valeur
predictions = np.array([])
true_values =  np.array([])
for x, y in test_set:
  predictions = np.concatenate([predictions, np.argmax(modele_resnet.predict(x), axis=-1)])
  true_values = np.concatenate([true_values, np.argmax(y.numpy(), axis=-1)])
```



```python
print(classification_report(y_true = true_values, y_pred = predictions, target_names = test_set.class_names))
```

                     precision    recall  f1-score   support
    
     fraise fraîche       0.93      0.86      0.90        50
     fraise pourrie       0.87      0.96      0.91        50
    grenade fraîche       0.89      0.89      0.89        62
    grenade pourrie       0.79      0.76      0.78        50
      pêche fraîche       0.94      0.90      0.92        50
      pêche pourrie       0.83      0.87      0.85        68
    
           accuracy                           0.87       330
          macro avg       0.88      0.87      0.87       330
       weighted avg       0.87      0.87      0.87       330
    


On remarque tout d'abord que l'accuracy sur les données de test est assez proche de celle obtenue sur les données de validation puisqu'elle est de 87%. Cela semble indiquer que notre modèle se généralise bien. Le rapport de classification de scikit-learn permet de voir plus précisément les forces et faiblesses du modèle. En particulier, pour chaque classe, il donne :   

- la **précision** du modèle, qui permet de voir, pour l'ensemble des prédictions d'une classe, la part qui appartenait bien à cette classe.  

- le **rappel** du modèle, qui permet de voir, pour l'ensemble des images d'une classe, la part ayant été correctement prédite.
   
Il faut viser d'avoir les **meilleures performances possible sur ces deux indicateurs**. En effet, un modèle **très précis sur une classe**, c'est à dire dont les prédictions de cette classe sont souvent correctes, peut avoir malgré tout un **très mauvais rappel**, c'est à dire _manquer_ beaucoup d'images de cette classe en les prédisant à tort comme appartenant à d'autres classes.    

Ici, le rapport de classification nous indique par exemple que ___87% des prédictions "fraise pourrie" sont correctes (`précision`) mais que 96% des fraises pourries sont bien repérées par le modèle (`rappel`)__. Cela montre que la quasi totalité des fraises pourries sont classées comme telles, mais que 13% des prédictions de cette classe concernent d'autres groupes de fruits. On constate également une précision et un rappel plus bas que les autres classes pour les _grenades pourries_.

On peut compléter cette analyse avec une matrice de confusion : 


```python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay 

cm = confusion_matrix(true_values, predictions)
cmd = ConfusionMatrixDisplay(cm, display_labels = test_set.class_names)
cmd.plot()
plt.xticks(rotation=45)
plt.show()

```


    
![Pelican](../images/keras_tl/output_27_0.png)
    


L'analyse de la matrice de confusion permet d'affiner le constat du rapport de classification : on constate par exemple que les taux de précision et de rappel relativement bas pour les grenades pourries sont principalement dûes à une confusion avec les pêches pourries et les grenades fraîches.

## Prédictions sur des nouvelles images

On propose enfin une méthode pour afficher la prédiction sur une nouvelle image. Il est en effet nécessaire de charger l'image mais aussi de la modifier pour qu'elle corresponde aux fonctions de `keras`. Voici comment :  


```python
from keras import utils
# chargement de l'image à la taille correspondante
image = utils.load_img("Set test/pêche pourrie/rotten_peach_32.jpg", target_size = (IMG_SIZE, IMG_SIZE))
# transformation en matrice de pixels 
image_array = utils.img_to_array(image)
# ajout d'une dimension pour simuler la matrice d'images
image_array2 = np.expand_dims(image_array, axis = 0)

# prédiction
prediction = modele_resnet.predict(image_array2)
```


La prédiction est un vecteur de probabilités, on crée la valeur correspondant à la probabilité et le label pour ensuite pouvoir afficher ces informations au moment de la prédiction :  


```python
# la proba max
proba = np.max(prediction).round(2)
# l'index de la classe associée
index = np.argmax(prediction)
# le label de la classe prédite
label = test_set.class_names[index]
print(proba, label)
```

    1.0 pêche pourrie


Il n'y a plus qu'à afficher l'image chargée et la prédiction associée, par exemple en la mettant en forme sous forme de titre `matplotlib` :  


```python
import matplotlib.pyplot as plt
from PIL import Image
img = np.asarray(Image.open("Set test/pêche pourrie/rotten_peach_32.jpg"))
plt.title("Notre modèle prédit une "+label+" avec une probabilité de "+str(proba))
plt.imshow(img)
plt.axis('off')
plt.show()
```


    
![Pelican](../images/keras_tl/output_35_0.png)
    


Et voilà! Sur notre machine locale et [sans GPU](https://blog.statoscop.fr/yolov7-gradient.html#le-gpu), on a réussi à entraîner un modèle qui fonctionne plutôt bien! Il ne vous reste plus qu'à mettre tout ça dans une fonction et à l'intégrer... à votre [application Flask](https://flask.palletsprojects.com/en/3.0.x/) par exemple. Promis on vous explique comment faire tout ça très rapidement.  

C'est tout pour aujourd'hui! N'hésitez pas à [visiter notre site](https://www.statoscop.fr) et à nous suivre sur [Twitter](https://twitter.com/stato_scop) et [Linkedin](https://www.linkedin.com/company/statoscop). Pour retrouver le code ayant servi à générer cette note, vous pouvez vous rendre sur le [github de Statoscop](https://github.com/Statoscop/notebooks-blog). 

