.. DeepStack documentation master file, created by
   sphinx-quickstart on Sun Nov  8 22:05:48 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Object Detection
================

The object detection API locates and classifies 80 different kinds of objects in a single image. 

To use this API, you need to enable the detection API when starting DeepStack


Starting DeepStack
------------------

Run the command below as it applies to the version you have installed

.. tabs::

  .. code-tab:: bash Docker CPU

    docker run -e VISION-DETECTION=True -v localstorage:/datastore -p 80:5000 deepquestai/deepstack
  
  .. code-tab:: bash Docker GPU

    sudo docker run --gpus all -e VISION-DETECTION=True -v localstorage:/datastore -p 80:5000 deepquestai/deepstack:gpu

  .. code-tab:: bash Windows OS

    deepstack --VISION-DETECTION True --PORT 80
  
  .. code-tab:: bash NVIDIA Jetson

    sudo docker run --runtime nvidia -e VISION-DETECTION=True -p 80:5000 deepquestai/deepstack:jetpack

  .. code-tab:: bash ARM64

    sudo docker run -e VISION-DETECTION=True -v localstorage:/datastore -p 80:5000 deepquestai/deepstack:arm64

  .. code-tab:: bash ARM64 Server

    sudo docker run -e VISION-DETECTION=True -v localstorage:/datastore -p 80:5000 deepquestai/deepstack:arm64-server
  
  .. code-tab:: bash Raspberry Pi

    sudo deepstack start "VISION-DETECTION=True"



*Basic Parameters*

**-e VISION-DETECTION=True** This enables the object detection API.

**-v localstorage:/datastore** This specifies the local volume where DeepStack will store all data.

**-p 80:5000** This makes DeepStack accessible via port 80 of the machine.



**Example**

.. figure:: ../static/family-and-dog.jpg


.. tabs::

    .. code-tab:: python

        import requests

        image_data = open("test-image3.jpg","rb").read()

        response = requests.post("http://localhost:80/v1/vision/detection",files={"image":image_data}).json()

        for object in response["predictions"]:
            print(object["label"])

        print(response)
    
    .. code-tab:: js

        const request = require("request")
        const fs = require("fs")

        image_stream = fs.createReadStream("test-image3.jpg")

        var form = {"image":image_stream}

        request.post({url:"http://localhost:80/v1/vision/detection", formData:form},function(err,res,body){

            response = JSON.parse(body)
            predictions = response["predictions"]
            for(var i =0; i < predictions.length; i++){

                console.log(predictions[i]["label"])

            }

            console.log(response)
        })
    
    .. code-tab:: c#

        using System;
        using System.IO;
        using System.Net.Http;
        using System.Threading.Tasks;
        using Newtonsoft.Json;


        namespace appone
        {

        class Response {

            public bool success {get;set;}
            public Object[] predictions {get;set;}

        }

        class Object {

            public string label {get;set;}
            public float confidence {get;set;}
            public int y_min {get;set;}
            public int x_min {get;set;}
            public int y_max {get;set;}
            public int x_max {get;set;}

        }

        class App {

            static HttpClient client = new HttpClient();

            public static async Task detectFace(string image_path){

                var request = new MultipartFormDataContent();
                var image_data = File.OpenRead(image_path);
                request.Add(new StreamContent(image_data),"image",Path.GetFileName(image_path));
                var output = await client.PostAsync("http://localhost:80/v1/vision/detection",request);
                var jsonString = await output.Content.ReadAsStringAsync();
                Response response = JsonConvert.DeserializeObject<Response>(jsonString);

                foreach (var user in response.predictions){

                    Console.WriteLine(user.label);

                }

                Console.WriteLine(jsonString);

            }

            static void Main(string[] args){

                detectFace("test-image3.jpg").Wait();

            }

        }

        }


**Response**

.. code-block:: text

    dog
    person
    person
    {'predictions': [{'x_max': 819, 'x_min': 633, 'y_min': 354, 'confidence': 99, 'label': 'dog', 'y_max': 546}, {'x_max': 601, 'x_min': 440, 'y_min': 116, 'confidence': 99, 'label': 'person', 'y_max': 516}, {'x_max': 445, 'x_min': 295, 'y_min': 84, 'confidence': 99, 'label': 'person', 'y_max': 514}], 'success': True}


We can use the coordinates returned to extract the objects

