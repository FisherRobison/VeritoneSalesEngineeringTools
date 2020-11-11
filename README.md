# Welcome to the Sales Engineering Resource Page
I will be updating these pages with useful GraphQL commands, API tips, and general platform knowledge. 

If you would like to commit to this project please reach out to me on Slack.

## If you came here for guides check out our Tutorials Section
[All Tutorials](./tutorials.html).


## Table of Contents:
[Authentication](#authentication)<br>
[Ingestion](#ingestion)<br>
[Processing](#processing)<br>
[Retrieval](#retrieval)<br>
[Search](#search)<br>
[Folders](#folders)<br>
[Library](#library)<br>
[Structured Data](#structured-data)<br>
[Org Setup](#org-setup)<br>
[Miscellaneous](#miscellaneous)<br>
[Real Time](#real-time)<br>


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

### Run Ingestion Job "On Demand" (Specify Start and Stop Time)
```
# Note: You must first create an ingestion job in "On Demand" scheduling mode, and pass that ID into the query.
mutation runIngestionJobStartStopTime {
  launchScheduledJobs(input: {
    scheduledJobId: "46384"
    payload: {
        recordStartTime: "2019-03-11T23:30:00Z"
        recordEndTime: "2019-03-11T23:45:00Z"
    }
  }) {
    id
    targetId
  }
}
```

### Run Ingestion Job "On Demand" (Specify Duration of Recording)
```
# Note: You must first create an ingestion job in "On Demand" scheduling mode, and pass that ID into the query.
mutation runIngestionJobDuration {
  launchScheduledJobs(input: {
    scheduledJobId: "46384"
    payload: {
        recordDuration: "15m"
    }
  }) {
    id
    targetId
  }
}
```

### Create S3 Source to Monitor Bucket
```
mutation createS3SourceBucket {
  createSource(input: {
    sourceTypeId: "11"
    name: "S3"
    details:
    { 
      accessKeyId: "accessKeyValue"
      secretAccessKey: "secretAccessKeyValue"
      buckets: ["bucket-name"]
    }
  })
  {
    id
  }
}
```

### Create S3 Source to Monitor Folder in Bucket
```
mutation createS3SourceFolder {
  createSource(input: {
    sourceTypeId: "11"
    name: "S3"
    details:
    { 
      accessKeyId: "accessKeyValue"
      secretAccessKey: "secretAccessKeyValue"
      directories: ["bucket-name/folder-name/"]
    }
  })
  {
    id
  }
}
```

### Create S3 Scheduled Recurring Ingestion Job
```
# Pass in "sourceId" values from an existing S3 source (see "Create S3 Source..." queries above).
mutation createJobTemplate {
  createJobTemplate(input: {
     jobConfig: {
      createTDOInput: {
        sourceData: {
          sourceId: "62648"
        }
      }
    }
    taskTemplates: [
      # S3 Adapter:
      {
        engineId: "61b28b11-ca2e-4825-962e-4a9312f15aa0"
        payload: {
          sourceId: "62648"
        }
      },
      # Optional engines to run:
      {
	engineId: "091d48a2-7cb1-446d-b4f3-9c472ebbd2bb"
        payload: {
          definitionId: "d328a482-e615-467b-93a1-8729de6661bb"
        }
      },
      {
	engineId: "54525249-da68-4dbf-b6fe-aea9a1aefd4d"
      }      
    ],
    jobPipelineStage: 1  
  }) {
    id
    jobPipelineId
    taskTemplates {
      records {
        id
        engineId
      }
    }
  }
}

# Then pass in "jobPipelineId" value from previous mutation.  The "startDateTime" must be in GMT time zone.
mutation createScheduledJob {
  createScheduledJob (input: {
    name: "S3 Ingestion"
    runMode: Recurring
    jobPipelineIds: ["ca403edb-a98b-4911-93e5-4a0c99e4a47d"]
    startDateTime: "2019-05-30T19:20:00.000Z"
    recurringScheduleParts: [
      {
        repeatIntervalUnit: Hours
        repeatInterval: 1
      }
    ]  
  })
  {
    id
    jobPipelineIds
  }
}
```

## Processing:

### Create Textraction Job with Translation 

```
mutation newTDO {
  createTDOWithAsset(input:{  
    assetType: "media"  
    contentType: "text/plain"
    startDateTime: 1548356446
    stopDateTime:1548356447
  uri: "https://s3-veritone-voatest-or-1.s3-us-west-2.amazonaws.com/MRT_TRANSCRIPT.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIND47I4UD76OM7XQ/20190318/us-west-2/s3/aws4_request&X-Amz-Date=20190318T151421Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=10f79e998f6194e35fa3c65d086a8a48737922db1ac9750808711e7e0af381ab"}) 
  {
    id
  }
}

#Text Extraction - COMPLETED SUCCESSFULLY
mutation createJob {
  createJob(input: {
    targetId: "410318594",
    isReprocessJob:true
    tasks: [{
        engineId: "b02da568-2952-4b87-91a5-abc019c31ffa",
      payload: {
      }
    }]
  }) {
    id
  }
}

#Amazon Translate USEAST V2F
mutation createJob{
  createJob(input:{
    targetId:"410318594"
    isReprocessJob:true
    tasks:[
    {
      engineId:"c06ac5eb-3754-4055-b9fb-047b72660a0a"
      payload: {
                target: "ru"
              }
    }
  ]
  }){
    id
    status
  }
}
```

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

### Reprocess Face Detection in Redact
```
mutation reprocessFaceDetection {
   createJob(input : {
     targetId: "331580431"
     tasks:[{
       engineId: "b9eca145-3bd6-4e62-83e3-82dbc5858af1",
       payload: {
         recordingId: "331580431"
         confidenceThreshold: 0.7
         videoType: "Bodycam"
       }
     }]
   }){
     id
   }
}
```

### Run V3 Engine Job: Transcription
```
mutation runV3TranscriptionJob {
  createJob(input: {
    # Use `target` to create a new TDO with the job, or use `targetId` to reference an existing TDO
    target: {
      name: "V3 Transcription Test.mp4"
      startDateTime: "2020-07-23T19:40:04.000Z"
      stopDateTime: "2020-07-23T19:40:04.000Z"
    }
    # Default production cluster in US-PROD
    clusterId: "rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
    # Engine tasks:
    tasks: [
      {
        # Webstream Adapter (WSA)
        engineId: "9e611ad7-2d3b-48f6-a51b-0a1ba40fe255"
        payload: {
          url: "https://s3.amazonaws.com/dev-chunk-cache-tmp/AC.mp4"
        }
        ioFolders: [
          {
            referenceId: "wsaOutput"
            mode: stream
            type: output
          }
        ]
        executionPreferences: {
          priority: -5
        }
      },
      {
        # Stream Ingestor 2 (SI2) Playback engine to store playback segments
        engineId: "352556c7-de07-4d55-b33f-74b1cf237f25" 
        ioFolders: [
          {
            referenceId: "playbackInput"
            mode: stream
            type: input
          }
        ]
        executionPreferences: {
      	  parentCompleteBeforeStarting: true
          priority: -5
        }
      },
      {
        # SI2 Chunk engine to split media into chunks for processing
        engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"  
        payload: {
          ffmpegTemplate: "audio"
          chunkOverlap: 3
          customFFMPEGProperties: {
	    chunkSizeInSeconds: "60"
            chunkOverlapDuration: "3s"
            outputChunkDuration: "1m"
          }
        }
        ioFolders: [
          {
            referenceId: "chunkAudioInput"
            mode: stream
            type: input
          },
          {
            referenceId: "chunkAudioOutput"
            mode: chunk
            type: output
          }
        ]
       	executionPreferences: {
      	  parentCompleteBeforeStarting: true
          priority: -5
        }
      },
      {
        # Speechmatics English Transcription engine
        engineId: "c0e55cde-340b-44d7-bb42-2e0d65e98255"
        ioFolders: [
          {
            referenceId: "transcriptionInput"
            mode: chunk
            type: input
          },
          {
            referenceId: "transcriptionOutput"
            mode: chunk
            type: output
          }
        ]
       	executionPreferences: {
      	  maxEngines: 10
          parentCompleteBeforeStarting: true
          priority: -5
        }        
      },
      {
        # Output Writer (OW) for collating VTN-Standard output from cognitive engine
        engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
        ioFolders: [
          {
            referenceId: "owInput"
            mode: chunk
            type: input
          }
        ]
       	executionPreferences: {
          parentCompleteBeforeStarting: true
          priority: -10
        }         
      }
    ]
    # Routes : A route connects a parent output folder to a child input folder
    routes: [
      {  # WSA --> SI2 Playback
        parentIoFolderReferenceId: "wsaOutput"
        childIoFolderReferenceId: "playbackInput"
        options: {}
      },
      {  # WSA --> SI2 Chunk Audio
        parentIoFolderReferenceId: "wsaOutput"
        childIoFolderReferenceId: "chunkAudioInput"
        options: {}
      }
      {  # SI2 Chunk Audio --> Transcription
        parentIoFolderReferenceId: "chunkAudioOutput"
        childIoFolderReferenceId: "transcriptionInput"
        options: {}
      }
      {  # Transcription --> OW
        parentIoFolderReferenceId: "transcriptionOutput"
        childIoFolderReferenceId: "owInput"
        options: {}
      } 
    ]
  }) {
    id
    targetId
    clusterId    
    tasks {
      records{
        id
        engineId
        payload
        taskPayload
        status
        output
        ioFolders {
          referenceId
          type
          mode
        }
      }
    }
    routes {
      parentIoFolderReferenceId
      childIoFolderReferenceId
    }
  }
}
```

### Run V3 Engine Job: Translation
```
mutation runV3TranslationJob {
	createJob(input: {
		targetId: "1200851778"
		clusterId: "rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
		tasks: [
      {
				engineId: "95c910f6-4a26-4b66-96cb-488befa86466"
        payload: {
          url: "https://vt-maxagg-test.s3.amazonaws.com/media/spanish.txt"
        }
				ioFolders: [
        	{
						referenceId: "translationOutput"
						mode: chunk
						type: output
					}]
				executionPreferences: {
          priority: -20
				}
			}
      {
        engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"  
        ioFolders: [
          {
            referenceId: "owInput"
            mode: chunk
            type: input
          } 
        ]
				executionPreferences: {
					parentCompleteBeforeStarting: true
          priority: -21
				}        
      }      
		]
		routes: [{
        parentIoFolderReferenceId: "translationOutput",
        childIoFolderReferenceId: "owInput"
    	}    
		]
	}) {
		id
    targetId
	}
}
```

### Run V3 Engine Job: Object Detection
```
mutation runV3ObjectJob {
  createJob(input: {
    targetId: "1200851778"
    clusterId: "rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
    tasks: [
      {
        engineId: "9e611ad7-2d3b-48f6-a51b-0a1ba40fe255"
        payload: {
          url: "https://ben-veritone-testing.s3.amazonaws.com/Body+Cam+(Retail)%2C+Short.mp4"
        }
        ioFolders: [
          {
            referenceId: "wsaOutput"
            mode: stream
            type: output
          }
        ]
        executionPreferences: {
          priority: -5
        }
      },
      {
        engineId: "352556c7-de07-4d55-b33f-74b1cf237f25" 
        ioFolders: [
          {
            referenceId: "playbackInput"
            mode: stream
            type: input
          }
        ]
        executionPreferences: {
      	  parentCompleteBeforeStarting: true
          priority: -5
        }
      },
      {
        engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"  
        payload: {
          ffmpegTemplate: "frame"
        }
        ioFolders: [
          {
            referenceId: "chunkVideoInput"
            mode: stream
            type: input
          },
          {
            referenceId: "chunkVideoOutput"
            mode: chunk
            type: output
          }
        ]
       	executionPreferences: {
      	  parentCompleteBeforeStarting: true
          priority: -5
        }
      },
      {
        engineId: "d66f553d-3cef-4c5a-9b66-3e551cc48b4b"
        ioFolders: [
          {
            referenceId: "tagboxInput"
            mode: chunk
            type: input
          },
          {
            referenceId: "tagboxOutput"
            mode: chunk
            type: output
          }
        ]
       	executionPreferences: {
      	  maxEngines: 10
          parentCompleteBeforeStarting: true
          priority: -5
        }        
      },
      {
        engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
        ioFolders: [
          {
            referenceId: "owTagboxInput"
            mode: chunk
            type: input
          }
        ]
       	executionPreferences: {
          parentCompleteBeforeStarting: true
          priority: -10
        }         
      }
        ]
       	executionPreferences: {
      	  maxEngines: 10
          parentCompleteBeforeStarting: true
          priority: -5
        }        
      }
    ]
    routes: [
      { 
        parentIoFolderReferenceId: "wsaOutput"
        childIoFolderReferenceId: "playbackInput"
        options: {}
      },
      { 
        parentIoFolderReferenceId: "wsaOutput"
        childIoFolderReferenceId: "chunkVideoInput"
        options: {}
      },
      { 
        parentIoFolderReferenceId: "chunkVideoOutput"
        childIoFolderReferenceId: "tagboxInput"
        options: {}
      },
      { 
        parentIoFolderReferenceId: "tagboxOutput"
        childIoFolderReferenceId: "owTagboxInput"
        options: {}
      }    
    ]
  }) {
    id
    targetId
    clusterId    
    tasks {
      records{
        id
        engineId
        payload
        taskPayload
        status
        output
        ioFolders {
          referenceId
          type
          mode
        }
      }
    }
    routes {
      parentIoFolderReferenceId
      childIoFolderReferenceId
    }
  }
}
```

### Run V3 Engine Job: Text Recognition (OCR)
```
mutation runV3OcrJob {
  createJob(input: {
    targetId: "1200851778"
    clusterId: "rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
    tasks: [
      {
        engineId: "9e611ad7-2d3b-48f6-a51b-0a1ba40fe255"
        payload: {
          url: "https://ben-veritone-testing.s3.amazonaws.com/Body+Cam+(Retail)%2C+Short.mp4"
        }
        ioFolders: [
          {
            referenceId: "wsaOutput"
            mode: stream
            type: output
          }
        ]
        executionPreferences: {
          priority: -5
        }
      },
      {
        engineId: "352556c7-de07-4d55-b33f-74b1cf237f25" 
        ioFolders: [
          {
            referenceId: "playbackInput"
            mode: stream
            type: input
          }
        ]
        executionPreferences: {
      	  parentCompleteBeforeStarting: true
          priority: -5
        }
      },
      {
        engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"  
        payload: {
          ffmpegTemplate: "frame"
        }
        ioFolders: [
          {
            referenceId: "chunkVideoInput"
            mode: stream
            type: input
          },
          {
            referenceId: "chunkVideoOutput"
            mode: chunk
            type: output
          }
        ]
       	executionPreferences: {
      	  parentCompleteBeforeStarting: true
          priority: -5
        }
      },
      {
        engineId: "1e5b4e36-acbe-49f1-bebf-1d3a0edd6219"
        ioFolders: [
          {
            referenceId: "googleOcrInput"
            mode: chunk
            type: input
          },
          {
            referenceId: "googleOcrOutput"
            mode: chunk
            type: output
          }
        ]
       	executionPreferences: {
      	  maxEngines: 10
          parentCompleteBeforeStarting: true
          priority: -5
        }        
      },
      {
        engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
        ioFolders: [
          {
            referenceId: "owGoogleOcrInput"
            mode: chunk
            type: input
          }
        ]
       	executionPreferences: {
          parentCompleteBeforeStarting: true
          priority: -10
        }         
      }      
    ]
    routes: [
      { 
        parentIoFolderReferenceId: "wsaOutput"
        childIoFolderReferenceId: "playbackInput"
        options: {}
      },
      { 
        parentIoFolderReferenceId: "wsaOutput"
        childIoFolderReferenceId: "chunkVideoInput"
        options: {}
      },
      { 
        parentIoFolderReferenceId: "chunkVideoOutput"
        childIoFolderReferenceId: "googleOcrInput"
        options: {}
      },
      { 
        parentIoFolderReferenceId: "googleOcrOutput"
        childIoFolderReferenceId: "owGoogleOcrInput"
        options: {}
      },      
    ]
  }) {
    id
    targetId
    clusterId    
    tasks {
      records{
        id
        engineId
        payload
        taskPayload
        status
        output
        ioFolders {
          referenceId
          type
          mode
        }
      }
    }
    routes {
      parentIoFolderReferenceId
      childIoFolderReferenceId
    }
  }
}
```

### Get Jobs for TDO
```
query getJobs {
  jobs(targetId: "390287267") {
    records {
      id
      createdDateTime
      status
      tasks {        
        records {          
          id
          status
          startedDateTime
          completedDateTime  
          engine {
            id
            name
            category {
              name
            }
          }           
          payload
          taskOutput
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
  temporalDataObject(id: "390287267") {
    primaryAsset(assetType: "media") {
      id
      signedUri
    }
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


###  Query Identify TDOs by Case Folder

```
query IDentifyTDOsbyCaseFolder{
  folder(id:"59c62b7d-4e3c-44a5-9b32-b330eb0fa041"){
    name
    childTDOs{
      records{
        id
        name
      }
    }
  }
}
```


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


## Structured Data:

### Create Data Registry
```
mutation createDataRegistry {
  createDataRegistry(input: {
    name: "Vehicle"
    description: "RMS metadata pertaining to a Vehicle record type."
    source: "field deprecated"
  }) {
    id
  }
}
```

### List Data Registries
```
# Note: Partial matching supported for `name` value
query listDataRegistries {
  dataRegistries(name: "Vehicle") {
    records {
      id
      name
      description
      source
    }
  }
}
```

### Create Schema Draft (in Data Registry)
```
# Note: Use the `dataRegistryId` value from the `createDataRegistry` mutation
mutation createSchemaDraft {
  upsertSchemaDraft(input: {
    dataRegistryId: "fa2714f0-f961-4dd9-a66d-2df874d19be7"
    majorVersion: 1
    schema: {
      type: "object",
      title: "Vehicle",
      required: [
        "caseId",
        "vehicleId"
      ],
      properties: {
        caseId: {
          type: "string"
        },
        vehicleId: {
          type: "string"
        },
        licensePlateNumber: {
          type: "string"
        },
        stateOfIssue: {
          type: "string"
        },
        vehicleYear: {
          type: "string"
        },
        vehicleMake: {
          type: "string"
        },
        vehicleModel: {
          type: "string"
        },
        vehicleStyle: {
          type: "string"
        },
        vehicleColor: {
          type: "string"
        }
      },
      description: "RMS metadata pertaining to a Vehicle record type."      
    }
  }) {
    id
    majorVersion
    minorVersion
    status
    definition
  }
}
```

### Publish Schema Draft
```
# Note: Pass in the Schema ID for the `id` value
mutation publishSchemaDraft {
  updateSchemaState(input: {
    id: "1b342627-e943-4293-920a-41221636e123"
    status: published
  }) {
    id
    majorVersion
    minorVersion
    status
  }
}
```

### List Schemas
```
# Note: Partial matching supported for `name` value
query listSchemas {
  schemas(name: "Vehicle") {
    records {
      id
      createdDateTime
      majorVersion
      minorVersion
      status
      definition
      dataRegistryId
    }
  }
}
```

### Create TDO with SDO (Content Template)
```
# Note: Pass in the Schema ID for the `schemaId` value
mutation createTdoWithSdo {
  createTDO(
    input: {
      startDateTime: "2019-12-11T21:37:55.000Z"
      stopDateTime: "2019-12-11T21:37:55.000Z"
      name: "GO0005203850001"
      details: {
        name: "GO0005203850001"
        veritoneFile: {
          filename: "GO0005203850001"
        }
      }
      contentTemplates: [
        {
          schemaId: "27ef1725-2844-43f4-ab45-8df1087b2b08"
          data: {
            caseId: "520385"
            vehicleId: "412591"
            licensePlateNumber: "6XDN071"
            stateOfIssue: "CA"
            vehicleYear: "2012"
            vehicleMake: "Chevrolet"
            vehicleModel: "Tahoe"
            vehicleStyle: "CARRY-ALL (e.g. BLAZER,JEEP,BRONCO)"
            vehicleColor: "WHI"
          }
        }
      ]
    }
  )
  {
    id
    status
  }
}
```

### Get SDOs (Content Templates) in TDO
```
query getTdoContentTemplates {
  temporalDataObject(id: "490830814") {
    id
    name
    assets(assetType: "content-template") {
      records {
        id
        signedUri
        jsondata
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

### Whitelist IDentify Engines
```
mutation whitelistIDentifyEngines {
  addToEngineWhitelist(toAdd:{
    organizationId:    16750
    engineIds:["616f6d39-b338-4c28-b9f2-902242f93c71"]
  }){
    organizationId
  }
}
```

### Whitelist Redact Engines (Environment Specific)
```
# US-PROD:
mutation whitelistRedactEnginesUs {
  addToEngineWhitelist(toAdd:{
    organizationId: 16750
    engineIds: ["01bd9b24-7d09-4fb1-abd5-7db28c1a4d89","3f03e804-cab6-413f-805c-ec36b6e33f5b","e924437d-e9c1-401c-bc3f-d0fccad945ff"]
  }){
    organizationId
  }
}

# UK-PROD:
mutation whitelistRedactEnginesUk {
  addToEngineWhitelist(toAdd:{
    organizationId: 16750
    engineIds: ["34b859a1-998d-419f-8d61-47f9f1d10046","01bd9b24-7d09-4fb1-abd5-7db28c1a4d89","3f03e804-cab6-413f-805c-ec36b6e33f5b","e924437d-e9c1-401c-bc3f-d0fccad945ff"]
  }){
    organizationId
  }
}

# SLED2:
mutation whitelistRedactEnginesSled2 {
  addToEngineWhitelist(toAdd:{
    organizationId: 16750
    engineIds: ["ea0ada2a-7571-4aa5-9172-b5a7d989b041","9e611ad7-2d3b-48f6-a51b-0a1ba40feab4","54525249-da68-4dbf-b6fe-aea9a1aefd4d","34b859a1-998d-419f-8d61-47f9f1d10046","01bd9b24-7d09-4fb1-abd5-7db28c1a4d89","3f03e804-cab6-413f-805c-ec36b6e33f5b","e924437d-e9c1-401c-bc3f-d0fccad945ff"]
  }){
    organizationId
  }
}
```

### Blacklist Non-Facebox Engines
```
mutation blacklistNonFaceboxEngines {
  addToEngineBlacklist(toAdd:{
    organizationId: 16817
    engineIds:["b74d4058-90f6-453a-9636-5982e34abe0c","fa7e75da-5955-476a-b20c-28e286fdfd8e","3f115b93-97be-46f0-b0f2-7460db15ec34","bab908d5-1eb0-4b94-9b0c-5c4bb6a81d78"]
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

### Suspend User Account
```
mutation suspendUser {
  updateUserStatus(input: {
    id: "4922caa3-dadb-41b7-bbd2-17961adc6f54"
    status: suspended
  }) {
    id
    name
    status
  }
}
```

### Delete Org
```
# Note: Replace `id` with the desired Org ID.
mutation deleteOrg {
  updateOrganization(input: {
    id: "12345"
    status: "deleted"
  }) {
    id
    name
    status
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
      isPublic
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

### List Engine's Custom Fields
```
query engineCustomFields {
  engine(id: "b396fa74-83ff-4052-88a7-c37808a25673") {
    id
    fields {
      type
      name
      defaultValue
      info
      options {
        key
        value
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

### Set Parent/Child Relationship on Engine Tasks in Ingestion Job
```
# First, create an ingestion job in CMS with only the parent engine selected for processing; then run the following query to obtain the "jobTemplateId" for the ingestion job and "parentTaskId" for the parent engine.
query scheduledJob {
  scheduledJob(id: "50077") {
    jobTemplates {
      records {
        id
        taskTemplates {
          records {
            id
            engine {
              id
              name
            }
            parentTaskId
            childTaskIds
          }
        }
      }
    }
  }
}

# Then pass in the "jobTemplateId", "parentTaskId", and desired "engineId" into the following query to add the child engine as a task in the template.
mutation createChildTask {
    createTaskTemplate(input: {
        engineId: "6fa160ef-9d3d-4c87-b955-eefaed611224"
        parentTaskId: "19041511_os0BKBZVwbWGgaY"
        jobTemplateId: "19041511_os0BKBZVwb"
        payload: {
            definitionId: "516771a5-ce8d-47e0-a0e4-478b1ecbbec9"
        }
    }) {
        id
    }
}
```

### List Engine Builds
```
query engineBuilds {
  engine(id: "6fa160ef-9d3d-4c87-b955-eefaed611224") {
    builds {
      records {
        id
        name
        status
      }
    }
  }
}
```

### Get Webhook Base URL (HTTP In) for Flow Process
```
# The "workflowRuntimeId" value below is "nr" appended with the org ID.
query flowWebhookBaseUrl {
  workflowRuntime(workflowRuntimeId: "nr16817") {
    uri
    authToken
  }
}
```

### Bounce (Restart) Workflow Runtime in Org
```
# First, stop the Flow instance with this mutation, where "workflowRuntimeId" is "nr" appended with the org ID.
mutation stopWorkflow {
  stopWorkflowRuntime(workflowRuntimeId: "nr16817") {
    uri
    success
    message
  }
}

# Then start the Flow instance with this mutation:
mutation startWorkflow {
  startWorkflowRuntime(workflowRuntimeId: "nr16817", orgId: "16817") {
    uri
    message
    success
  }
}
```

### Create High Priority Job in Irvine Cluster
```
mutation createHighPriorityJob {
  createJob(input: {
    targetId: "490455759", 
    isReprocessJob: true, 
    clusterId: "wpce02-useast1-irvine-cluster-key-clients", 
    tasks: [
      {engineId: "c0e55cde-340b-44d7-bb42-2e0d65e98141", payload: {keywords:"Keyword1, Keyword2"}}
    ]}) {
    id
  }
}
```

### Upload File to Redact and Process Engines
```
# First, create an empty TDO with the appropriate name and tags.
mutation createRedactTdo {
  createTDO(input: {
    name: "Body Cam.mp4",
    details: {
      tags: [
        {
            value: "in redaction",
            redactionStatus: "Draft"
        }
      ],
      veritoneFile: {
      	filename: "Body Cam.mp4"
      }
    },
    startDateTime: "2019-11-12T03:30:10.000Z",
    stopDateTime: "2019-11-12T03:30:10.000Z",
    addToIndex: true
  }) {
    id
    name
    status
  }
}

# Next, upload the file through Webstream Adapter and call the Redact engines.
mutation runRedactEngines {
  createJob(input: {
    targetId: "9457",
    tasks: [{
      engineId: "9e611ad7-2d3b-48f6-a51b-0a1ba40feab4",
      payload: {
        url: "https://veritone-aiware-430032233708-us-gov-prod-sled2-face.s3.us-gov-west-1.amazonaws.com/48/other/2019/10/2/_/Body%20Cam-1-17-837_bf75fdaf-cc05-4b32-840f-9d591a0b4da1.mp4?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAWIH7LETWKUZSAP7K%2F20191112%2Fus-gov-west-1%2Fs3%2Faws4_request&X-Amz-Date=20191112T011717Z&X-Amz-Expires=86400&X-Amz-Signature=adb91c62de8e21d4f26b2c3f3462a3643c0f02da47a7323554923e5c91601c7d&X-Amz-SignedHeaders=host"
      }
    },
    {
      engineId: "cc34d1cd-1369-4141-a60c-e51cda00d4ec"
      payload: {
         confidenceThreshold: 0.70,
         videoType: "Bodycam",
         stepSizeDetection: 3,
         detectionRate: 3,
         maxCosineDistance: 0.5
      }      
    },
    {
      engineId: "54525249-da68-4dbf-b6fe-aea9a1aefd4d"
    }]
  }) {
    id
  }
}

# Finally, create an SDO and reference the TDO so it is visible to Redact.
mutation createRedactSdo {
  createStructuredData(input: {
    schemaId: "e8910a32-d1b7-4c44-b664-c1fdb499749f",
    data: {
      tdoId: "9457",
      status: "Draft"
    }
  }) {
    id
    data
    createdDateTime
    modifiedDateTime
  }
}
```

### Get Redact Audit Log for TDO
```
query redactAuditLog {
  auditEvents(application: "redact", terms: [{tdoId: "15680"}]){
    records{
      id
      payload
      application
    }
  }
}
```

## Real Time:

### Real Time Job Quick Start Guide
```
# Create a TDO or "Container" for step 2
mutation createTDOWithAsset {
  createTDOWithAsset(input: {startDateTime: 1548880932, updateStopDateTimeFromAsset: true, contentType: "video/mp4", assetType: "media", addToIndex: true, uri: "https://s3.amazonaws.com/hold4fisher/Manchester+United+4-3+Real+Madrid+-+UEFA+CL+2002_2003+%5BHD%5D-W7HM1RfNfS4.mp4"}) {
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


# Create a Job with amazon face rekognition celebrity
mutation createJob {
  createJob(input: {
    targetId: "This should be ID from step 1 response", 
    isReprocessJob:true
    tasks: [
      {engineId: "5e651457-e102-4d16-a8f2-5c0c34f58851"}]}) {
    id
    status
  }
}
# Get the results from engine in step 2
query getEngineResults {
  engineResults(tdoId: "This should be same targetId used in step 2", engineIds: ["e8ba2d5e-e4f2-4f57-84f3-da90cb9a0ddd"]) {
    records {
      jsondata
    }
  }
}
```

### Real Time VTT Export Request with Speaker Seperation 

```
mutation createTDOWithAsset {
  createTDOWithAsset(input: {
    startDateTime: 1554415828, 
    updateStopDateTimeFromAsset: true, 
    contentType: "video/mp4",
    assetType: "media", 
    uri: "https://s3.amazonaws.com/hold4fisher/Community+La+Biblioteca+Spanish+Rap+HD-j25tkxg5Vws.mp4"
  
  }) {
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


# # Create a Job with:
# Speechmatics - Spanish - V2F: 
# 71ab1ba9-e0b8-4215-b4f9-0fc1a1d2b44d

# Veritone Speaker Separation Engine:
# 35622f4d-cff0-4016-b63f-9d86ed60b707

mutation runEngineJob {
  createJob(
    input: {
      targetId: "431011721",
      isReprocessJob:true <--Neccessary since we used createTDOWithAsset
      tasks: [
        {
          engineId: "71ab1ba9-e0b8-4215-b4f9-0fc1a1d2b44d"
        },
        {
          engineId: "35622f4d-cff0-4016-b63f-9d86ed60b707"
        }
      ]
    }
  )
  {
    id
  }
}

# Poll for Job Status
query pollStatus{
  job(id: "${jobId}") {
    id
    status
    tasks {
      records {
        id
        status
        output
        target {
          id
        }
      }
    }
    target {
      id
    }
  }
}

# Once transcription Job is complete, Create Export Request
mutation createExportRequest {
  createExportRequest(input: {
    includeMedia: false, 
    tdoData: [{tdoId: "431011721"}
    ], 
    outputConfigurations: [
      {  
            engineId:"35622f4d-cff0-4016-b63f-9d86ed60b707",
            formats:[  

            ]
         },
      {
      engineId: "71ab1ba9-e0b8-4215-b4f9-0fc1a1d2b44d"
        formats: [
          {extension: "vtt", 
            options: {
              newLineOnPunctuation: false
            
            
            }}
        
        ]
      }]}) {
    id
    status
    organizationId
    createdDateTime
    modifiedDateTime
    requestorId
    assetUri
  }
}



# Poll for Export 
query exportRequest{
			exportRequest(id: "2f61997d-26c2-48c2-a2dc-f34fa93eecb7") {
				status
				assetUri
				requestorId
			}
		}
    
```

## Structured Data (SDO):

### Create Data Registry for Schema
```
mutation createDataRegistry {
  createDataRegistry(input: {
    name: "Vehicle"
    description: "RMS metadata pertaining to a Vehicle record type."
    source: "field deprecated"
  }) {
    id
  }
}
```

### Find Data Registries By Name
```
query listDataRegistries {
  dataRegistries(name: "Vehicle") {
    records {
      id
      name
      description
      source
    }
  }
}
```

### Create Schema Draft in Data Registry
```
# Note: Pass in the `dataRegistryId` value from the `createDataRegistry` mutation response.
mutation createSchemaDraft {
  upsertSchemaDraft(input: {
    dataRegistryId: "fa2714f0-f961-4dd9-a66d-2df874d19be7"
    majorVersion: 1
    schema: {
      type: "object",
      title: "Vehicle",
      required: [
        "caseId",
        "vehicleId"
      ],
      properties: {
        caseId: {
          type: "string"
        },
        vehicleId: {
          type: "string"
        },
        licensePlateNumber: {
          type: "string"
        },
        stateOfIssue: {
          type: "string"
        },
        vehicleYear: {
          type: "string"
        },
        vehicleMake: {
          type: "string"
        },
        vehicleModel: {
          type: "string"
        },
        vehicleStyle: {
          type: "string"
        },
        vehicleColor: {
          type: "string"
        }
      },
      description: "RMS metadata pertaining to a Vehicle record type."      
    }
  }) {
    id
    majorVersion
    minorVersion
    status
    definition
  }
}
```

### Publish Schema in Data Registry
```
# Note: Pass in the `id` value from the `createSchemaDraft` mutation response.
mutation publishSchemaDraft {
  updateSchemaState(input: {
    id: "1b342627-e943-4293-920a-41221636e123"
    status: published
  }) {
    id
    majorVersion
    minorVersion
    status
  }
}

```

### Find Schemas by Name
```
query listSchemas {
  schemas(name: "Vehicle") {
    records {
      id
      createdDateTime
      majorVersion
      minorVersion
      status
      definition
      dataRegistryId
    }
  }
}
```

### Create TDO with SDO (Content Template)
```
mutation createTdoWithSdo {
  createTDO(
    input: {
      startDateTime: "2019-12-11T21:37:55.000Z"
      stopDateTime: "2019-12-11T21:37:55.000Z"
      name: "GO0005203850001"
      details: {
        name: "GO0005203850001"
        veritoneFile: {
          filename: "GO0005203850001"
        }
      }
      contentTemplates: [
        {
          schemaId: "27ef1725-2844-43f4-ab45-8df1087b2b08"
          data: {
            caseId: "520385"
            vehicleId: "412591"
            licensePlateNumber: "6XDN071"
            stateOfIssue: "CA"
            vehicleYear: "2012"
            vehicleMake: "Chevrolet"
            vehicleModel: "Tahoe"
            vehicleStyle: "CARRY-ALL (e.g. BLAZER,JEEP,BRONCO)"
            vehicleColor: "WHI"
          }
        }
      ]
    }
  )
  {
    id
    status
  }
}
```

### Update TDO with SDO (Content Template)
```
mutation updateTdoWithSdo {
  updateTDO(
    input: {
      id: "9941"
      contentTemplates: [
        {
          schemaId: "1d01c76c-9489-4169-add6-d7eb4a3ab2a8"
          data: {
            caseId: "521023"
            vehicleId: "412591"
            licensePlateNumber: "6XDN071"
            stateOfIssue: "CA"
            vehicleYear: "2012"
            vehicleMake: "Chevrolet"
            vehicleModel: "Tahoe"
            vehicleStyle: "CARRY-ALL (e.g. BLAZER,JEEP,BRONCO)"
            vehicleColor: "WHI/"
          }
        }
      ]
    }
  )
  {
    id
    status
  }
}
```

### Get SDOs (Content Templates) in TDO
```
query getTdoContentTemplates {
  temporalDataObject(id: "490830814") {
    id
    name
    assets(assetType: "content-template") {
      records {
        id
        signedUri
        jsondata
      }
    }
  }
}
```
