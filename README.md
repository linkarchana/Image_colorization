# Image_colorization
##Image Colorization Using Conditional GANs

In this project goal is to create coloured images using black and white image as input. In order  to achieve this the following pipeline is implemented –

1)RGB image is converted to LAB image and the three channels are separated, this L channel is passed to generator as input.

2)Generator produces channel AB  using conditional GAN loss and L1 loss. This generator incorporates a U-net architecture.

3)Then a Patch Discriminator is used to train the model further by comparing it to original image.

###RGB vs L*a*b

When we load an image, we get a rank-3 (height, width, color) array with the last axis containing the color data for our image. These data represent color in RGB color space and there are 3 numbers for each pixel indicating how much Red, Green, and Blue the pixel is.
In L*a*b color space, we have again three numbers for each pixel but these numbers have different meanings. The first number (channel), L, encodes the Lightness of each pixel and when we visualize this channel (the second image in the row below) it appears as a black and white image. The *a and *b channels encode how much green-red and yellow-blue each pixel is, respectively.
![image](https://user-images.githubusercontent.com/101972579/175314977-bf48db47-322a-4214-9089-a695a38bbf08.png)

In colorisation problems we use lab image space as it is easier to predict 2 (A&B) channels than 3(R,G&B) channels.

###Conditional GAN

GANs are unsupervised deep learning techniques. Usually, it is implemented using two neural networks: Generator and Discriminator. These two models compete with each other in a form of a game setting. The GAN model would be trained on real data and data generated by the generator. The discriminator’s job is to determine fake from real data. The generator is a learning model, so initially, it is likely to produce low or even completely noisy data that does not reflect the real distribution or the properties of the real data.

The generator model’s primary goal is generating artificial data that can pass the discriminator successfully. The model starts taking some noise, usually Gaussian noise and produces an image formatted as a vector of pixels. The generator must learn how to trick the discriminator and win a positive classification (produced image classified as real). The generation step’s loss is computed whenever any of those generated images detected successfully as “fake”. The discriminator has to learn how to identify those fake images progressively. Negative loss is given to the discriminator whenever the model fails to recognize a fake image. The key concept is the simultaneous training of the generator and the discriminator at the same time.
![image](https://user-images.githubusercontent.com/101972579/175315689-e3600f04-43e9-448e-93b3-0ce0af3c22b6.png)
 
LGAN (G, D) =Ey[log D(y)]+Ex,z[log(1 − D(G(x, z))].

In cGANs, a conditional setting is applied, meaning that both the generator and discriminator are conditioned on some sort of auxiliary information (such as class labels or data) from other modalities. As a result, the ideal model can learn multi-modal mapping from inputs to outputs by being fed with different contextual information.
![image](https://user-images.githubusercontent.com/101972579/175315844-50e3f1ad-390c-4304-a06b-152bcd4a5221.png)

LcGAN (G, D) =Ex,y[log D(x, y)]+Ex,z[log(1 − D(x, G(x, z))]

###Loss function

The earlier loss function helps to produce good-looking colorful images that seem real, but to further help the models and introduce some supervision in our task, we combine this loss function with L1 Loss (you might know L1 loss as mean absolute error) of the predicted colors compared with the actual colors:
![image](https://user-images.githubusercontent.com/101972579/175315980-5c5986c2-c13c-434e-9cd7-c0d01f76f771.png)
 
If we use L1 loss alone, the model still learns to colorize the images but it will be conservative and most of the time uses colors like “gray” or “brown” because when it doubts which color is the best, it takes the average and uses these colors to reduce the L1 loss as much as possible (it is similar to the blurring effect of L1 or L2 loss in super resolution task). Also, the L1 Loss is preferred over L2 loss (or mean squared error) because it reduces that effect of producing greyish images. So, our combined loss function will be:
![image](https://user-images.githubusercontent.com/101972579/175316092-4448940f-cefe-4f6d-a1ce-153000ef7e1d.png)
 
where λ is a coefficient to balance the contribution of the two losses to the final loss (of course the discriminator loss does not involve the L1 loss).
Both generator and discriminator use modules of the form convolution-BatchNorm-ReLu

###Generator

We provide noise in the form of dropout, applied on several layers of our generator at both training and test time. A defining feature of image-to-image translation problems is that they map a high-resolution input grid to a high-resolution output grid. In encoder-decoder network  the input is passed through a series of layers that progressively downsample, until a bottleneck layer, at which point the process is reversed. Such a network requires that all information flow pass through all the layers, including the bottleneck. For many image translation problems, there is a great deal of low-level information shared between the input and output, and it would be desirable to shuttle this information directly across the net. To give the generator a means to circumvent the bottleneck for information like this, we add skip connections, following the general shape of a “U-Net” . Specifically, we add skip connections between each layer i and layer n − i, where n is the total number of layers. Each skip connection simply concatenates all channels at layer i with those at layer n − i.
 ![image](https://user-images.githubusercontent.com/101972579/175316194-e6a54235-ed6c-4e48-9dc8-3b44ef32007e.png)

###Discriminator

To model high frequencies, it is sufficient to restrict our attention to the structure in local image patches. Therefore, we design a discriminator architecture – which we term a PatchGAN – that only penalizes structure at the scale of patches. This discriminator tries to classify if each N ×N(here 70*70) patch in an image is real or fake. We run this discriminator convolutionally across the image, averaging all responses to provide the ultimate output of D.

Mentioned pix2pix paper was of great help throughout [1611.07004.pdf](https://github.com/linkarchana/Image_colorization/files/8966086/1611.07004.pdf)
