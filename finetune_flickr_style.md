### 数据准备
- 首先生成txt文件（train.txt和test.txt），包含图片路径和标签。生成两份。  
一份中图片路径不要写成全部路径，因为接下来的要用caffe/build/tools/convert_imageset生成lmdb文件时，第四个参数要填写一个路径，这个路径不能为空，这个路径和txt里面的图片路径组合起来找到图片，然后生成lmdb文件。  
另外一份中图片路径为全部路径，用于train_val.prototxt文件中填写两个data层读取图片路径。
- 用caffe/build/tools/convert_imageset生成lmdb文件。eg：caffe/create_lmdb.sh
- 用caffe/build/tools/compute_image_mean生成mean.binaryproto文件，其中第二个参数为lmdb文件路径。eg:caffe/create_binary.sh
- 生成mean.npy。eg:caffe/create_mean.py
### 命令行
cpu模式。只需要solver文件和caffemodel文件。  
caffe/build/tools/caffe train --solver caffe/models/finetune_flickr_style/solver.prototxt --weights caffe/models/finetune_flickr_style/bvlc_reference_caffenet.caffemodel   
eg:caffe/create_run.sh
### 问题  
- 一共有400张train图片和100张test图片。在train_val.phototxt文件中设置train和test的bitch_size为20，在文件solver.phototxt文件中设置test_iter=5，test_interval=4，max_iter=20。  
- 出现问题：Error in `./build/tools/caffe': double free or corruption (out): 0x0000000001342890   
按照[修改办法](http://blog.csdn.net/lien0906/article/details/46816243)make clean，无效。调节test_interval,test_iter,max_iter无用，调节train_val.phototxt文件中test的bitch_size为15,出现问题：Aborted at 1506937655 (unix time) try "date -d @1506937655" if you are using GNU date 。按照[修改办法](http://blog.csdn.net/u014696921/article/details/74989941)，无效。于是继续调节bitch_size为10，没有报错，生成caffemodel文件。  
标签从零开始，最大的标签不要超过总类数，不然不收敛。
display是设置的train的显示周期，每次test都会显示。  
最终修改所有conv层Ir_mult为0，精确率提升为1。这样没有改变原网络对图片特征的提取，只是改变了最后一层特征值加权时的权值。  
预测用caffe/newclassify.py。(myclassify.py和denny_classify.py可以作为参考)
### 参考
- [solver.phototxt文件参数含义](http://blog.csdn.net/u010417185/article/details/52182833)
- [caffe安装仅cpu](http://blog.csdn.net/u011762313/article/details/47262549)
- [caffe学习](http://www.cnblogs.com/denny402/tag/caffe/)
- [finetune_flickr_style微调](http://blog.csdn.net/liumaolincycle/article/details/48501423)
