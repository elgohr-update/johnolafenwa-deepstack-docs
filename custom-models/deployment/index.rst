.. DeepStack documentation master file, created by
   sphinx-quickstart on Sun Nov  8 22:05:48 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Deploying Your Model With DeepStack
====================================
Deploying your model to DeepStack is the simplest part, once you have downloaded the best.pth file from your training.

Create a directory on your system to store your models, here we shall assume your folder is named ```my-models```
Put your best.pth file in the folder and rename it to whatever you want it to be, here we assume you named it catsanddogs.pt

Run DeepStack
=============

Starting DeepStack
------------------

Run the command below as it applies to the version you have installed

.. tabs::

  .. code-tab:: bash Docker CPU

    sudo docker run -v /path-to/custom-models-folder:/modelstore/detection -p 80:5000 deepquestai/deepstack
  
  .. code-tab:: bash Docker GPU

    sudo docker run --gpus all -v /path-to/custom-models-folder:/modelstore/detection -p 80:5000 deepquestai/deepstack:gpu

  .. code-tab:: bash Windows OS

    deepstack --MODELSTORE-DETECTION "C:/path-to-custom-models-folder" --PORT 80
  
  .. code-tab:: bash NVIDIA Jetson

    sudo docker run --runtime nvidia -v /path-to/custom-models-folder:/modelstore/detection -p 80:5000 deepquestai/deepstack:jetpack

  .. code-tab:: bash ARM64

    sudo docker run -v /path-to/custom-models-folder:/modelstore/detection -p 80:5000 deepquestai/deepstack:arm64

  .. code-tab:: bash ARM64 Server

    sudo docker run -v /path-to/custom-models-folder:/modelstore/detection -p 80:5000 deepquestai/deepstack:arm64-server


*Basic Parameters*

**-v /path-to/custom-models-folder:/modelstore/detection** This specifies the local directory where you stored your custom models

**-p 80:5000** This makes DeepStack accessible via port 80 of the machine.

Run Inference
=============

.. tabs::

    .. code-tab:: python

        import requests

        image_data = open("test-image.jpg","rb").read()

        response = requests.post("http://localhost:80/v1/vision/custom/catsanddogs",files={"image":image_data}).json()

        for object in response["predictions"]:
            print(object["label"])

        print(response)
    
    .. code-tab:: js

        const request = require("request")
        const fs = require("fs")

        image_stream = fs.createReadStream("test-image.jpg")

        var form = {"image":image_stream}

        request.post({url:"http://localhost:80/v1/vision/custom/catsanddogs", formData:form},function(err,res,body){

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
                var output = await client.PostAsync("http://localhost:80/v1/vision/custom/catsanddogs",request);
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
