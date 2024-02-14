<img width="250" alt="image" src="https://github.com/anish-narain/twitter-feed/assets/69715492/ae4c6892-212c-4b61-b6ec-3f0d78a7c95f">

# Twitter Feed
Twitter Feed is an **IOT device** using sensors, **AWS**, **Node.js** and **React**. Twitter Feed is a smart bird feeder that can **seamlessly detect the presence of birds using a dedicated weight sensor**, and simultaneously **take a picture that can be stored in an interactive web app**. 
* Twitter Feed can **predict the breed of bird** from the picture to various degrees of accuracy using a trained ML model and present data from previous days in dynamic graphs.
* Twitter Feed also **tracks the weight of your bird feed** at 5-second intervals and alerts you when food is below 50g.
* Twitter Feed keeps a **history of the time of day and temperature (using a temperature sensor) of when each breed is detected**, so you can have the best chance of seeing the bird in person.
* Twitter Feed uses a secure **AWS authentication system** where each user logs in with a unique username and password (with secure communication using AWS secret keys). **Multiple users** can be registered on one Twitter Feeder, and a **single user can access multiple feeders** in different locations by entering the Bird Feeder Serial ID when accessing the web app, allowing for scalability.


## Website

The promotional website for the product can be found [here](https://riyachard.wixsite.com/twitterfeed)

[![Video Title](http://img.youtube.com/vi/lOHj0jQQYi0/0.jpg)](http://www.youtube.com/watch?v=lOHj0jQQYi0)


## Running the Code
Server code: To run the prediction server which identifies the bird breeds, run `python3 prediction-server.py` on ec2 instance with `bird_model.py` as the dependency in the `bird-recognition-server/server` folder. The main server which interfaces with the DynamoDB table, S3 bucket and React app is in the `main-server` folder. It also runs on an ec2 instance and can be run using `node server.js` 

Web-app code: To run the React application, go into the `react-app/amplifyapp/Frontend/src` folder and run `npm start`.

Client code: In `raspberry-pi`, go into `populating-dynamo-db` and run `python3 fake-upload.py` to populate the DynamoDB table systematically with data that can be used for bird trends and historical information. The `main.py` raspberry pi code used in the demo can be found in `raspberry-pi/demo`

## Main Server 
The main server handles requests from client App and filter the Database Table and send back the requested data to client App. For all features, please check the description in file `new_testing_sensor_table.js`.

## React App Frontend
We use React for constructing our WebApp and AWS Amplify for handling user authentication procedure. For all features, please check the description in files in folder `Scalable_Main_FullStack\amplifyapp\Frontend`.

## Rasberry Pi:

Contains the code running on the Rasberry Pi, there are two files:

### upload
The Actual Data Upload Pipeline is in a single super loop, with `time` function handling faster and slower procedure. We used a super loop as it had little impact on latency. The bird detection sensor is constantly checking if a bird visists while the food weight and temperature sensor send the data every 5 seconds to the server for monitoring the change in data.

We did try to separate the faster and slower features into two separate threads. However, it seems like the the two weight data sensors are communicating to the same I2C ADC chips, so the data cannot be send in parallel.

**Main** `Main.py` 

**Weight Sensor** `weightSensor.py`   
* Two weight sensors: food weight and bird detection
* Create bus pipeline with two sensors and process weight data

**Temperature Sensor** `tempSensor.py` 

### populating-dynamo-db
The Fake Data Upload Pipeline runs in two separate threads. The main thread handles bird detection in real time with weight and temperature data and another thread handles weight and temperature upload every 5 minutes.

The following random generators produce some kind of "fake real" weight and temperature data:
```python
weight_food = round(random.choices([Decimal(current_weight), 
                    Decimal(max(50.0, random.uniform(current_weight-20, current_weight)))],
                    weights=[0.05, 0.95])[0], 1)
            

temperature = round(random.choices([Decimal(current_temperature), 
                    Decimal(min(10.5, max(7.1, random.uniform(current_temperature-0.5, current_temperature+0.5))))],
                    weights=[0.6, 0.4])[0], 1)
```
We tried to run prediction locally on raspberry pi after detecting a bird and send the whole package to the database. However, the ML network is so large that the inference time for a single picture on raspberry pi takes more than 15 minutes. Thus, we used a separate server for bird detection which takes around 40 seconds to make a single prediction.

The server code structured to process the data in a queue and asynchronously running on the server so that it doesn't cause the upload procedure to wait until prediction finishes.

Good performance on Internet images, becuase the network is trained on those images, with over 95% of accuracy. **Poor performance on actual images taken from the bird feeder. Could retrained a network based on the actual images to improve performance.**

## AWS Database
1. Twitter_Table_New: stores all the data

<img width="1277" alt="image" src="https://github.com/anish-narain/twitter-feed/assets/69715492/c186ae9b-0a63-46da-bddc-04042a2ead7b">


2. Twitter_User_Table: binds user_id and bird_feeder's serial number




