# Face Swap Tutorial

## Tutorial
- Facial Landmark Detection
- Delaunay Triangulation
- Face Morphing
- Seamless Cloning


### 換臉的困難點
人腦可以很輕易的識別每個人的臉部特徵，並且能夠簡單的將兩個人臉對調。但是如果要完全透過電腦自動將人臉對調且不會讓人發現破綻其實是很複雜的一件事。人必須要透過專業才能PS出一個擬真的圖像，更何況是一個未經訓練的電腦。

以下整理四個換臉的困難點。首先人的面部特徵都大不相同，此外其實人臉並非完全對稱的。第二個是臉部的燈光照明與皮膚色調相結合可以使圖像看起來非常不同。第三個是當相機拍攝的角度不同每張照片的人臉角度也大不相同。最後一點是人的膚質都大不相同，如果硬是把臉部對調會發現在邊緣部分會很違和。

## FaceSwap : Step by Step using OpenCV
### 1. Face Alignment
人臉校準（alignment）是給定一張臉，找出需要的特徵點的位置，比如鼻子、眼睛、上嘴脣下側等等點的位置。下圖是[1adrianb/face-alignment](https://github.com/1adrianb/face-alignment)使用深度學習模型所得到的人臉 landmarks。

![](https://github.com/1adrianb/face-alignment/raw/master/docs/images/2dlandmarks.png)

#### 1.1 Facial Landmark Detection
如果我們要將某個人臉貼到目標的臉型上，透過 face alignment 的 landmarks 知道了點的位置之後再做一下位置驅動的變形，臉就能成功貼到目標物體上了。在進行 alignment 前，需要先取得 facial landmarks 才能進行校準，目前最常用的 facial landmark model 則是 Dlib 所提供的 68 點模型。

![](https://i.imgur.com/5DSa16y.png)

#### 1.2 Find Convex Hull
在一個實數向量空間中，對於給定集合，所有包含X的凸集的交集被稱為 convex hull。間單來說如下圖，所有的點找出一個最外層的線將所有的點框起來就稱為 convex hull。我們可以使用OpenCV 的 [concealHull 函式](https://docs.opencv.org/2.4/doc/tutorials/imgproc/shapedescriptors/hull/hull.html) 來計算一組點的 convex hull。

![](https://i.imgur.com/uD2B1vY.png)

#### 1.3 Delaunay Triangulation
下一步將所有的點使用德勞內三角化將臉部特徵分成較小的部分。德勞內三角化，由數學家Delaunay於1934所發明的一種三角剖分方法，亦即，假設平面上有數個點所形成的集合P，那麼我們可以將這些點用線連結起來，而且這些線段所構成任一三角形的外接圓，其圓內沒有任何其它的點在裏面。

- Voronoi Diagram
透過Voronoi Diagram方法可將所有距離黄色點最近的點劃為同一區域，事實上，這些黃色點就是Delaunay Triangulation三角形外切圓的圓心
![](https://i.imgur.com/eojxBvv.png)

#### 1.4 Affine warp triangles
這個步驟是將目標跟要被驅動的人臉找出 Delaunay Triangulation，之後再透過兩張圖彼此間的三角形頂點做臉部的融合。其作法就是將源面三角形仿射扭曲到目標面上。

![](https://i.imgur.com/T7n2jDM.png)
[reference](http://www.magicandlove.com/blog/2018/08/20/)

下圖為上述所有提到的方法最終的結果。我們可以發現人臉可以成功的對調了，但是似乎看起來有點違和感。尤其是在邊緣的部分，一看過去就能發現這是透過剪貼的感覺。接下來第二步驟 Seamless Cloning 就來跟各位介紹，如何將 mask 可以完整的融合到目標圖片中。

![](https://i.imgur.com/zZZ3TqR.png)


### 2. Seamless Cloning
圖像合成是通過將源圖像中一個物體或者一個區域嵌入到目標圖像生成一個新的圖像。在對圖像進行合成的過程中，為了使合成後的圖像更自然，合成邊界應當保持無縫。在 OpenCV 中有一個函式 `cv2.seamlessClone()` 專門可以將 mask 圖片完美的融合在目標影像中。我們可以從一個圖像中復制某一個物件，然後將此物件粘貼到另一圖像中，使合成看起來無縫且自然。此方法是根據原圖像的梯度以及目標圖像的邊界，利用插值的方法重新構建出合成區域內的圖像像素。

![](https://i.imgur.com/4aU50gE.png)


## Reference
[face-swap-using-opencv-c-python](https://www.learnopencv.com/face-swap-using-opencv-c-python/#download)

[Fast Face-swap Using Convolutional Neural Networks](https://arxiv.org/pdf/1611.09577.pdf)