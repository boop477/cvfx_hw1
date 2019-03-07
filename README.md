## CVFX Team 1

**Goal**: 將`Source Image/Painting`的色調轉換成`Target Image/Painting`的色調。

### RGB Color Space to lαβ Color Space
&emsp;&emsp;為了將一張圖片上的特徵轉移到另一張圖片上，需要選擇一個合適的color space以此進行操作。而常見的RGB space中R、G、B三個通道之間具有一定程度的關聯性，改變顏色的同時恰當的改變三通道比較困難。而通道間沒有關聯性（正交）的lαβ color space時，能夠較好的保持原圖的自然效果。

#### Steps:
1. 取得Target Image與Source Image的RGB值。
2. RGB space->LMS space。因為lαβ space是LMS cone space轉變來的，所以先將RGB space圖像轉變為LMS space。包含以下兩個步驟：
* &emsp;&ensp;RGB space->XYZ tristimulus values
<br>&ensp;![](https://i.imgur.com/kSAInLC.png)
* &emsp;&ensp;XYZ tristimulus values->LMS space
<br>&emsp;![](https://i.imgur.com/zH8wIwC.png)
3. 分別對LMS取以10為底的對數，得到相對應的**L&nbsp;M&nbsp;S**。
<br>&emsp;&emsp;![](https://i.imgur.com/Ue2Wf1T.png)
4. LMS space->lαβ color space.
<br>&emsp;![](https://i.imgur.com/3Whv0yH.png)
5. 用統計學的方法來進行顏色校正。
* 分別求出l、α、β各自的平均值(<>)和標準差(σ)。
* Source Image中l、α、β減去各自的均值，並乘上Target Image與Source Image的比值，得到如下公式l'、α'、β'的值。
<br>&emsp;&ensp;![](https://i.imgur.com/ir7uNi7.png)&emsp;&emsp;![](https://i.imgur.com/inkmYEr.png)
* 再加上Target Image中l、α、β的均值。
6. lαβ space->LMS space->RGB space.將Step5最後得到的結果通過step4、2的對應的反矩陣計算轉回XYZ color space。

#### Result：
* **Real photo**

![](https://i.imgur.com/ddbDQ7O.png)
&emsp;&emsp;本方法將一張圖片（Target Image）的顏色表現轉移到另一張圖（Original Image）上，但當Target Image和Source Image差異過大時，其效果並沒有表現得很好。

* **Photo to Monet**

![](https://i.imgur.com/D0AQ8Gn.png=500x500)<br>

&emsp;&emsp;當莫內的畫的風格轉移到真實照片上時，可以看到莫內風格的畫在色彩的表達上比較明艷，最後僅顏色的飽和度有很明顯的提升，但油畫感並不強烈。



---
### Histogram Equalization on RGB space
&emsp;&emsp;histogram equalization（直方圖均化）是一種圖像增强方式。我們以灰階圖片為範例，取得灰階圖片的累積分布函數（CDF），把原本集中在某區塊的機率函數(PDF)平均分布在所有顏色上面，達到增加圖片的對比度的效果。
![](https://i.imgur.com/EVM5MmV.png)
<br>![](https://i.imgur.com/STMmlJl.png)
<br>&emsp;&emsp;在image color transfer中，我們可以利用提取Target image的RGB三通道的累積分布函數（CDF），來對Source Image進行histogram equalization（直方圖均化），以達到將Source image的顔色分佈特徵轉化為Target image的顔色分佈特徵，從而實現image color transfer.

#### Steps:
1. 取得Target Image與Source Image的RGB三通道累積分布函數（CDF）。
2. 使用Target Image的RGB三通道的累積分布函數（CDF）對Source Image的RGB三通道分別進行histogram equalization（直方圖均化）.
3. Target Image的顔色分佈特徵將應用在Source Image上。

#### Result:
![](https://i.imgur.com/TtTXUWO.jpg)
<br>*Left*: Source images. *Middle*: Target paintings. *Right*: Results  with histogram equalization on R, G, and B channels.

![](https://i.imgur.com/wDeKlMh.jpg)
<br>*Left*: Source paintings. *Middle*: Target images. *Right*: Results  with histogram equalization on R, G, and B channels.


---

### Histogram Matching
&emsp;&emsp;Histogram Matching是以累積分布函數（CDF）來建立新舊亮度色彩分佈的對應關係，或對不同圖像進行比較建立對應關係。也正是這一特性，我們可以通過Histogram Matching來將一張圖像（Target Image）的色彩分佈（color distribution）應用在另一張圖像上（Source Image），實現color transfer。

#### Step:
1. 計算取得Target Image與Source Image的累積分布函數（CDF）![](https://i.imgur.com/s4Cr08V.png)和![](https://i.imgur.com/3h0xVqW.png)。
2. 爲了使Source Image的色彩向Target Image靠近，我們需要計算得出![](https://i.imgur.com/s4Cr08V.png)和![](https://i.imgur.com/3h0xVqW.png)的最小絕對差(演算法如下)，以8bit的圖片爲例，我們需要分別重新映射(mapping)RGB三個通道的256個顔色層級，使兩張圖片的顔色分佈靠近。
![](https://i.imgur.com/Otr06Y9.png)
3. 將得出的重新映射(mapping))數組應用在Source Image上。

#### Result:
* **Real photo**<br>
![](https://i.imgur.com/ZiRhu3f.png)
![](https://i.imgur.com/Y7bIqa2.png)
![](https://i.imgur.com/L45EKHy.png)
![](https://i.imgur.com/8H5mhzU.png)


<br>*Left*: Source images. *Middle*: Target real photograph. *Right*: Results  with histogram matching on R, G, and B channels.
* **Photo to Monet**
![](https://i.imgur.com/A42czSl.jpg)
![](https://i.imgur.com/IKuWiLy.jpg)
![](https://i.imgur.com/r3qVjsp.jpg)

<br>*Left*: Source paintings. *Middle*: Target paintings. *Right*: Results  with histogram matching on R, G, and B channels.

&emsp;&emsp;我們可以看到通過Histogram Equalization或Histogram Matching之後Source Image的color distribution與Target Image靠近了許多，兩張圖的顔色分佈特徵已經十分靠近，雖然兩種方法本質不盡相同，但是過於基礎的演算法在transfer的質量上的反應都并不優質。Real photo的color transfer過於生硬甚至有輕微色偏，Photo to Monet中油畫的感覺并不突出，這兩種方法的作用主要的還是體現在基礎的color distribution transfer之中。




---
###  N-dimensional PDF transfer 

&emsp;&emsp;在兩個圖像之間的顏色轉移過程中，如果Target Image的機率密度函數（probability density function，PDF）是均勻的，則可以用來進行簡單的非現實渲染。通過迭代一維的pdf來確保其傳輸的精度，最後收斂至Target Image的pdf，且僅有線性的複雜度O(M)（M為處理的樣本數量），在顏色映射中能夠具有更高的準確度。

#### Steps：
1. 旋轉迭代第k代樣本X<sup>(k)</sup>和Target Image樣本Y，以此來改變坐標系。
2. 兩個樣本的分佈分別投影到新的座標軸上（邊緣f<sub>i</sub>，g<sub>i</sub>）。
3. 用如下公式找到每個軸的映射t<sub>i</sub>，將邊緣f<sub>i</sub>轉移到g<sub>i</sub>。X和Y分別是f(x)和g(x)的pdf，C<sub>x</sub>和C<sub>y</sub>分別是X和Y的累積。
![](https://i.imgur.com/9aluqs5.png)
4. 再用得到的t將樣本（x<sub>1</sub>,...,x<sub>N</sub>）轉換為t(x<sub>1</sub>,...,x<sub>N</sub>)=(t<sub>1</sub>(x<sub>1</sub>),...,t<sub>N</sub>(x<sub>N</sub>))。
5. 用R-1旋轉樣本完成迭代回到原本的坐標系中。

&emsp;&emsp;如圖演算法所示。
![](https://i.imgur.com/Z54vHOY.png)
&emsp;&emsp;如果重複足夠多次不同的旋轉，該算法可以收斂至f<sup>(∞)</sup> = g （隨機旋轉已經足夠收斂）。

#### Result：
* **Real Photo**

![](https://i.imgur.com/PlSDGLf.jpg)

<br>&emsp;&emsp;Original Image是因為雲太厚、方向沒有正對太陽升起的地方而沒看到的日出。
<br>&emsp;&emsp;Target Image是高美濕地的日落。
<br>&emsp;&emsp;Result Image合成出來有海邊晚霞的感覺。
<br>&emsp;&emsp;將兩張圖片分別迭代1次、5次和10次的結果如上圖所示。可以看出迭代次數越多，Target Image的色彩轉移到Original Image上就越豐富，但是同時隨著迭代次數的增加，圖片的顆粒感加重，並在圖片上方中間產生些許破碎感。

* **Photo to Monet**

![](https://i.imgur.com/xxovc68.png)
<br>&emsp;&emsp;用莫內風格的畫對於真實照片進行處理，可以很明顯看出處理後的照片邊緣銳化，部分處理得有些失真，但對於色彩鮮艷對比度高的照片處理後和莫內風格還是有些許相似。


---

### CycleGAN
&emsp;&emsp;CycleGAN使用Neural Network做color transfer。CycyeGAN使用一組Generator生成兩個domain的圖片，以及一組Discriminator分辨真實圖片和生成圖片。
<br>![](https://i.imgur.com/IW9NOYj.jpg)<br>
Generator `G` 將domain `X` 的圖片生成對應到domain `Y` 的圖片；Generator `F` 將domain `Y` 的圖片生成對應到domain `X` 的圖片；Discriminator `D_x` 分辨domain `X`圖片的真偽；Discriminator `D_y` 分辨domain `Y`圖片的真偽 。

CycleGAN在training時期需要training data，因此在測試圖片時是轉換成在training時期學到所有圖片的色調，和前面提到可以指定單張target image的color transfer方法有所不同。

#### Steps :
1. Train generators and discriminators by minimizing the loss function
    <br>![](https://i.imgur.com/DdhtJ7a.jpg)<br>
    which consists of adversarial loss
    <br>![](https://i.imgur.com/2BBPCDu.jpg)<br>
    and cycle consistency loss.
    <br>![](https://i.imgur.com/PeCFt1f.jpg)<br>
    
2. Use generator `G` to transfer images of domain `X` to domain `Y`; use generator `F` to transfer images of domain `Y` to domain `X`.

#### Result:

我們跑了160 iterations.
![](https://i.imgur.com/dubYzNE.jpg)

* **Photo to Monet**
    <br>![Alt Text](https://media.giphy.com/media/oHxVAEVT1UWl6aKsGJ/giphy.gif)
    <br>![Alt Text](https://media.giphy.com/media/YWbwRFT9yzIKl5uemH/giphy.gif)
    <br>![Alt Text](https://media.giphy.com/media/wsWUEcP6q0PXS3sLcy/giphy.gif)
    <br>*Left* : Photo → Monet. *Right* : Photo.
    
    我們比較Monet to Photo在不同iteration的效果
    
    ![](https://i.imgur.com/G7WA7PJ.jpg)
    ![](https://i.imgur.com/f4uuHxZ.jpg)
    ![](https://i.imgur.com/yPh7TtY.jpg)
    ![](https://i.imgur.com/ItGzfQc.jpg)
    
    在前期iteration著重整體色調變化，後期iteration著重在模仿莫內捕捉光影的方式，轉換後圖片也出現比較明顯的筆觸以及強調光影變化的色塊。
    
    ![](https://i.imgur.com/gMjRKQd.jpg)
    ![](https://i.imgur.com/YJ97AsS.jpg)
    ![](https://i.imgur.com/UNqWLVW.jpg)
    
    這組轉換後效果比較差，可能因為在training set裡面以莫內的戶外風景畫為主，而且缺少建築物的畫。



    
* **Monet to Photo**
    <br>![Alt Text](https://media.giphy.com/media/3Foz71Mh6IMOzjBzRB/giphy.gif)
    <br>![Alt Text](https://media.giphy.com/media/1NXFxgF2D5Fl3kMnrf/giphy.gif)
    <br>![Alt Text](https://media.giphy.com/media/pVZRrZhlVO200JzDDn/giphy.gif)
    <br>*Left* : Monet → Photo. *Right* : Photo.
    
    我們比較Monet to Photo在不同iteration的效果
    
    ![](https://i.imgur.com/Zsb0lZH.jpg)
    ![](https://i.imgur.com/Miqc63T.jpg)
    ![](https://i.imgur.com/9Sc7kw2.jpg)
    ![](https://i.imgur.com/Lui4F9d.jpg)
    ![](https://i.imgur.com/YA6VYod.jpg)
    ![](https://i.imgur.com/Yru7XOq.jpg)
    ![](https://i.imgur.com/1XUbYBK.jpg)
    
    前期iteration著重在轉換整張畫面的色調而忽略細節，後期則著重在調整細節的色調，像是左側樹叢的陰影和遠方水上的倒影。
    
    ![](https://i.imgur.com/3LzXNid.jpg)
    ![](https://i.imgur.com/SC17BZy.jpg)
    
    把50th iteration和160th iteration比較之下發現：原畫中被紅框圈起來的部分
    
    ![](https://i.imgur.com/iVwmDQs.png)
    
    在160th iteration被轉換成厚雲層蓋住橘色天空的畫面(藍色框)。
    
    ![](https://i.imgur.com/iG8h6w3.png)
    
* **Other photos and paintings**
    <br>我們也使用自己拍攝的照片測試結果，挑選和莫內的畫有相似的取景以及內容的相片來測試。
    <br>![Alt Text](https://media.giphy.com/media/PQb9RWoMFH0tE1zJFG/giphy.gif)
    <br>![Alt Text](https://media.giphy.com/media/7TkSVM4JwLwwTx6UOy/giphy.gif)
    <br>![Alt Text](https://media.giphy.com/media/vL7tRZlKvuaKFMPDAW/giphy.gif)
    <br>*Left* : Photo → Monet. *Right* : Photo.
    
    三張照片的內容是樹和倒影(110th iteration)、夕陽(160th iteration)以及睡蓮(160th iteration)。
    
    ![](https://i.imgur.com/ojNfPpB.jpg)
    
    我們拿上面三張生成出的照片請一位學西畫、開了畫廊並且賣了不少畫的畫家鑑定。他表示這些畫是學法國印象派的畫風，第二張(夕陽)長得很像莫內的*印象．日出(Impression, soleil levant)*，第三張像莫內的*睡蓮(Water Lilies)*。
    
    我們也測試Monet to Photo的效果，不過選用了其他畫家的作品來做測試。
    ##### 米勒(Millet)
    ![Alt Text](https://media.giphy.com/media/37q1LhM3pMkeiVOIfb/giphy.gif)
    <br>*Left* : Painting → Photo. *Right* : Painting.
    
    米勒表現手法和莫內不同，風格偏向寫實。不論前期或是後期生成的圖片都在變化整體色調
    ##### 雷諾瓦(Renoir)
    <br>![Alt Text](https://media.giphy.com/media/1wPBn9e49es8ZYh400/giphy.gif)
    <br>![Alt Text](https://media.giphy.com/media/3JuBfnVX8JnjL4tiGk/giphy.gif)
    <br>*Left* : Painting → Photo. *Right* : Painting.
    
    雷諾瓦和莫內畫風相似，同樣是印象派的領導人物。
    
    ![](https://i.imgur.com/5SVkn30.jpg)
    ![](https://i.imgur.com/oA9po61.jpg)
    
    150th iteration生成出的照片比較有立體感，整體色調比50th iteration更像自然影像，遠方的天空也能處理得比較平滑。

    ##### 布丹(Eugène Boudin)
    <br>![Alt Text](https://media.giphy.com/media/9xyGHahdX7Nl2eqPRW/giphy.gif)
    <br>![Alt Text](https://media.giphy.com/media/QNVQnratVHMQqTaH1x/giphy.gif)
    <br>*Left* : Painting → Photo. *Right* : Painting.
    
    布丹的風格介於沈穩寫實的米勒和強調光影的莫內之間，不過轉換後的效果並不是很明顯。
    
    ![](https://i.imgur.com/tNfN843.jpg)
    ![](https://i.imgur.com/hZ3SvEH.jpg)
    
    原圖長這樣子：
    
    ![](https://i.imgur.com/0l3RUjD.jpg)
    *(Rouen, View over the River Seine, 1895 by Eugène Boudin)*
    
---
    
### Reference
[1] E. Reinhard, M. Ashikhmin, B. Gooch, and P. Shirley. Color transfer between images. IEEE Computer Graphics and Ap- plications, 21:34–41, 2001.
[2] F. Pitie ́, A. Kokaram, R. Dahyot, N-Dimensional probability density function transfer and its application to colour transfer, in: Interna- tional Conference on Computer Vision (ICCV’05), Beijing, 2005.
[3] Jun-Yan Zhu, Taesung Park, Phillip Isola, and Alexei A. Efros. Unpaired image-to-image translation using cycle-consistent adversarial networks. CoRR, abs/1703.10593, 2017.


