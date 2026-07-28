---
trigger: always_on
description: Information about the Agent, or other insurance representative who has responsibility for selling insurance.
---


# Agent in PropertyandCasualtyDataModel

Information about the Agent, or other insurance representative who has responsibility for selling insurance.  
  
 Latest version of the JSON entity definition is available on <a href="https://github.com/Microsoft/CDM/tree/master/schemaDocuments/FinancialServices/PropertyandCasualtyDataModel/Agent.cdm.json" target="_blank">GitHub</a>.  

## Traits

<details>
<summary>Traits for this entity are listed below.  
</summary>

**is.CDM.entityVersion**  
    <table><tr><th>Parameter</th><th>Value</th><th>Data type</th><th>Explanation</th></tr><tr><td>versionNumber</td><td>"1.1"</td><td>string</td><td>semantic version number of the entity</td></tr></table>

**is.identifiedBy**  
  names a specifc identity attribute to use with an entity  <table><tr><th>Parameter</th><th>Value</th><th>Data type</th><th>Explanation</th></tr><tr><td>attribute</td><td>[Agent/(resolvedAttributes)/agentId](#agentId)</td><td>attribute</td><td></td></tr></table>

**means.entityState**  
  the attribute represents the current state of the entity.  <table><tr><th>Parameter</th><th>Value</th><th>Data type</th><th>Explanation</th></tr><tr><td>attribute</td><td>[Agent/(resolvedAttributes)/statecode](#statecode)</td><td>attribute</td><td></td></tr></table>

**is.CDM.attributeGroup**  
  identifies standard groups of attributes in CDM entities.  <table><tr><th>Parameter</th><th>Value</th><th>Data type</th><th>Explanation</th></tr><tr><td>groupList</td><td><table><tr><th>attributeGroupReference</th></tr><tr><td>/FinancialServices/PropertyandCasualtyDataModel<br>/Agent.cdm.json/Agent/hasAttributes<br>/attributesAddedAtThisScope</td></tr></table></td><td>entity</td><td></td></tr></table>

**is.localized.describedAs**  
  Holds the list of language specific descriptive text for an object.  <table><tr><th>Parameter</th><th>Value</th><th>Data type</th><th>Explanation</th></tr><tr><td>localizedDisplayText</td><td><table><tr><th>languageTag</th><th>displayText</th></tr><tr><td>en</td><td>Information about the Agent, or other insurance representative who has responsibility for selling insurance.</td></tr></table></td><td>entity</td><td>a reference to the constant entity holding the list of localized text</td></tr></table>

**is.localized.displayedAs**  
  Holds the list of language specific display text for an object.  <table><tr><th>Parameter</th><th>Value</th><th>Data type</th><th>Explanation</th></tr><tr><td>localizedDisplayText</td><td><table><tr><th>languageTag</th><th>displayText</th></tr><tr><td>en</td><td>Agent</td></tr></table></td><td>entity</td><td>a reference to the constant entity holding the list of localized text</td></tr></table>

**is.CDS.sourceNamed**  
  the unique name that identifies this object in CDS for Applications.  <table><tr><th>Parameter</th><th>Value</th><th>Data type</th><th>Explanation</th></tr><tr><td>name</td><td>"msfsi_agent"</td><td>string</td><td></td></tr></table>

**has.entitySchemaAbstractionLevel**  
  A level of abstraction assigned to an Entity schema. Logical schema descriptions use complex dataTypes, inheritance, and entities as attributes. Resolved descriptions contain none of those things, only final trait and attribute sets are shown. A composition schema manipulates, guides or re-states parts of logical schemas to produce one resolved schema.  <table><tr><th>Parameter</th><th>Value</th><th>Data type</th><th>Explanation</th></tr><tr><td>level</td><td>"resolved"</td><td>string</td><td>Possible values: logical, composition, resolved</td></tr></table>

</details>

## Attributes

|Name|Description|First Included in Instance|
|---|---|---|
|[agentId](#agentId)|Unique identifier for entity instances.|<a href="Agent.md" target="_blank">PropertyandCasualtyDataModel/Agent</a>|
|[createdOn](#createdOn)|Date and time when the record was created.|<a href="Agent.md" target="_blank">PropertyandCasualtyDataModel/Agent</a>|
|[modifiedOn](#modifiedOn)|Date and time when the record was modified.|<a href="Agent.md" target="_blank">PropertyandCasualtyDataModel/Agent</a>|
|[statecode](#statecode)|Status of the Agent|<a href="Agent.md" target="_blank">PropertyandCasualtyDataModel/Agent</a>|
|[statuscode](#statuscode)|Reason for the status of the Agent|<a href="Agent.md" target="_blank">PropertyandCasualtyDataModel/Agent</a>|
|[importSequenceNumber](#importSequenceNumber)|Sequence number of the import that created this record.|<a href="Agent.md" target="_blank">PropertyandCasualtyDataModel/Agent</a>|
|[overriddenCreatedOn](#overriddenCreatedOn)|Date and time that the record was migrated.|<a href="Agent.md" target="_blank">PropertyandCasualtyDataModel/Agent</a>|
|[timeZoneRuleVersionNumber](#timeZoneRuleVersionNumber)|For internal use only.|<a href="Agent.md" target="_blank">PropertyandCasualtyDataModel/Agent</a>|
|[UTCConversionTimeZoneCode](#UTCConversionTimeZoneCode)|Time zone code that was in use when the record was created.|<a href="Agent.md" target="_blank">PropertyandCasualtyDataModel/Agent</a>|
|[name](#name)|Required name field.|<a href="Agent.md" target="_blank">PropertyandCasualtyDataModel/Agent</a>|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/common-data-model-and-service](https://github.com/MicrosoftDocs/common-data-model-and-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