.. tabs::

    .. code-tab:: python

        import requests
        from PIL import Image

        image_data = open("test-image3.jpg","rb").read()
        image = Image.open("test-image3.jpg").convert("RGB")

        response = requests.post("http://localhost:80/v1/vision/detection",files={"image":image_data}).json()
        i = 0
        for object in response["predictions"]:

            label = object["label"]
            y_max = int(object["y_max"])
            y_min = int(object["y_min"])
            x_max = int(object["x_max"])
            x_min = int(object["x_min"])
            cropped = image.crop((x_min,y_min,x_max,y_max))
            cropped.save("image{}_{}.jpg".format(i,label))

            i += 1
    
    .. code-tab:: js

        const request = require("request")
        const fs = require("fs")
        const easyimage = require("easyimage")

        image_stream = fs.createReadStream("test-image3.jpg")

        var form = {"image":image_stream}

        request.post({url:"http://localhost:80/v1/vision/detection", formData:form},function(err,res,body){

            response = JSON.parse(body)
            predictions = response["predictions"]
            for(var i =0; i < predictions.length; i++){

                pred = predictions[i]
                label = pred["label"]
                y_min = pred["y_min"]
                x_min = pred["x_min"]
                y_max = pred["y_max"]
                x_max = pred["x_max"]

                easyimage.crop(
                    {
                    src: "test-image3.jpg",
                    dst: i.toString() + "_" + label+"_.jpg",
                    x: x_min,
                    cropwidth: x_max - x_min,
                    y: y_min,
                    cropheight: y_max - y_min,
                }
            )

            }

        })
    
    .. code-tab:: c#

        using System;
        using System.IO;
        using System.Net.Http;
        using System.Threading.Tasks;
        using Newtonsoft.Json;
        using SixLabors.ImageSharp;
        using SixLabors.ImageSharp.Processing;
        using SixLabors.Primitives;

        namespace appone
        {

        class Response {

        public bool success {get;set;}
        public Object[] predictions {get;set;}

        }

        class Object {

        public string label {get;set;}
        public float confidence {get;set;}
        public int y_min {get;set;}
        public int x_min {get;set;}
        public int y_max {get;set;}
        public int x_max {get;set;}

        }

        class App {

        static HttpClient client = new HttpClient();

        public static async Task recognizeFace(string image_path){

            var request = new MultipartFormDataContent();
            var image_data = File.OpenRead(image_path);
            request.Add(new StreamContent(image_data),"image",Path.GetFileName(image_path));
            var output = await client.PostAsync("http://localhost:80/v1/vision/detection",request);
            var jsonString = await output.Content.ReadAsStringAsync();
            Response response = JsonConvert.DeserializeObject<Response>(jsonString);

            var i = 0;

            foreach (var user in response.predictions){

                var width = user.x_max - user.x_min;
                var height = user.y_max - user.y_min;

                var crop_region = new Rectangle(user.x_min,user.y_min,width,height);

                using(var image = Image.Load(image_path)){

                    image.Mutate(x => x
                    .Crop(crop_region)
                    );
                    image.Save(user.label + i.ToString() + "_.jpg");

                }

                i++;

            }

            }

            static void Main(string[] args){

                recognizeFace("test-image3.jpg").Wait();

            }

        }

        }


.. figure:: ../static/dog.jpg

.. figure:: ../static/man.jpg

.. figure:: ../static/woman.jpg


Setting Minimum Confidence
--------------------------

By default, the minimum confidence for detecting objects is 0.45. The confidence ranges between 0 and 1. If the confidence level for an object falls below the min_confidence, no object is detected.

The min_confidence parameter allows you to increase or reduce the minimum confidence.

We lower the confidence allowed below.

.. tabs::

    .. code-tab:: python

        import requests

        image_data = open("test-image3.jpg","rb").read()

        response = requests.post("http://localhost:80/v1/vision/detection",
        files={"image":image_data},data={"min_confidence":0.30}).json()
    
    .. code-tab:: js

        const request = require("request")
        const fs = require("fs")

        image_stream = fs.createReadStream("test-image3.jpg")

        var form = {"image":image_stream, "min_confidence":0.30}

        request.post({url:"http://localhost:80/v1/vision/detection", formData:form},function(err,res,body){

            response = JSON.parse(body)
            predictions = response["predictions"]

            console.log(response)
        })
        

CLASSES
-------

The following are the classes of objects DeepStack can detect in images

.. code-block:: text

    person,   bicycle,   car,   motorcycle,   airplane,
    bus,   train,   truck,   boat,   traffic light,   fire hydrant,   stop_sign,
    parking meter,   bench,   bird,   cat,   dog,   horse,   sheep,   cow,   elephant,
    bear,   zebra, giraffe,   backpack,   umbrella,   handbag,   tie,   suitcase,
    frisbee,   skis,   snowboard, sports ball,   kite,   baseball bat,   baseball glove,
    skateboard,   surfboard,   tennis racket, bottle,   wine glass,   cup,   fork,
    knife,   spoon,   bowl,   banana,   apple,   sandwich,   orange, broccoli,   carrot,
    hot dog,   pizza,   donot,   cake,   chair,   couch,   potted plant,   bed, dining table,
    toilet,   tv,   laptop,   mouse,   remote,   keyboard,   cell phone,   microwave,
    oven,   toaster,   sink,   refrigerator,   book,   clock,   vase,   scissors,   teddy bear,
    hair dryer, toothbrush.


Performance
-----------

DeepStack offers three modes allowing you to tradeoff speed for performance. During startup, you can specify performance mode to be , **High** , **Medium** and **Low**. 

The default mode is **Medium**.

You can specify a different mode during startup as seen below as seen below

.. tabs::

  .. code-tab:: bash Docker CPU

    docker run -e VISION-DETECTION=True -e MODE=High -v localstorage:/datastore -p 80:5000 deepquestai/deepstack
  
  .. code-tab:: bash Docker GPU

    sudo docker run --gpus all -e VISION-DETECTION=True -e MODE=High -v localstorage:/datastore -p 80:5000 deepquestai/deepstack:gpu

  .. code-tab:: bash Windows OS

    deepstack --VISION-DETECTION True --MODE High --PORT 80
  
  .. code-tab:: bash NVIDIA Jetson

    sudo docker run --runtime nvidia -e VISION-DETECTION=True -e MODE=High -p 80:5000 deepquestai/deepstack:jetpack



**Speed Modes are not available on the Raspberry PI Version**


.. toctree::
   :maxdepth: 2
   :caption: Contents:



* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
