# Welcome to the Sales Engineering Resource Page
I will be updating these pages with useful GIFs, GraphQL commands, API tips, and general platform knowledge. 

If you would like to commit to this project please reach out to me on Slack.

## Table of Contents:
[Authentication](#authentication)<br>
[Ingestion](#ingestion)<br>
[Processing](#processing)<br>
[Retrieval](#retrieval)<br>
[Folders](#folders)<br>
[Library](#library)<br>
[Org Setup](#org-setup)<br>
[Miscellaneous](#miscellaneous)

## Authentication:

### Log In as User and Get Session Token
```
mutation userLogin {
  userLogin(input: {userName: "jdoe@mycompany.com" password: "Password123"}) {
    token
  }
}
```

### Validate Token
```
mutation validateToken {
  validateToken(token: "78cac3d0-684d-486e-8d86-8890dbae72e2") {
    token
  }
}
```

### Log Out User Session
```
mutation userLogout {
  userLogout(token: "3c15e237-94a5-4563-9be7-4882acc7fa74")
}
```

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
# Note: The last three "engineId" values are needed for the TDO to appear correctly in CMS.
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
query getJobs {
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

### Get List of Running Jobs
```
query runningJobs {
  jobs(status: running, limit: 1000) {
    count
    records {
      id
      name
      createdDateTime
      tasks {
        records {
          id
          name
          payload
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

### Get Engine Results in Veritone Standard Format
```
query getEngineOutput {
  engineResults(tdoId: "102014611", engineIds: ["transcribe-speechmatics-container-en-us"]) {
    records {
      tdoId
      engineId
      startOffsetMs
      stopOffsetMs
      jsondata
      assetId
      userEdited
    }
  }
}
```

### Get Engine Output by Job ID
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

### Get Engine Output by TDO ID
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

### Export Transcription Results
```
mutation createExportRequest {
 createExportRequest(input: {
   includeMedia: true
   tdoData: [{tdoId: "96972470"}, {tdoId: "77041379"}]
   outputConfigurations:[
     {
       engineId:"transcribe-speechmatics"
       categoryId:"67cd4dd0-2f75-445d-a6f0-2f297d6cd182"
       formats:[
         {
           extension:"srt"
           options: {
             maxCharacterPerLine:32
             linesPerScreen: 3
             newLineOnPunctuation: true
           }
         },
         {
           extension:"vtt"
           options: {
             maxCharacterPerLine:32
             linesPerScreen: 2
             newLineOnPunctuation: false
           }
         },
         {
           extension:"ttml"
           options: {
             maxCharacterPerLine:32
             newLineOnPunctuation: true
           }
         },
         {
           extension:"txt"
           options: {
             maxCharacterPerLine:50
           }
         }
       ]
     }
   ]
 }) {
   id
   status
   organizationId
   createdDateTime
   modifiedDateTime
   requestorId
   assetUri
 }
}

# How to check the status:
query checkExportStatus {
 exportRequest(id:"10f6f809-ee36-4c12-97f4-d1d0cf04ea85") {
  status
   assetUri
   requestorId
 }
}
```

## Folders:

### List Folders in CMS and Contained TDOs
```
query listFolders {
  rootFolders(type: cms) {
    id
    name
    subfolders {
      id
      name
      childTDOs {
        count
        records {
          id
        }
      }
    }
    childTDOs {
      count
      records {
        id
      }
    }
  }
}
```

### Get Folder Info by TDO ID
```
query getFolderInfoByTDO {
  temporalDataObject(id: "112971783") {
    folders {
      id
      name
      childTDOs {
        records {
          id
        }
      }      
    }
  }
}
```

### Get Folder Info by Folder ID
```
query getFolderInfo {
  folder(id: "0ef7e0e3-63f5-47b9-8ed1-4c172c5a1e0a") {
    id
    name
    childTDOs {
      records {
        id
      }
    }    
  }
}
```

## Library:

### Get Entity Info
```
query getEntityInfo {
  entity(id: "02de18d5-4f70-450c-9716-de949668ec40") {
    name
    jsondata
  }
}
```

### Get Unpublished Entities in Library
```
query unpublishedEntities {
  entities(libraryIds: "ffd171b9-d493-41fa-86f1-4e02dd769e73", isPublished: false, limit: 1000) {
    count
    records {
      name
      id
    }
  }
}
```

### Publish/Train Library
```
mutation publishLibrary {
  publishLibrary(id: "169f5db0-1464-48b2-b5e1-59fe5c9db7d9") {
    name
  }
}
```

### Get Library Training Stats
```
# Get trainJobId value for desired engine model (e.g. "Machine Box Facebox Similarity")
query getTrainJobID {
  library(id: "1776029a-8447-406a-91bc-2402bf33443b") {
    engineModels {
      records {
        id
        createdDateTime
        modifiedDateTime
        engine {
          id
          name
        }
        libraryVersion
        trainStatus
        trainJobId
      }
    }
  }
}

# Use trainJobId value to obtain train job details
query getTrainJobDetails {
  job(id: "18104324_t5XpqlGOfm") {
    tasks {
      records {
        id
        status
        startedDateTime
        completedDateTime
      }
    }
  }
}
```

## Org Setup:

### Whitelist Engine
```
mutation whitelistEngine {
  addToEngineWhitelist(toAdd:{
    organizationId: 16994
    engineIds:["95d62ae8-edc2-4fb9-ad08-fe33646f0ece"]
  }){
    organizationId
  }
}
```

### Whitelist All Ingestion Adapter Engines
```
mutation whitelistIngestionAdapters {
  addToEngineWhitelist(toAdd:{
    organizationId: 15495
    engineIds:["d82fa1f5-f1c2-4e98-9c5f-573f8776852f",
"d1bc57fe-675d-435d-9f4d-2f074485ec55",
"829846b5-3c87-4c22-a890-e80c82cc072b",
"9e611ad7-2d3b-48f6-a51b-0a1ba40feab4",
"4d0b2407-f2af-4f19-bdfc-83f74981790a",
"b82077b1-4755-4ee1-9f40-c589ae0ba1a5",
"3bfcd954-7c0c-4456-903d-28e37caee711",
"b79d94cc-5b95-4bbd-ac01-f54b147f406f",
"9b4357b3-93bc-4886-bc62-53384eede9f5",
"447a3128-6348-4644-99d9-22401322b46b",
"61b28b11-ca2e-4825-962e-4a9312f15aa0",
"4b2dadf2-e27a-49b2-9c54-192e0b3e562a",
"ccf24d4d-e9c0-4dae-887c-d2b5f9cc33cd"
]
  }){
    organizationId
    engines{
      id
      name
    }
  }
}
```

### Whitelist IDentify and Redact Engines
```
mutation whitelistIDentifyRedactEngines{
  addToEngineWhitelist(toAdd:{
    organizationId:    16750
    engineIds:["616f6d39-b338-4c28-b9f2-902242f93c71","34b859a1-998d-419f-8d61-47f9f1d10046","6465796c-e8fe-4df3-a083-d6c64fb2c043","8081cc99-c6c2-49b0-ab59-c5901a503508","66a83b19-f691-46b2-ba85-443fc74602ed","687c3fe5-38ef-4c9f-8ea0-960970d2cead","e5315e17-7418-490d-9048-c27376a3fe71","e924437d-e9c1-401c-bc3f-d0fccad945ff"]
  }){
    organizationId
  }
}
```

## Miscellaneous:

### Get TDO Details (Filename, Tags, etc.)
```
query getTDODetails {
  temporalDataObject(id: "102014611") {   
    details
  }
}
```

### Get Duration of Media File (In Seconds)
```
query getDuration {
  temporalDataObject(id: "112971783") {
    tasks {
      records {
        id
        mediaLengthSec
      }
    }
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

### Get Organization Info for Current Session
```
query getOrgInfo {
  me {
    organization {
      name
      id
    }
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

### Fix Video Playback Issue in CMS
```
# List all TDOs and identify which ones have a bad primary asset based on "signedUri" field
query listPrimaryAsset {
  temporalDataObjects(limit: 100) {
    records {
      id
      primaryAsset(assetType:"media"){
        signedUri
      }
    }
  }
}

# Identify the right Asset ID to use as the TDO's primary asset
query primaryAsset {
  temporalDataObject(id: "117679345") {
    details
    primaryAsset(assetType: "media") {
      signedUri
      type
    }
    assets {
      records {
        id
        assetType
        signedUri
      }
    }
  }
}

# Pass in TDO ID and Asset ID to update the primary asset
mutation setPrimaryAsset {
  updateTDO(input: {id: 117679345, primaryAsset: {id: "117679345_t6l92LBhp0", assetType: "media"}}) {
    id
  }
}
```
