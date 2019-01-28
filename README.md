# Welcome to the Sales Engineering Resource Page
I will be updating these pages with useful GIFs, GraphQL commands, API tips, and general platform knowledge. 

If you would like to commit to this project please reach out to me on Slack.

## Table of Contents:
[Authentication](#authentication)<br>
[Ingestion](#ingestion)<br>
[Processing](#processing)<br>
[Retrieval](#retrieval)<br>
[Search](#search)<br>
[Folders](#folders)<br>
[Library](#library)<br>
[Org Setup](#org-setup)<br>
[Miscellaneous](#miscellaneous)
[Real Time](#real-time)


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

### Create Empty TDO (No Asset)
```
# Note: This can be used in conjunction with the "Download File and Run Engine" query in the "Processing" section.
mutation createTDO {
  createTDO(
    input: {
      startDateTime: 1548432520, 
      stopDateTime: 1548436341
    }
  ) 
  {
    id
    status
  }
}
```

## Processing:

### Run Iron Engine Job on Existing TDO
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

### Run Real-Time (RT) Engine Job on External File (using Web Stream Adapter) and Add Results to Existing TDO
```
# Note: "url" must be public; change only the second "engineId" value.  The existing TDO must be empty (no assets).
mutation runRTEngineJob {
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

### Run Engine Job with Standby Task
```
mutation createTranscriptionJobWithStandby {
    createJob(input: {
        targetId: "53796349",
        tasks: [
            {
                engineId: "transcribe-speechmatics-container-en-us",
                standbyTask: {
                    engineId: "transcribe-speechmatics-container-en-us",
                    standbyTask: {
                        engineId: "transcribe-voicebase",
                        payload: { language: "en-US", priority: "low" }
                    }
                }
            }]
    }) {
        id
        tasks {
            records {
                id
            }
        }
    }
}
```

### Download File and Run Engine (Old CMS Upload)
```
# Note: This query follows the old CMS upload process by downloading a file as an asset to an existing TDO that is empty (see "Create Empty TDO (No Asset)" query in the "Ingestion" section) and then running one or more engines against that asset.  This works with all V1F (Iron) engines.
mutation downloadFileAndRunEngine {
  createJob(
    input: {
      targetId: "331178425"
    	tasks: [{
        engineId: "download-file",
        payload: {
          tdoId: "331178425",
          startDateTime: 1548432520,
          fileUri: "https://s3.amazonaws.com/hold4fisher/s3Test.mp4"
        }},
        {
          engineId: "transcribe-speechmatics-container-en-us"
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
  }){
    id
  }
}
```

### Run Library-Enabled Engine Job (e.g. Face Recognition)
```
# Note: "libraryEngineModelId" can be obtained by running the "Get Library Training Stats" query in the "Library" section.
mutation runLibraryEngineJob {
  createJob(input: {
    targetId: "119586271"
    tasks: [
      {
        engineId: "0744df88-6274-490e-b02f-107cae03d991"
        payload: {
          libraryId: "ef8c7263-6c7c-4b8f-9cb5-93784e3d89f5"
          libraryEngineModelId: "5750ca7e-8c4a-4ca4-b9f9-df8617032bd4"
        },
      }
      ]
    }) {
    id
    targetId
    tasks {
      records {
        id
        engineId
        order
        payload
        status
      }
    }
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
      targetId
      createdDateTime
      tasks {
        records {
          id
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
            name
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
  temporalDataObject(id: "331178425") {
    tasks {
      records {       
        engine {
          id
          name
        }    
        id        
        status        
        startedDateTime
        completedDateTime
        log {
          text
          uri
        }
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

## Retrieval:

### Get Assets for TDO
```
query getAssets {
  temporalDataObject(id: "280670774") {
    assets {
      records {
        sourceData {
          engine {
            id
            name
          }
        }        
        id
        createdDateTime
        assetType
        signedUri
      }
    }
  }
}
```

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

### Get Transcription and Speaker Separation Results in Veritone Standard Format
```
query vtn {
  engineResults(tdoId: "107947027", engineIds: ["40356dac-ace5-46c7-aa02-35ef089ca9a4", "transcribe-speechmatics-container-en-us"]) {
    records {
      jsondata
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

## Search

### Search by SDO
```
query searchSDO {
    searchMedia(search: {
        index: ["mine"],
        limit: 20,
        offset: 0,
        query: {
            conditions: [
                { operator: "term", field: "QUANTITY", value: 2 }
               	{ operator: "term", field: "ORD_ID", value: "12343" }
            ],
            operator: "or"
        },
        type: "schemaId"
    })
    {
        jsondata
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

### Check if Engine is Library-Trainable
```
query engineLibraryTrainable {
  engine(id: "95d62ae8-edc2-4fb9-ad08-fe33646f0ece") {
    libraryRequired
  }
}
```

### Update Entity Thumbnail in library
```
# Obtain the URL for the entity's first identifier
query getEntityImageURL {
  entity(id: "3fd506ec-aa31-43b3-b51e-5881a57965a1") {
    identifiers(limit: 1) {
      records {
        url
      }
    }
  }
}

# Then pass the identifier's URL in as the "profileImageUrl"
mutation updateEntityThumbnail {
  updateEntity(input: {
    id: "3fd506ec-aa31-43b3-b51e-5881a57965a1", 
    profileImageUrl: "https://veritone-aiware-430032233708-us-gov-prod-sled2-recordings.s3-us-gov-west-1.amazonaws.com/3b82ec4a-0679-49cf-999b-7e76b899de56/3fd506ec-aa31-43b3-b51e-5881a57965a1/75a5f021-7c06-495a-9f03-ed5dc209dff3.jpeg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIAWIH7LETWO45I3Q7X%2F20190117%2Fus-gov-west-1%2Fs3%2Faws4_request&X-Amz-Date=20190117T232649Z&X-Amz-Expires=86400&X-Amz-Security-Token=FQoDYXdzEDIaDO0%2FlJcJeJRIyJ86myLABEvH3vewwLDedka8Cn%2FcSPfDPA8rGer8JiyVtS7rB4mbYJ8kCkTzmWYdkm9DmBVyNxcMSqxUz9sW7U1yexuERkYcpD5ajBThw2JS8uwQeL4%2FXsO3vlk9ODBjFoT%2BWn%2B7p%2Bti54npv7BPSNHk7I%2BHLMfrSQQB6CmM8RZNAjGiyh%2FWk8Ls2ykyU92lIqAZKdKYbvhNfIGgSalH%2B%2BWnrEzVeBbDKN6ahLcqCbIsqfEIChjGk8XAwvfrTmnkXGjNZWheb6rHnl%2F24chxOyX4utZVuJeJIGZh6%2BOR%2F8TU4pSGiyvTuePmcdZkczLQKLu1g3TB5WbEPR2Ip1t%2F9Xr%2Fc54ph2qBXk4dEMlFHii0DCt4Ov7HXqsQtFS0fpNY3yrg3MxVLNfx5zpsUN0jwSB%2BAWGJ3jIOCe4djCx75dejnxAEse2lxxImPZnIhi1OspqHbb6TxTUfIkNHqLrFQ3Ari%2FtRnjRJID0ocwtziGgY7Husm1lWrJMyyhY9PGv1pHtfCmlXEcNKa0VLz52vsR9sOr6zIhM20GWJ8aSakEbauKBKDUeoA43ZQMZn4p05QAuIHIVOxcJ1VwPdrSLDFgi5MJawZYMgHbjIgSLpn6AD4aaIL7dkX6EiLrKir0OEXAiyU9P33btNVMPgWIWv%2FGGF%2FGakalGhTK8%2FoTlo%2BJIZdxmfk%2FH%2F%2BeAbTtHU0yVDwgh0PIGHUmL1rg2MkOXCEN4fT8dSY99s56Axrsr%2FhYIrimid2CI0AoFzCCyXGmng3VsScUeCoCiXuYPiBQ%3D%3D&X-Amz-Signature=d31cfb938032f55c73251ead4deadfaf0ab46d8b38728f4da998b517dd207859&X-Amz-SignedHeaders=host"
  }) {
    id
    profileImageUrl
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
mutation whitelistIDentifyRedactEngines {
  addToEngineWhitelist(toAdd:{
    organizationId:    16750
    engineIds:["616f6d39-b338-4c28-b9f2-902242f93c71","34b859a1-998d-419f-8d61-47f9f1d10046","6465796c-e8fe-4df3-a083-d6c64fb2c043","8081cc99-c6c2-49b0-ab59-c5901a503508","66a83b19-f691-46b2-ba85-443fc74602ed","687c3fe5-38ef-4c9f-8ea0-960970d2cead","e5315e17-7418-490d-9048-c27376a3fe71","e924437d-e9c1-401c-bc3f-d0fccad945ff"]
  }){
    organizationId
  }
}
```

### Blacklist Deprecated Facebox Engines
```
mutation blacklistDeprecatedFaceboxEngines {
  addToEngineBlacklist(toAdd:{
    organizationId: 16750
    engineIds:["bab908d5-1eb0-4b94-9b0c-5c4bb6a81d78","dcef5300-5cc1-4fe3-bd8f-5c4d3a09b281","3f115b93-97be-46f0-b0f2-7460db15ec34","fb61580f-0bf8-4040-823d-64bada691059"]
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


## Real Time:

### Real Time Job Quick Start Guide
```
mutation CreateTDO {
  createTDO(input: {startDateTime: 1548195057, stopDateTime: 1548195065}) {
    id
  }
}

mutation createJob {
  createJob(input: {targetId: "330972585", tasks: [{engineId: "9e611ad7-2d3b-48f6-a51b-0a1ba40feab4", payload: {url: "https://s3.amazonaws.com/hold4fisher/IFC_BAVS_207_DAI.mp4"}}, {engineId: "dcef5300-5cc1-4fe3-bd8f-5c4d3a09b281", payload: {libraryId: "13e6f4a3-0d5c-4e11-9a30-913e981cb9ad", minimumConfidence: 75}}]}) {
    id
  }
}

query getEngineResults {
  engineResults(tdoId: "330972585", engineIds: ["dcef5300-5cc1-4fe3-bd8f-5c4d3a09b281"]) {
    records {
      jsondata
    }
  }
}
```