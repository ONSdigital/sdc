# SPIKE REPORT

## Case group currently holds two unique identifiers for a business with the business party id and sample unit ref. This is unnecessary only one is needed. Business party id is left empty for social cases.

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

