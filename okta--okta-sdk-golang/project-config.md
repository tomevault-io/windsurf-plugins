---
trigger: always_on
description: Name | Type | Description | Notes
---

# Agent

## Properties

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**Id** | Pointer to **string** | Unique identifier for the agent that&#39;s generated during installation | [optional] [readonly] 
**IsHidden** | Pointer to **bool** | Determines if an agent is hidden from the Admin Console | [optional] 
**IsLatestGAedVersion** | Pointer to **bool** | Determines if the agent is on the latest generally available version | [optional] 
**LastConnection** | Pointer to **int64** | Unix timestamp in milliseconds when the agent last connected to Okta | [optional] 
**Name** | Pointer to **string** | Agent name | [optional] 
**OperationalStatus** | Pointer to **string** | Operational status of a given agent | [optional] 
**PoolId** | Pointer to **string** | Pool ID | [optional] 
**Type** | Pointer to **string** | Agent types that are being monitored | [optional] 
**UpdateMessage** | Pointer to **string** | Status message of the agent | [optional] 
**UpdateStatus** | Pointer to **string** | Status for one agent regarding the status to auto-update that agent | [optional] 
**Version** | Pointer to **string** | Agent version number | [optional] 
**Links** | Pointer to [**LinksSelf**](LinksSelf.md) |  | [optional] 

## Methods

### NewAgent

`func NewAgent() *Agent`

NewAgent instantiates a new Agent object
This constructor will assign default values to properties that have it defined,
and makes sure properties required by API are set, but the set of arguments
will change when the set of required properties is changed

### NewAgentWithDefaults

`func NewAgentWithDefaults() *Agent`

NewAgentWithDefaults instantiates a new Agent object
This constructor will only assign default values to properties that have it defined,
but it doesn't guarantee that properties required by API are set

### GetId

`func (o *Agent) GetId() string`

GetId returns the Id field if non-nil, zero value otherwise.

### GetIdOk

`func (o *Agent) GetIdOk() (*string, bool)`

GetIdOk returns a tuple with the Id field if it's non-nil, zero value otherwise
and a boolean to check if the value has been set.

### SetId

`func (o *Agent) SetId(v string)`

SetId sets Id field to given value.

### HasId

`func (o *Agent) HasId() bool`

HasId returns a boolean if a field has been set.

### GetIsHidden

`func (o *Agent) GetIsHidden() bool`

GetIsHidden returns the IsHidden field if non-nil, zero value otherwise.

### GetIsHiddenOk

`func (o *Agent) GetIsHiddenOk() (*bool, bool)`

GetIsHiddenOk returns a tuple with the IsHidden field if it's non-nil, zero value otherwise
and a boolean to check if the value has been set.

### SetIsHidden

`func (o *Agent) SetIsHidden(v bool)`

SetIsHidden sets IsHidden field to given value.

### HasIsHidden

`func (o *Agent) HasIsHidden() bool`

HasIsHidden returns a boolean if a field has been set.

### GetIsLatestGAedVersion

`func (o *Agent) GetIsLatestGAedVersion() bool`

GetIsLatestGAedVersion returns the IsLatestGAedVersion field if non-nil, zero value otherwise.

### GetIsLatestGAedVersionOk

`func (o *Agent) GetIsLatestGAedVersionOk() (*bool, bool)`

GetIsLatestGAedVersionOk returns a tuple with the IsLatestGAedVersion field if it's non-nil, zero value otherwise
and a boolean to check if the value has been set.

### SetIsLatestGAedVersion

`func (o *Agent) SetIsLatestGAedVersion(v bool)`

SetIsLatestGAedVersion sets IsLatestGAedVersion field to given value.

### HasIsLatestGAedVersion

`func (o *Agent) HasIsLatestGAedVersion() bool`

HasIsLatestGAedVersion returns a boolean if a field has been set.

### GetLastConnection

`func (o *Agent) GetLastConnection() int64`

GetLastConnection returns the LastConnection field if non-nil, zero value otherwise.

### GetLastConnectionOk

`func (o *Agent) GetLastConnectionOk() (*int64, bool)`

GetLastConnectionOk returns a tuple with the LastConnection field if it's non-nil, zero value otherwise
and a boolean to check if the value has been set.

### SetLastConnection

`func (o *Agent) SetLastConnection(v int64)`

SetLastConnection sets LastConnection field to given value.

### HasLastConnection

`func (o *Agent) HasLastConnection() bool`

HasLastConnection returns a boolean if a field has been set.

### GetName

`func (o *Agent) GetName() string`

GetName returns the Name field if non-nil, zero value otherwise.

### GetNameOk

`func (o *Agent) GetNameOk() (*string, bool)`

GetNameOk returns a tuple with the Name field if it's non-nil, zero value otherwise
and a boolean to check if the value has been set.

### SetName

`func (o *Agent) SetName(v string)`

SetName sets Name field to given value.

### HasName

`func (o *Agent) HasName() bool`

HasName returns a boolean if a field has been set.

### GetOperationalStatus

`func (o *Agent) GetOperationalStatus() string`

GetOperationalStatus returns the OperationalStatus field if non-nil, zero value otherwise.

### GetOperationalStatusOk

`func (o *Agent) GetOperationalStatusOk() (*string, bool)`

GetOperationalStatusOk returns a tuple with the OperationalStatus field if it's non-nil, zero value otherwise
and a boolean to check if the value has been set.

### SetOperationalStatus

`func (o *Agent) SetOperationalStatus(v string)`

SetOperationalStatus sets OperationalStatus field to given value.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okta/okta-sdk-golang](https://github.com/okta/okta-sdk-golang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
