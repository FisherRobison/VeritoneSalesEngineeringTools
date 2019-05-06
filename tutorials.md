
## Table of Contents:
[WorkFlow](#workFlow)<br>


## WorkFlow:


After working with numerous customers on creating cognitive engines, it dawned on me that Flow is the perfect tool to wrap up these engines. As you will see in this walk through cognitive engines contain a large amount of repeat code. 


The end goal of this engine is to categorize our transcription according to the [IAB standard](https://www.iab.com/guidelines/iab-quality-assurance-guidelines-qag-taxonomy)


First we will drag in a v2f in node, this will allow you to consume messages from the Veritone queue and get the values needed in the rest of our flow such as tdoid (Temporal Data Object Id or sometimes referred to as recording Id). 

<img width="800" alt="portfolio_view" src="https://s3.amazonaws.com/hold4fisher/public-v2f-in.gif">





Next we will extract the following values from the event, TdoID, TaskID, JobID we will set these to flow level variables. 







Next we will drag in the API Node, inside this node we will paste the following graphQL query which will make a query for the transcription asset with the following code. Note category Id is the transcription category and tdoId will come from variables set in step 2

```js
query engineResults{
engineResults(engineCategoryIds:["67cd4dd0-2f75-445d-a6f0-2f297d6cd182"],tdoId:"412720139"){
     sourceId
     records{
       tdoId
       assetId
       jsondata
     }
   }
}
```

Next we will drag in a function node to concatenate the entire json response from transcription into a complete paragraph. See `transformSentenceToParagraph()` function execution where we pass in the transcription series from the api node. I also use a helper function `getAllSentenceEnd()` to find sentence ends. 
```js
const getAllSentenceEnd = async (arr) => {
   let indexes = [], i;
   for (i = 0; i < arr.length; i++)
       if (arr[i].words[0].word.includes('.'))
           indexes.push(i);
   return indexes;
}
const transformSentenceToParagraph = async (transcriptSeries) => {
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

   return paragraph;
}

transformSentenceToParagraph(msg.payload.engineResults);
```


Next we will drag in the function node to extract the IAB topics from this paragraph. 

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
