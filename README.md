# DeepOSM

Detect roads and features in satellite imagery, by training neural networks with OpenStreetMap (OSM) data. The gist:

* Download a chunk of satellite imagery
* Download OSM data that shows roads/features for that area
* Generate training and evaluation data

Read below to run the code. [I am blogging my work journal too](http://trailbehind.github.io/DeepOSM/). 

Contributions are welcome. Open an issue if you want to discuss something to do, or [email me](mailto:andrew@gaiagps.com).

## Background on Data - NAIPs and OSM PBF

For training data, DeepOSM cuts tiles out of [NAIP images](http://www.fsa.usda.gov/programs-and-services/aerial-photography/imagery-programs/naip-imagery/), which provide 1 meter per pixel resolution, with RGB+infrared data bands.

For training labels, DeepOSM uses PBF extracts of OSM data, which contain features/ways in binary format, which can be munged with Python.

The [NAIPs come from a requester pays bucket on S3 set up by Mapbox](http://www.slideshare.net/AmazonWebServices/open-data-innovation-building-on-open-data-sets-for-innovative-applications), and the OSM extracts come [from geofabrik](http://download.geofabrik.de/).

## Install Requirements

### AWS Credentials

You need AWS credentials to download NAIPs from an S3 requester-pays bucket. This only costs a few cents for a bunch of images, but you need a credit card on file.

 * get your [AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY from AWS](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)

 * export them as environment variables (and maybe add to your bash or zprofile)

    export AWS_ACCESS_KEY_ID='FOO'

    export AWS_SECRET_ACCESS_KEY='BAR'

### Install Docker

First, [install a Docker Binary](https://docs.docker.com/engine/installation/).

I also needed to set my VirtualBox default memory to 12GB. libosmium needed 4GB, and the neural net needed even more. This is easy:

 * start Docker, per the install instructions
 * stop Docker
 * open VirtualBox, and increase the memory of the VM Docker made

### Run Scripts

Start Docker, then run:

```bash
make dev
```

### Download NAIP, PBF, and Analyze

Inside Docker, the following Python script will work. It will download all source data, tile it into training/test data and labels, train the neural net, and generate image and text output. The default data is three NAIPs, which gets tiled into NxNx4 bands of data (RGB-IR bands). The training labels derive from PBF files that overlap the NAIPs.

#### Dry Run, Predicts Poorly (random, about 50%)

    python src/run_analysis.py

For output, it will produce some console logs, and then JPEGs of the ways, labels, and predictions overlaid on the tiff. 

#### Full-Data Run, Predicts Well (90+% perhaps)

If you then re-run the analysis, but set the training_batches parameter to around 5000, it will be more than 90% accurate on a CPU, after running all night, (or so it seems at this point). On the second run, the data prep won't take nearly as long, but the neural net training takes much longer.

    python src/run_analysis.py --training_batches=5000

Full usge is as follows, or use -h on the command line:

 ```run_analysis.py [-h] [--tile_size TILE_SIZE]
                         [--training_batches TRAINING_BATCHES]
                         [--batch_size BATCH_SIZE] 
                         [--bands BANDS]
                         [--extract_type EXTRACT_TYPE]
                         [--cache_way_bmp CACHE_WAY_BMP]
                         [--clear_way_bmp_cache CLEAR_WAY_BMP_CACHE]
                         [--render_results RENDER_RESULTS] 
                         [--model MODEL]```

![NAIP with Ways and Predictions](https://pbs.twimg.com/media/Cg2F_tBUcAA-wHs.png)

### Jupyter Notebook

Alternately, development/research can be done via jupyter notebooks:

```bash
make notebook
```

To access the notebook via a browser on your host machine, find the IP VirtualBox is giving your default docker container by running:

```bash
docker-machine ls

NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER    ERRORS
default   *        virtualbox   Running   tcp://192.168.99.100:2376           v1.10.3
```

The notebook server is accessible via port 8888, so in this case you'd go to:
http://192.168.99.100:8888

### Readings

* [TensorFlow](https://www.tensorflow.org/) - using this for the deep learning, do multilayer, deep CNN
* [Learning to Detect Roads in High-Resolution Aerial
Images](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.232.1679&rep=rep1&type=pdf) (Hinton) 
* [Machine Learning for Aerial Image Labeling](https://www.cs.toronto.edu/~vmnih/docs/Mnih_Volodymyr_PhD_Thesis.pdf)- Minh's 2013 thesis, student of Hinton's
best/recent paper on doing this, great success with these methods
* Similar Efforts with OSM Data
    * [OSM-Crosswalk-Detection](https://github.com/geometalab/OSM-Crosswalk-Detection) - uses Keras to detect crosswalks, a class project (Fall 2015)
    * [OSM-HOT-ConvNet](https://github.com/larsroemheld/OSM-HOT-ConvNet) - attempted use for disaster response, author thinks it's only 69% accurate at pixel level (Fall 2016)
* [Parsing Natural Scenes and Natural Language
with Recursive Neural Networks (RNNs)](http://ai.stanford.edu/~ang/papers/icml11-ParsingWithRecursiveNeuralNetworks.pdf)
* Links from the Tensorflow site
    * [MNIST Data and Background](http://yann.lecun.com/exdb/mnist/)
    * all the other links to Nielsen’s book and [Colah’s blog](http://colah.github.io/posts/2015-08-Backprop/)
* Deep Background
    * [original Information Theory paper by Shannon](http://worrydream.com/refs/Shannon%20-%20A%20Mathematical%20Theory%20of%20Communication.pdf)


### Papers that Cite Hinton/Mnih, Relevant to Project

I am reviewing these papers from Google Scholar that both cite the key papers and seem relevant to the topic. 

* http://link.springer.com/chapter/10.1007/978-3-319-11752-2_17#page-1

* http://www.sciencedirect.com/science/article/pii/S0924271614001543

* http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=6602035&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D6602035

* http://www.cv-foundation.org/openaccess/content_cvpr_workshops_2015/W13/html/Paisitkriangkrai_Effective_Semantic_Pixel_2015_CVPR_paper.html

* http://www.tandfonline.com/doi/abs/10.1080/15481603.2013.802870

* https://www.computer.org/csdl/proceedings/icpr/2014/5209/00/5209d708-abs.html

* http://opticalengineering.spiedigitallibrary.org/article.aspx?articleid=1679147

* http://proceedings.spiedigitallibrary.org/proceeding.aspx?articleid=1354584

* http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.309.8565

* https://www.itc.nl/library/papers_2012/msc/gem/shaoqing.pdf

* http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=7326745&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D7326745

* http://proceedings.spiedigitallibrary.org/proceeding.aspx?articleid=2191094

* http://arxiv.org/abs/1509.03602

* http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=7112625&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D7112625

* http://www.sciencedirect.com/science/article/pii/S0924271615001690

* http://arxiv.org/abs/1405.6137

* https://www.itc.nl/external/ISPRS_WGIII4/ISPRSIII_4_Test_results/papers/Onera_2D_label_Vaih.pdf

* http://link.springer.com/chapter/10.1007/978-3-319-23528-8_33#page-1

* http://arxiv.org/abs/1508.06163

* http://www.mdpi.com/2072-4292/8/4/329

* http://arxiv.org/abs/1510.00098

* http://link.springer.com/article/10.1007/s10489-016-0762-6

* http://www.tandfonline.com/doi/abs/10.1080/01431161.2015.1054049

* http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=7393563&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D7393563

* http://www.cv-foundation.org/openaccess/content_iccv_2015/html/Mattyus_Enhancing_Road_Maps_ICCV_2015_paper.html

* http://www.cv-foundation.org/openaccess/content_iccv_2015/html/Zheng_Minimal_Solvers_for_ICCV_2015_paper.html

* http://arxiv.org/abs/1405.6136

* http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.681.1695&rep=rep1&type=pdf

* http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=7120492&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D7120492

* http://www.tandfonline.com/doi/abs/10.3846/20296991.2014.890271

* http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=7362660&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D7362660

* http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=7414402&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D7414402

* http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=6663455&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D6663455

* http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=7337372&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D7337372

* https://www.researchgate.net/profile/Moslem_Ouled_Sghaier/publication/280655680_Road_Extraction_From_Very_High_Resolution_Remote_Sensing_Optical_Images_Based_on_Texture_Analysis_and_Beamlet_Transform/links/55c0d9da08ae092e96678ff3.pdf

* http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=7159022&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D7159022

* http://www.sciencedirect.com/science/article/pii/S0303243415300283

* http://dl.acm.org/citation.cfm?id=2666389

* http://www.ijicic.org/ijicic-15-04045.pdf

### Papers - Not All that Relevant

* [Uses map data and shapes of overpasses to then detect pictures of the objects? Seems like a cool paper to read if it was free.](http://dl.acm.org/citation.cfm?id=2424336)

* [New technique for classification of sub-half-meter data into different zones](http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=6827949&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D6827949)

* [Couldn't access text, focused on usig lidar data](http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=6238909&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D6238909)

* [Proposes a way to extract network topology, and maybe this can be used as a post processor?](http://www.cv-foundation.org/openaccess/content_cvpr_2013/html/Wegner_A_Higher-Order_CRF_2013_CVPR_paper.html)



### Original Idea

This was the general idea to start, and working with TMS tiles sort of worked (see first 50 or so commits), so DeepOSM got switched to better data:

![Deep OSM Project](https://gaiagps.mybalsamiq.com/mockups/4278030.png?key=1e42f249214928d1fa7b17cf866401de0c2af867)