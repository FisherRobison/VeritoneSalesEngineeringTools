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
