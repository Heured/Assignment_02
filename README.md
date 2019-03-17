# 作业2
  
ps:vlfeat安装参考:[这里](http://yongyuan.name/pcvwithpython/installation.html)
## SIFT特征原理
[参考:SIFT特征详解](https://www.cnblogs.com/wangguchangqing/p/4853263.html)
  
  
  
## SIFT特征匹配比较Harris特征匹配
原图：
  
![emmmm](https://github.com/Heured/Assignment_02/blob/master/data/P90317-103350-1.jpg)
![emmmm](https://github.com/Heured/Assignment_02/blob/master/data/P90317-103353-1.jpg)
  
  
### SIFT特征匹配
  
代码如下:  

```python
from PIL import Image
from pylab import *
import sys
from PCV.localdescriptors import sift


if len(sys.argv) >= 3:
  im1f, im2f = sys.argv[1], sys.argv[2]
else:
#  im1f = '../data/sf_view1.jpg'
#  im2f = '../data/sf_view2.jpg'
  im1f = './data/P90317-103353-1.jpg'
  im2f = './data/P90317-103350-1.jpg'
#  im1f = '../data/climbing_1_small.jpg'
#  im2f = '../data/climbing_2_small.jpg'
im1 = array(Image.open(im1f))
im2 = array(Image.open(im2f))

sift.process_image(im1f, 'out_sift_1.txt')
l1, d1 = sift.read_features_from_file('out_sift_1.txt')
figure()
gray()
subplot(121)
sift.plot_features(im1, l1, circle=False)

sift.process_image(im2f, 'out_sift_2.txt')
l2, d2 = sift.read_features_from_file('out_sift_2.txt')
subplot(122)
sift.plot_features(im2, l2, circle=False)

#matches = sift.match(d1, d2)
matches = sift.match_twosided(d1, d2)
print '{} matches'.format(len(matches.nonzero()[0]))

figure()
gray()
sift.plot_matches(im1, im2, l1, l2, matches, show_below=True)
show()
```
  
遇到问题：  
![emmmm](https://github.com/Heured/Assignment_02/blob/master/imgToShow/sift_match_error_1.JPG)
  
分析：百度无果，也许是vlfeat没有配置好
  
暂时解决：折腾了几个小时没弄好，只好先用cmd命令行运行生成out_sift_1.txt和out_sift_2.txt文件，然后把sift.process_image加上注释，日后再想办法
  
![emmmm](https://github.com/Heured/Assignment_02/blob/master/imgToShow/sift_match_proceed_1.JPG)
  
  
解决办法：尝试sift.py文件里vlfeat路径的\统统改为/，运行不再报错了(ps:在此环境下只能用win32的vlfeat，win64的只会生成空的.txt文件)
  
```python
cmmd = str("D:/pyCharm/pycharm_workspace/vlfeat/win32/sift.exe "+imagename+" --output="+resultname+
                " "+params)
```
  
结果：  
![emmmm](https://github.com/Heured/Assignment_02/blob/master/imgToShow/sift_match_done_2.JPG)
![emmmm](https://github.com/Heured/Assignment_02/blob/master/imgToShow/sift_match_done_1.JPG)
  
  
### Harris特征匹配
代码如下：  
```python
 # -*- coding: utf-8 -*-
from pylab import *
from PIL import Image

from PCV.localdescriptors import harris
from PCV.tools.imtools import imresize

"""
This is the Harris point matching example in Figure 2-2.
"""

# Figure 2-2上面的图
#im1 = array(Image.open("../data/crans_1_small.jpg").convert("L"))
#im2= array(Image.open("../data/crans_2_small.jpg").convert("L"))

# Figure 2-2下面的图
im1 = array(Image.open("./data/P90317-103353-1.jpg").convert("L"))
im2 = array(Image.open("./data/P90317-103350-1.jpg").convert("L"))

# resize加快匹配速度
im1 = imresize(im1, (im1.shape[1]/2, im1.shape[0]/2))
im2 = imresize(im2, (im2.shape[1]/2, im2.shape[0]/2))

wid = 5
harrisim = harris.compute_harris_response(im1, 5)
filtered_coords1 = harris.get_harris_points(harrisim, wid+1)
d1 = harris.get_descriptors(im1, filtered_coords1, wid)

harrisim = harris.compute_harris_response(im2, 5)
filtered_coords2 = harris.get_harris_points(harrisim, wid+1)
d2 = harris.get_descriptors(im2, filtered_coords2, wid)

print 'starting matching'
matches = harris.match_twosided(d1, d2)

figure()
gray()
harris.plot_matches(im1, im2, filtered_coords1, filtered_coords2, matches)
show()
```
  
结果：  
![emmmm](https://github.com/Heured/Assignment_02/blob/master/imgToShow/harris_match_done_1.JPG)
  
  
  
  
小结：sift运行速度比较快而且匹配更加准确
  
  
  
## 地理标记图像匹配
代码如下：
```python
# -*- coding: utf-8 -*-
from pylab import *
from PIL import Image
from PCV.localdescriptors import sift
from PCV.tools import imtools
import pydot

""" This is the example graph illustration of matching images from Figure 2-10.
To download the images, see ch2_download_panoramio.py."""

#download_path = "panoimages"  # set this to the path where you downloaded the panoramio images
#path = "/FULLPATH/panoimages/"  # path to save thumbnails (pydot needs the full system path)

download_path = r"D:\pyCharm\pycharm_workspace\HarrisAndSift\data"  # set this to the path where you downloaded the panoramio images
path = r"D:\pyCharm\pycharm_workspace\HarrisAndSift\data"  # path to save thumbnails (pydot needs the full system path)

# list of downloaded filenames
imlist = imtools.get_imlist(download_path)
nbr_images = len(imlist)

# extract features
featlist = [imname[:-3] + 'sift' for imname in imlist]
for i, imname in enumerate(imlist):
    sift.process_image(imname, featlist[i])

matchscores = zeros((nbr_images, nbr_images))

for i in range(nbr_images):
    for j in range(i, nbr_images):  # only compute upper triangle
        print 'comparing ', imlist[i], imlist[j]
        l1, d1 = sift.read_features_from_file(featlist[i])
        l2, d2 = sift.read_features_from_file(featlist[j])
        matches = sift.match_twosided(d1, d2)
        nbr_matches = sum(matches > 0)
        print 'number of matches = ', nbr_matches
        matchscores[i, j] = nbr_matches
print "The match scores is: \n", matchscores

# copy values
for i in range(nbr_images):
    for j in range(i + 1, nbr_images):  # no need to copy diagonal
        matchscores[j, i] = matchscores[i, j]

#可视化

threshold = 2  # min number of matches needed to create link

g = pydot.Dot(graph_type='graph')  # don't want the default directed graph

for i in range(nbr_images):
    for j in range(i + 1, nbr_images):
        if matchscores[i, j] > threshold:
            # first image in pair
            im = Image.open(imlist[i])
            im.thumbnail((100, 100))
            filename = path + str(i) + '.png'
            im.save(filename)  # need temporary files of the right size
            g.add_node(pydot.Node(str(i), fontcolor='transparent', shape='rectangle', image=filename))

            # second image in pair
            im = Image.open(imlist[j])
            im.thumbnail((100, 100))
            filename = path + str(j) + '.png'
            im.save(filename)  # need temporary files of the right size
            g.add_node(pydot.Node(str(j), fontcolor='transparent', shape='rectangle', image=filename))

            g.add_edge(pydot.Edge(str(i), str(j)))
g.write_png('zhongShanJNG.png')
```
  
遇到问题:
  
![emmmm](https://github.com/Heured/Assignment_02/blob/master/imgToShow/site_match_error_1.JPG)
  
  
分析：
解决办法：
