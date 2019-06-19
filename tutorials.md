
## Table of Contents:
[WorkFlow](#workFlow)<br>


## WorkFlow:


After working with numerous customers on creating cognitive engines, it dawned on me that Flow is the perfect tool to wrap up these engines. As you will see in this walk through, cognitive engines contain a large amount of repeatable code. 


The end goal of this engine is to categorize our transcription according to the [IAB standard](https://www.iab.com/guidelines/iab-quality-assurance-guidelines-qag-taxonomy)

We’ll tackle this in five steps:

* Drag in required fields (process & ready Endpoints)
* Write code to retrieve transcript asset
* Write code to process transcript through IAB classifier
* Write code to transform to VTN Output Standard
* Update Veritone API




## Required Fields 


The following Gif references a few required HTTP nodes that start and respond to the engine being called. One is an HTTP Input Node: POST with endpoint named /process, his will allow you to consume messages from the Veritone queue and get the values needed in the rest of our flow sample input payload here:

```
{
    "applicationId": "e930828e-4786-457a-be53-cb6f189e1c00",
    "definitionId": "4c9a1602-66f1-4c5d-a1bc-754c74a78895",
    "jobId": "19052122_F4njh9Ch4Q",
    "organizationId": "16817",
    "recordingId": "500931981",
    "taskId": "19052122_F4njh9Ch4QPwWDN",
    "taskPayload": {
        "definitionId": "4c9a1602-66f1-4c5d-a1bc-754c74a78895",
        "organizationId": 16817,
        "recordingId": "500931981",
        "tdoId": "500931981"
    },
    "tdoId": "500931981",
    "token": "SAMPLE TOKEN WOULD BE HERE",
    "veritoneApiBaseUrl": "https://api.veritone.com"
}
```



You will also need to add an HTTP Input Node: GET with endpoint named /ready with an HTTP response node with a 200 status 


<img width="800" alt="portfolio_view" src="https://hold4fisher.s3.amazonaws.com/automate/Step1.gif">


Drag in a function node to consume this payload, for this specific engine we want to grab the 
token, tdoId and the taskId to be used in future functions.

<img width="800" alt="portfolio_view" src="https://hold4fisher.s3.amazonaws.com/automate/Step2.gif">


`flow.set("orgToken", msg.payload.token);
flow.set("tdoId", msg.payload.recordingId);
flow.set("taskId", msg.payload.taskId);

msg.payload = msg.payload.recordingId;
msg.orgToken = msg.payload.token; 

return msg;`



## Retrieve transcript asset

Drag in the API Node, inside this node we will paste the following graphQL query which will make a query for the transcription asset with the following code. Note, engineCategoryIds is the transcription category and tdoId will come from variables set in step 2

```js
query engineResults{
engineResults(engineCategoryIds:["67cd4dd0-2f75-445d-a6f0-2f297d6cd182"],tdoId:{{payload}}){
     sourceId
     records{
       tdoId
       assetId
       jsondata
     }
   }
}
```

## Process transcript asset against IAB

Drag in a function node to concatenate the entire json response from transcription into a complete paragraph. See `transformSentenceToParagraph()` function execution where we pass in the transcription series from the api node. I also use a helper function `getAllSentenceEnd()` to find sentence ends. 
```js
function getAllSentenceEnd (arr){
        node.warn(arr);

   let indexes = [], i;
   for (i = 0; i < arr.length; i++)
       if (arr[i].words[0].word.includes('.'))
           indexes.push(i);
   return indexes;
}
function transformSentenceToParagraph (transcriptSeries) {
   let indexes = getAllSentenceEnd(transcriptSeries);
   let sentenceArray = [];
   let base = 0;
   indexes.map((currElement, index) => {
       let arrObj = transcriptSeries.slice(base, indexes[index] + 1);
       let sentence = arrObj.map(result => result.words[0].word).join(' ');
       sentenceArray.push(sentence.replace(' .','.'));

       base = currElement + 1;
   })
 
   let paragraph = sentenceArray.slice(0, sentenceArray.length - 1).join(' ')
   node.send({payload: paragraph}) ;
}

transformSentenceToParagraph(msg.payload.engineResults.records[0].jsondata.series);
```


Drag in the function node to extract the IAB topics from this paragraph. 

```js
const extractTopics = async (paragraph) => {
 let engineOutput = [];
 let entities = await classify(paragraph);
   entities.categories.map(entity => {
       engineOutput.push({
           "startTimeMs": 'tdoStartTime',
           "stopTimeMs": 'tdoEndTime',
           "object": {
               "type": "object",
               "label": entity.label,
               "confidence": entity.score
           }
       })
   })


  console.log(engineOutput)
};
```


Next we will drag in the HTTP Request node just set the Method to POST and leave the other fields blank, The function node before this set the uri and body properties for us. 


Finally we will add a 200 Success Call Back or a 500 Service error callback

<img width="800" alt="portfolio_view" src="https://s3.amazonaws.com/hold4fisher/callback.gif">



## Creating a Library Entity:

First you will want to create the library entity using the 

```js
mutation createEntity {
					createEntity(input: {name: "${name}", libraryId: "${libraryId}"}) {
					  id
					}
	}
```


And then you will add a indentifier or image to this entity 

```js
`mutation{
  createEntityIdentifier(input:{
    entityId:"${entityId}",
    identifierTypeId:"face",
    contentType: "image/jpeg"
  }){
    id
  }
}`
```



Anywhere on the flow: Add an HTTP Input Node: GET with endpoint named /ready with an HTTP response node with a 200 status 

Next follow these steps: 

HTTP Input Node: POST with endpoint named /process
Upon startup this will receive the following payload : 
```
{
    "applicationId": "e930828e-4786-457a-be53-cb6f189e1c00",
    "definitionId": "4c9a1602-66f1-4c5d-a1bc-754c74a78895",
    "jobId": "19052122_F4njh9Ch4Q",
    "organizationId": "16817",
    "recordingId": "500931981",
    "taskId": "19052122_F4njh9Ch4QPwWDN",
    "taskPayload": {
        "definitionId": "4c9a1602-66f1-4c5d-a1bc-754c74a78895",
        "organizationId": 16817,
        "recordingId": "500931981",
        "tdoId": "500931981"
    },
    "tdoId": "500931981",
    "token": "SAMPLE TOKEN WOULD BE HERE",
    "veritoneApiBaseUrl": "https://api.veritone.com"
}
```

Function Node Tied to process endpoint which  can set variables needed in the flow such as orgToken and taskId in this example. Any data from the receive payload can also be used. 


Sub in your business logic here:


Then take output from business logic in this case the dandelion API and format to your desired VTN Output in this case: 


Get a signed uri (you will use this to put the VTN Output from above into it)
            query signedUri{
  getSignedWritableUrl{
    url
    unsignedUrl
  }
}  

Push VTN output data to this signed url. 
          


Use signed url from step above to upload engine results.
 
 mutation uploadEngineResult{
      uploadEngineResult(input: {
        taskId: "${taskId}"
        contentType: "vtn-standard"
        assetType:"vtn-standard"
        uri: "${msg.payload}"
        completeTask: true
        setTaskOutput: false
      }) {
        id
        uri
        type
      }
    }


Set status to complete:

   mutation updateTask{
  updateTask(input:{
    id:"${taskId}"
    status:complete
  }){
    id
  }
}


