Convolutional Neural Network - MNIST
================
Machiel Visser
12/11/2020

## Load packages

``` r
library(knitr)
library(tidyverse)
library(keras)

set.seed(20201112)
```

## YouTube tutorial by Andrew Couch

<https://www.youtube.com/watch?v=E9IVgP5cTxA>

## Load MNIST data

``` r
mnist_df <- dataset_mnist()

train_images <- mnist_df$train$x / 255
train_labels <- to_categorical(mnist_df$train$y)

test_images <- mnist_df$test$x / 255
test_labels <- to_categorical(mnist_df$test$y)
```

## Plot one image

``` r
image <- t(train_images[18, , ])

colnames(image) <- c(letters, "za", "zb")
rownames(image) <- c(letters, "za", "zb")

image %>% 
  as.data.frame() %>%
  rownames_to_column(var = "row") %>%
  pivot_longer(cols = a:zb, names_to = "column", values_to = "value") %>% 
  ggplot(aes(x = row, y = column, fill = value)) +
  geom_raster() +
  scale_fill_continuous(low = "grey", high = "red")+
  labs(title = paste("Training image of digit", mnist_df$train$y[18]),
       x = "",
       y = "") +
  theme_void()
```

![](Convolutional-Neural-Network-MNIST/CNN-MNIST-unnamed-chunk-4-1.png)<!-- -->

## Neural network

### Reshape datasets for neural network

``` r
train_images <- array_reshape(train_images, 
                              c(dim(train_images)[1], 
                                dim(train_images)[2] * dim(train_images)[3]))
test_images <- array_reshape(test_images, 
                             c(dim(test_images)[1], 
                               dim(test_images)[2] * dim(test_images)[3]))
```

### Define neural network model

``` r
standard_image_model <- keras_model_sequential()

standard_image_model %>% 
  layer_dense(units = 512, input_shape = ncol(train_images), 
              activation = "relu") %>% 
  layer_dense(units = 512, activation = "relu") %>% 
  layer_dense(units = 512, activation = "relu") %>% 
  layer_dense(units = 10, activation = "softmax")

standard_image_model %>% 
  compile(optimizer = "adam",
          loss = "categorical_crossentropy",
          metrics = "accuracy")
```

### Train neural network model

``` r
test <- standard_image_model %>% 
  fit(x = train_images,
      y = train_labels,
      epochs = 50,
      batch_size = 128,
      validation_split = .8,
      callbacks = callback_early_stopping(patience = 5))
```

## Convolutional neural network

### Reshape datasets for convolutional neural network

``` r
conv_train_images <- array_reshape(mnist_df$train$x, c(dim(mnist_df$train$x)[1], 
                                                       dim(mnist_df$train$x)[2],
                                                       dim(mnist_df$train$x)[3], 
                                                       1))

conv_test_images <- array_reshape(mnist_df$test$x, c(dim(mnist_df$test$x)[1], 
                                                     dim(mnist_df$test$x)[2],
                                                     dim(mnist_df$test$x)[3], 
                                                     1))
```

### Define convolutional neural network model

``` r
conv_image_model <- keras_model_sequential()

conv_image_model %>% 
  layer_conv_2d(filters = 32, kernel_size = c(3, 3), input_shape = c(28, 28, 1), 
                activation = "relu") %>% 
  layer_max_pooling_2d(pool_size = c(2, 2)) %>% 
  layer_conv_2d(filters = 32, kernel_size = c(3, 3), activation = "relu") %>% 
  layer_max_pooling_2d(pool_size = c(2, 2)) %>%
  layer_flatten() %>% 
  layer_dense(units = 128, activation = "relu") %>% 
  layer_dense(units = 10, activation = "softmax")

conv_image_model %>% 
  compile(optimizer = "adam",
          loss = "categorical_crossentropy",
          metrics = "accuracy")
```

### Train convolutional neural network model

``` r
conv_image_model %>% 
  fit(x = conv_train_images,
      y = train_labels,
      epochs = 50,
      batch_size = 128,
      validation_split = .8,
      callbacks = callback_early_stopping(patience = 5))
```

### Evaluate model performances

``` r
evaluate(standard_image_model, test_images, test_labels)
```

    ## $loss
    ## [1] 0.1817176
    ## 
    ## $accuracy
    ## [1] 0.9582

``` r
evaluate(conv_image_model, conv_test_images, test_labels)
```

    ## $loss
    ## [1] 0.1299641
    ## 
    ## $accuracy
    ## [1] 0.9732
