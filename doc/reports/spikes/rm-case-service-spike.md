# SPIKE REPORT

Case group currently holds two unique identifiers for a business with the business party id and sample unit ref. This is unnecessary only one is needed. Business party id is left empty for social cases.


## Case Service

### Usages of `partyId` Lombok builder method
```
givenCaseGroupStatusWhenCaseGroupStatusTransitionedThenTransitionIsAudited()
getCasesAndCaseGroups()
CaseGroupFindForExecutedCollectionExercisesSuccess()
testCaseGroupFindByCollectionExerciseAndRuRefSuccess()
testCaseGroupFindBySurveyId()
testCaseGroupFindByPartyIdSuccess()
testCaseGroupFindByIdSuccess()
findCaseGroupById()
givenCaseGroupStatusWhenCaseGroupStatusTransitionedThenTransitionIsSaved()
CaseGroupFindForExecutedCollectionExerciseReturnNullCollectionExerciseThrowsException()
CaseGroupFindForExecutedCollectionExercisesReturnNullCollectionExercisesThrowsException()
```
TOTAL: 11 (all tests)


### Usages of `getPartyId`
```
testCreateCaseAndCaseGroupWithoutChildFromMessage()
testCreateCaseAndCaseGroupWithChildFromMessage()
givenCaseGroupStatusWhenCaseGroupStatusTransitionedThenTransitionIsSaved()
givenCaseGroupStatusWhenCaseGroupStatusTransitionedThenTransitionIsAudited()
```
TOTAL: 4 (all tests)



### Usages of `setPartyId`
```
CaseService.createNewCaseGroup()
```
TOTAL: 1


### Usages of `sampleUnitRef` Lombok builder method
```
findCaseGroupById()
givenCaseGroupStatusWhenCaseGroupStatusTransitionedThenTransitionIsSaved()
givenCaseGroupStatusWhenCaseGroupStatusTransitionedThenTransitionIsAudited()
testCaseGroupFindByIdSuccess()
testCaseGroupFindByPartyIdSuccess()
testCaseGroupFindBySurveyId()
testCaseGroupFindByCollectionExerciseAndRuRefSuccess()
CaseGroupFindForExecutedCollectionExercisesSuccess()
CaseGroupFindForExecutedCollectionExercisesReturnNullCollectionExercisesThrowsException()
CaseGroupFindForExecutedCollectionExerciseReturnNullCollectionExerciseThrowsException()
```
TOTAL: 10 (all tests)


### Usages of `setSampleUnitRef`
```
CaseService.createNewCaseGroup()
```
TOTAL: 1


### Usages of `getSampleUnitRef`
```
testCreateCaseAndCaseGroupWithoutChildFromMessage()
testCreateCaseAndCaseGroupWithChildFromMessage()
CaseService.createNewCaseGroup()
```
TOTAL: 3 (2 tests)


## Action Service

### Usages of `getSampleUnitRef`
```
SampleAttributes.decorateActionRequest()
SampleUnitRefAddress.decorateActionRequest()
```
TOTAL: 3

### Usages of `setPartyId`
```
mockCaseDetailsMock()
```
TOTAL: 1 (1 test)


## Party Service

### Usages of `/casegroups/partyid/{partyId}` REST API endpoint
```
account_controller.py - request_casegroups_for_business
mocks.py - MockRequests
```
TOTAL: 2 (1 test)


## Response Operations UI

### Usages of `/casegroups/partyid/{partyId}` REST API endpoint
```
case_controller.py - get_case_groups_by_business_party_id
test_change_response_status.py - url_get_case_groups_by_business_party_id
test_reporting_units.py - url_get_casegroups_by_business_party_id
```
TOTAL: 3 (2 tests)


## CONCLUSIONS

The least impact and easiest change would be to remove the sampleUnitRef, because it doesn't change the interface contract, however the party ID is null for social cases, which seems inconsistent, although perhaps the only impact would be on reporting.

In the author's opinion, the correct thing to do is to use the RU/sampleUnitRef and get rid of the party ID. This requires changes to the REST API and the clients which use it.

The highest impact change will affect Case, Action, Party and the Response Operations UI, which would be a minimum of 4 PRs, but it seems to leave the code and data in the tidiest state eventually, although it requires more work.