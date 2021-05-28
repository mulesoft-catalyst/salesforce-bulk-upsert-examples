# salesforce-bulk-upsert-examples

Pre-requisites to run this project

- 3 Objects in Salesforce should be setup as shown below
- testbulkdependent
- testbulkobject
- testorder
        
  ![image](https://user-images.githubusercontent.com/45771183/119894500-ae502c00-bf0a-11eb-8158-52a29be1edbd.png)
        
        - Structure of each object is as below: 
        
  ![image](https://user-images.githubusercontent.com/45771183/119894677-e6576f00-bf0a-11eb-9039-62e160c314af.png)

  ![image](https://user-images.githubusercontent.com/45771183/119894756-fe2ef300-bf0a-11eb-835b-805dd9d87f1e.png)

  ![image](https://user-images.githubusercontent.com/45771183/119894811-1272f000-bf0b-11eb-99d8-5b68245acb29.png)

  - Ensure there is data loaded into these objects (As appropriate) before starting to play around this example
          - Use initial-load-helperflow to load the data into testbulkdependent__c object (This data is loaded initial as this table acts like a dependency table for a lookup example)

  - Add Scheduler OR HTTP listener OR any other listener onto the flows you wish to test
      - **Salesforce-Bulk-Upsert-Framework.xml** _(This file consists of all the Bulk loads mechanisms from Mulesoft to Salesforce via Salesforce Bulk V1 and Bulk V2 apis)_
      
          - **bulk-v1-load-5k** : This example is to load 5K records into an object using Bulk V1 API which needs you to create a job followed by creation of batch and closing the batch.
          - **bulk-v1-load-11k-ParallelProcessing** : Currently there is 10K limit on the Bulk V1 creation of batch, hence 11K records are loaded via for-each. You could also use Mule Batch Scope as well.
          Within this example, while creation of the job, you need to specify the concurrency mode to be used at Salesforce to process the batch (Serial or Parallel). In this example, Parallel is selected which gives the faster processing of batch records.
          Please note that 10K batch size has to be configured on the For-each scope within mule and Salesforce connector doesnt seggregate the batches by itself in 10K unlike Bulk V2 API connector.
          - **bulk-v1-load-11k-SerialProcessing** : This example is same like above, but with Serial processing. Serial processing is used to ensure there are no records locking scenarios when batch is processed at Salesforce.
          - **bulk-v2-load-5k** : Bulk V2 Connector is very simple to use but takes in a CSV data as an input, but een when you have Java/JSON input you could convert it into CSV via dataweave and then have Bulk V2 Connector do the upserts. In this case see that we are doing the upserts using the streaming mechanism _(Read File > Streaming=True)_
          - **bulk-v2-load-11k** : This is same like above but if you notice , when you load more than 10K records, then Bulk V2 API of connector auto creates 2 separate jobs i.e. 10K and 1K each. 
          - bulk-v2-load-11k-with-jsoniput : This is same as above, but in this example am converting to JSON but adding a conversion to CSV within Bulk V2 connector. This is to showcase that Bulk V2 can take only CSV inputs.

_Kindly note that Bulk v2 job results do not have the request data snapshot within Salesforce, while bulk V1 shows the request data and responses for each record._


 - **Salesforce-upsert-with-lookup.xml** _(This file consists of all the upserts mechanisms from Mulesoft to Salesforce via Salesforce Bulk V1 and Bulk V2 apis involving lookups to other objects)_
    - Lookup used in these examples is with testbulkdependent__c with testorder i.e. when adding the testorder record, you need to look up "Name" from the input into testbulkdependent__c and then use the ID of that object and then add into testorder record for relationships.
        - **salesforce-upsert-with-lookupFlow-single** : This showcases a single upsert record using a lookup. Please note that if there are more than 200 upserts then you have to use Bulk V1 or Bulk V2 or else you will need to separate out the payload using for-each in 200 subsets. 
        -  **salesforce-upsert-with-lookupFlow-bulk-v1-limit-10k** : 10k records being added into the object using Bulk V1 API with lookup of all the records.
        -  **salesforce-upsert-with-lookupFlow-bulkv2** : > 10K records using Bulk V2 API using lookup. In this case, the CSV itself has the right set of column names and no conversions are needed. Hence, full streaming is utilized. If you notice the CSV file used, the lookup is via "testbulkdependent__r.Name" structure.
        -  **salesforce-upsert-with-lookupFlow-bulkv2-withJsonInput** : Similar to above, but this example will showcase how the conversion could be done for the lookups.


Please use the right V1 or V2 or regular Upsert APIs/connectors as appropriate to the use case. On a very high level adding a below go-to strategy for quick reference. 

-  >10K records --> BULK V2 Or V1_(if you could do for-each or batch scope)_ Or V1_(If you wish to have a control on Serial or Parallel processing)_
- <10K records --> Bulk V1 _(If you wish to have a control on Serial or Parallel processing)_ or Bulk V2
- <200 records --> Regular Upserts OR Bulk V1 or Bulk V2

