# Guide
## ER project
### 1. Clone the code
```bash
git clone https://github.com/Mzhongwei/er_embedding_streaming.git
```
you will get code with the following structure:
```
|----config/
|    |------default/     # Default configuration settings
|    |------examples/    # Example configuration files
|----dataprocessing/     # All data processing methods, receive data by kafka consumer
|----dynamic_embedding/  # Core methods for dynamic embedding
|----Data_example/       # Example datasets for testing
|----utils/              # Utility functions and helper scripts
|----main.py
|----requirements.txt
|----README.md
```

### 2. Create a virtual environment, activate and install dependencies:

```bash
python3 -m venv venv
source venv/bin/activate

pip install -r requirements.txt
pip install git+https://github.com/dpkp/kafka-python.git
```

### 3. Execute the Pre-training process:

#### 3.1. Set configuration variables
- For testing, do not need to change anything
- During experiments, set your own variables. You can find some examples in `config/examples/`

> more config settings and corresponding explication can be found in files under the directory `config/default/`, but do not modify parameters in these files directly.

#### 3.2. Execute the pre-training script:

```bash
python main.py -f config/examples/config-batch.yaml
```
In termial, you'll get:
![alt text](image.png)
This may take a few seconds or minutes. 
#### 3.3. Verify the output in the following folders:
```
pipeline/graph/<output_file_name>.graphml   # xml
pipeline/embeddings/<output_file_name>.emb  # binary file
```
`output_file_name` is set in configuration file, by default, `output_file_name: fodors_zagats`

### 4. Start streaming process

#### 4.1. Set configuration

- Nothing to do when testing
- During experiments, keep in mind the following variables.
```
graph_file: pipeline/graph/<output_file_name>.graphml
embeddings_file: pipeline/embeddings/<output_file_name>.emb
kafka:
    topicid: <user_name>
    groupid: <user_name>_consumer_group
```
> ⚠️ Attention 1:  `kafka_topic_id` needs to be the same as topic ID of producer, here the producer is created by `Simulator`.

> ⚠️ Attention 2: Each user should set `topicid` by his or her own `user_name` to prevent mixing data from different producer-consumer applications when multiple applications were running at the same time

#### 4.2. Execute the kafka consumer

```bash
python main.py -f config/examples/config-stream.yaml
```
You'll get..
![alt text](image-1.png)
Kafka consumer service is waiting for the producer to send messages to broker.

#### 4.3. Run the simulator
In another terminal, run the simulator, start Kafka producer service, details in section #Simulator

Messages are received when you see the following prompts:

<img src='image-4.png' style='width:30%'>

The embedding model is being trained if you see..

![alt text](image-5.png)

#### 4.4. Stop the program

All data is processed when you see `process over!!!!` Like this:
![alt text](image-6.png)

In terminal, stop the program with ```ctrl + c```

<img src='image-7.png' style='width:60%'>

#### 4.5. Verify output
You can find similarity file in
```
pipeline/similarity/<output_file_name>.db
```
### 5. Evaluate
#### 5.1. Set config

- nothing to do when testing
- remember to modify the following variables during experiments. `Similarity_file` is the output result in step 4.5, `match_file` is the ground truth for these dataset
```
similarity_file:pipeline/similarity/<output_file_name>.db
match_file:<ground truth file>
```
#### 5.2. Run the evaluation process
Execute the following command:
```bash
python main.py -f config/examples/config-evaluation.yaml
```
Results of evaluation are shown as follows: 

<img src='image-8.png' style='width:70%'> <br><br>


> Repete 3-6 for more tests

## Simulator
### 1. Clone the code
```bash
git clone https://github.com/Mzhongwei/dataStreamSimulator.git
```
### 2. Set configuration
#### 2.1. Get config file
Config file path: `src/main/resources/application.properties.example`

Remove extension `.example`
#### 2.2. Modify the file by your own settings
Uncomment the line `# csv.file.path=<your file path>`

Replace `<your file path>` by the file path to the dataset which will be sent as streams
- For initial test, replace the `csv.file.name` with the path where you cloned er_embedding_streaming in section ER  step 1:
`csv.file.path=<rootFolder>/er_embedding_streaming/Data_example/fodors_zagats-tableB.csv`
- During experiments, pay attention to the following variables:
```
spring.kafka.producer.topic-id=<user name>
spring.kafka.producer.group-id=<user name>_producer_group
csv.file.name=<location of dataset>
```
> Make sure kafka producer ID is your own ID, which is also the same as what you set for the consumer.

> Remember to modify the file path to dataset as well

### 3. Run simulator
In the project directory, execute the following command to run the simulator:
```bash
mvn spring-boot:run
```
The simulator is running correctly if you see the following messages:
![alt text](image-2.png)