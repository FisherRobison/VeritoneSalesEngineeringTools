# Welcome to the Sales Engineering Resource Page
I will be updating these pages with useful GIFs, GraphQL commands, API tips, and general platform knowledge. 

If you would like to commit to this project please reach out to me on Slack.

## Table of Contents:
[Ingestion](#ingestion)<br>
[Processing](#processing)<br>
[Retrieval](#retrieval)<br>
[Org Setup](#org-setup)<br>
[Miscellaneous](#miscellaneous)

## Ingestion:

### Create TDO and Upload Asset
```
# Note: "uri" must be a public URL.
mutation createTDOWithAsset {
  createTDOWithAsset(
    input: {
      startDateTime: 1533761172, 
      stopDateTime: 1533761227, 
      contentType: "video/mp4", 
      assetType: "media", 
      addToIndex: true, 
      uri: "https://s3.amazonaws.com/hold4fisher/s3Test.mp4"
    }
  ) 
  {
    id
    status
    assets {
      records {
        id
        type
        contentType
        signedUri
      }
    }
  }
}
```

## Processing:

### Run Engine Job on Existing TDO
```
# Note: The last three "engineId" values are needed for the TDO to look correct in CMS.
mutation runEngineJob {
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
          engineId: "thumbnail-generator"
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

### Run Real-Time Engine Job on External File (using Web Stream Adapter) and Add Results to Existing TDO
```
# Note: "url" must be public; change only the second "engineId" value.
mutation createJobOnCluster {
  createJob(input: {
    targetId: "88900861",
    tasks: [{
      engineId: "9e611ad7-2d3b-48f6-a51b-0a1ba40feab4",
      payload: {
        url: "https://s3.amazonaws.com/hold4fisher/s3Test.mp4"
      }
    },
    {
      engineId: "38afb67a-045b-43db-96db-9080073695ab"
    }]
  }) {
    id
  }
}
```

### Get Jobs for TDO
```
query getJobsAndTasks {
  jobs(targetId: "102014611") {
    records {
      id
      createdDateTime
      status
      tasks {        
        records {       
          id
          status
          engine {
            id
            name
            category {
              name
            }
          }             
        }
      }
    }
  }
}
```

### Get Running Jobs
```
query runningJobs {
  jobs(status: running) {
    count
    records {
      id
      createdDateTime
      tasks {
        records {
          id
          engine {
            id
            name
            category {
              id
            }
          }
        }
      }
    }
  }
}
```

### Check Job Status
```
query jobStatus {
  job(id: "18114402_busvuCo21J") {
    status
    createdDateTime
    targetId
    tasks {
      records {
        status
        createdDateTime
        modifiedDateTime
        id
        engine {
          id
          name
          category {
            id
          }
        }
      }
    }
  }
}
```

### Get Logs for Tasks
```
query getLogs {
  temporalDataObject(id: "102014611") {
    tasks {
      records {
        log {
          text
          uri
        }
      }
    }
  }
}
```

## Retrieval:

## Get Engine Output by Job
```
query getEngineOutputByJob {
  job(id: "18083316_nNXUOSnxJH") {
    tasks {
      records {
        id
        output
        status
        engine {
          id
          name
          category {
            name
          }
        }
      }
    }
  }
}
```

## Get Engine Output by TDO
```
query getEngineOutputByTDO {
  temporalDataObject(id: "102014611") {
    tasks {
      records {
        id
        engine {
          id
          name
          category {
            name
          }
        }
        status
        output
      }
    }
  }
}
```

### Get Transcription Output in JSON Format
```
query getTranscriptJSON {
  temporalDataObject(id: "102014611") {
    primaryAsset(assetType: "transcript") {
      signedUri
      transform(transformFunction: Transcript2JSON)
    }
    details
  }
}
```

## Org Setup:

## Miscellaneous:

### Get TDO Details (Filename, Tags, etc.)
```
query getTDODetails {
  temporalDataObject(id: "102014611") {   
    details
  }
}
```

### Cancel Job in Progress
```
mutation cancelJob {
    cancelJob(id: "18114402_busvuCo21J") {
      id
      message
    }
  }
```

### Delete TDO
```
mutation deleteTDO {
  deleteTDO(id: "64953347") {
    id
    message
  }
}
```

### List Available Engines (Flat List)
```
query listEngines {
  engines(limit: 1000) {
    count
    records {
      id
      name
      category {
        id
        name
      }
      fields {
        name
        type
        max
        min
        step
        info
        label
        defaultValue
        options {
          key
          value
        }
      }
    }
  }
}
```

### List Available Engines (Grouped By Category)
```
query listEnginesByCategory {
  engineCategories {
    records {
      id
      name
      engines(limit: 1000) {
        records {
          id
          name
          fields {
            name
            options {
              value
            }
          }
        }
      }
    }
  }
}
```

### List Available Engine Categories
```
query listEngineCategories {
  engineCategories {
    records {
      id 
      name
      description
    }
  }
}
```

### List Engines For Specific Category
```
query listCategoryEngines {
  engines(category: "transcription", limit: 1000) {
    count
    records {
      id
      name
      fields {
        name
        type
        max
        min
        step
        info
        label
        defaultValue
        options {
          key
          value
        }
      }
    }
  }
}
```
