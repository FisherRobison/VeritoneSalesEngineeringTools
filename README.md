## Welcome to the Sales Engineering Resource Page
I will be updating these pages with useful Gifs, GraphQL commands, API Tips, and General Platform Knowledge. 

If you would like to commit to this project please reach out to me on slack. 


# Ingestion

### Create a TDO with asset
```
mutation {
    createTDOWithAsset(input: {
        startDateTime: 1521052518,
        stopDateTime: 1521052518,
        uri: "https://www.popsci.com/sites/popsci.com/files/images/2017/08/depositphotos_3979974_original.jpg"
    }) {
        id
    }
}

```
### Create a Real Time Job
```
mutation createJobOnCluster {
  createJob(input: {
    targetId: "88900861",
    tasks: [{
      engineId: "9e611ad7-2d3b-48f6-a51b-0a1ba40feab4",
      payload: {
        url: "https://s3.amazonaws.com/dev-chunk-cache-tmp/AC.mp4"
      }
    },
    {
      engineId: "38afb67a-045b-43db-96db-9080073695ab"
    },
    {
      engineId: "f99d363b-d20a-4498-b3cc-840b79ee78d9"
    }]
  }) {
    id
  }
}

```

### Run Engine Job on Existing TDO
```
mutation {
  createJob(
    input: {
      targetId: "102014611", 
      tasks: [
        {
          engineId: "8081cc99-c6c2-49b0-ab59-c5901a503508"
        }, 
        {
          engineId: "insert-into-index"
        }, 
        {
          engineId: "mention-generate"
        }
      ]
    }
  ) 
  {
    id
  }
}
```

### Get Logs for Tasks
```
query getLogs{
  temporalDataObject(id:"64953347"){
    tasks{
      records{
        log{
          text
          uri
        }
      }
    }
  }
}
```

### Delete TDO
```
mutation deleteTDO{
  deleteTDO(id:"ID"){
    id
    message
  }
}
  ```

# Processing 


# Retrieval




# Org Setup



# Misc
