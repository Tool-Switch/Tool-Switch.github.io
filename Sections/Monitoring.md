To effectively implement and utilize the MAPE-K framework, follow these steps:

**Upload the Following Files:**

- **`monitor.py`:**
  - Description: This file monitors the relevant metrics for adaptation.
  - Guidelines: Refer to code below to extract nessesary metrics from datastorage.
  - Implementation: Define a planner object and pass the extracted metrics to it.
  

<b>Fetching last n metrics average from Elasticsearch</b>

| Metrics Name            | Purpose                                                                                        |
|-------------------------|------------------------------------------------------------------------------------------------|
| timestamp               | Indicates the date and time when the current image processing by the model was completed.                                                   |
| log_id                  | Represents the cumulative count of images processed up to the current timestamp.                  |
| confidence              | Measures the model's certainty in its predictions and the accuracy of the detected objects.                        |
| model_name              | Identifies the specific machine learning model used from the model repository for processing the current image.                          |
| cpu                     | Reflects the CPU usage or load during the processing phase.                                                 |
| detection_boxes         | Indicates the count of entities detected in the current image.                              |
| model_processing_time   | Records the duration taken by the model to process the input image.                                            |
| image_processing_time   | Captures the total time elapsed from when the image was sent as an API request until it was fully processed on the server.                                                    |
| absolute_time_from_start| Tracks the time elapsed since the initiation of the process or experiment.                                     |
| utility                 | Represents a custom-defined metric used to evaluate the performance of the system.|

    
      def fetch_past_n_metrics_average():
    
        fields = ["model_processing_time", "detection_boxes", "confidence"] 
        #monitoring only 3 metrics, other metrics can be added as per need of stakeholder.

        # Get the total count of documents in the index
        doc_count = es.count(index=index_name)["count"]

        # Calculate the number of documents to fetch
        num_docs_to_fetch = min(num_documents, doc_count)

        # Set the query to fetch the desired documents
        query = {
            "size": num_docs_to_fetch,
            "sort": [
                {"log_id": {"order": "desc"}}
            ]
        }

        # Fetch the documents from Elasticsearch
        response = es.search(index=index_name, body=query)

        # Initialize dictionaries to store the values for each field
        field_values = {field: [] for field in fields}

        # Extract the field values from the fetched documents
        for hit in response["hits"]["hits"]:
            for field in fields:
                field_value = hit["_source"][field]
                try:
                    field_value = float(field_value)
                    field_values[field].append(field_value)
                except ValueError:
                    pass

        # Calculate the mean for each field
        mean_values = {field: sum(field_values[field]) / len(field_values[field]) if field_values[field] else 0
                      for field in fields}

        # Return the mean values
        temp_dict = {}
        for field, mean_value in mean_values.items():
            temp_dict[field] = mean_value

        return [temp_dict["confidence"], temp_dict["model_processing_time"], temp_dict["detection_boxes"]]


  <b>Code to Extract model in use and input rate:</b>
     
The current model utilizes a CSV file to store input rates. This file is located within the directory named `NAIVE`. The following code outlines the process for monitoring these two metrics.

      def extract_metric(file_name):
          df = pd.read_csv(file_name, header=None)
          
          array = df.to_numpy()
          return array[0][0]

      monitor_dict = {}  # Initialize the dictionary to store monitored values
      monitor_dict["model"] = extract_metric("../model.csv")
      monitor_dict["Input_rate"] = extract_metric("../monitor.csv")

  


 - **`planner.py`:**
   - Description: This code is responsible for determining the necessity of adaptation.
   - Implementation: Develop logic within this file to plan and decide whether adaptation is required.

 - **`Analyzer.py`:**
   - Description: This code determines the result of the adaptation process.
   - Implementation: Include logic to determine the adaptation step.

 - **`Execute.py`:**
   - Description: Executes the model switch.
   - Guidelines: Refer to the code below for model switching.
   - Implementation: Integrate the necessary logic to perform the model switch.

    <details>
    <summary><b>Model switching code</b></summary>

        def switch_model(model_name):
          f = open("../model.csv", "w")
          f.write(model_name)
          f.close()
      
        def perform_action(act):
          # model switch takes place by changing the model name in the model.csv file .
          if (act == 1):
              # switch model to n
              switch_model("YOLOv5n")

          elif (act == 2):
              # switch model to s
              switch_model("YOLOv5s")

          elif (act == 3):
              # switch model to m
              switch_model("YOLOv5m")

          elif (act == 4):
              # switch model to l
              switch_model("YOLOv5l")

          elif (act == 5):
              # switch model to xl
              switch_model("YOLOv5x")

          print("Adaptation completed.")
    </details>


 - **`Knowledge.zip`:**
   - Description: A zip file containing all the knowledge files required by the MAPE-K framework for the successful generation and execution of adaptations.

**Folder Structure:**

   - Your code files are saved in a folder structure: `NAIVE/external_MAPE_K_<id>`.
   - You have the flexibility to make direct changes to any file within this specified directory.

**Note:**

Ensure that the code files adhere to the specified guidelines for seamless integration with the MAPE-K framework.