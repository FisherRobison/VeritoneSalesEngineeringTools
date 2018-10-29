## Welcome to the Sales Engineering Resource Page
I will be updating these pages with useful Gifs, GraphQL commands, API, and platform knowledge. 

If you would like to commit to this project please reach out to me on slack. 


# CMS Tools

### Delete TDO
```
mutation deleteTDO{
  deleteTDO(id:"ID"){
    id
    message
  }
}
  ```
### Get Logs of Tasks
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
