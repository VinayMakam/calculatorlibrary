# Changelog

## 0.1.2 (August 9th, 2020)
### Bug fixes:
- Jira #1412 - AHSExtractor Read resume token from the mongodb.cfg if it is something other than null 
- Jira #1413 - AHSExtractor Logging requirements (additional comments)
- Jira #1452 - AHSExtractor Anonymization Errors with serial number
- Jira #1453 - AHSExtractor Anonymization needs to be considered for serialPhysical/serialLogical fields in ahs_system system array
- Jira #1462 - AHSExtractor anonymization of serialNumber in filenames should be by replace pattern
- Jira #1468 - AHSExtractor post needs to have collated data, not just the latest collection data

### Improvements:
- Referenced the InfoSight logger and Vault library files

## 0.1.1 (August 8th, 2020) 
### Bug fixes:
- Jira #1408 - Service crashes 
- Jira #1412 - AHSExtractor - Read resume token from the mongodb.cfg if it is something other than null 
- Jira #1463 - ahsextractor has hardcoded the MongoDB, should be configurable at Vault 

### Improvements:
- Integrated MONGODB details with Vault in shared profile
- Configured the constants for event size and wait time from Vault

## 0.1.0 (August, 4th, 2020) - Initial Release
### Features:
- Initial commit
- Provided basic documentation in README.md
- MongoDB change stream, watch changes on a database (Infosight_AHS)
- Advance the MongoDB change stream resume token cursor without blocking indefintely 
- Anonymize MongoDB change stream event in-flight for the selected collections: BIOS, Device inventory, Firmware, iLO events, Drivers, Events, FaultAnalysis, IML Events, On-Board Administrator, Processor, Services, USB, STAR, Storage, System, Memory, Network, SD Cards, Updates, and Power
- Combine few/more MongoDB change stream events and post to Nginx server - InfoSight data-platform
