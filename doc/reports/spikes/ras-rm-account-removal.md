# Removing user account spike

We need to look into the different ways we can remove an account if a request is made for us to do so.
There seem to be 2 ways to do this, these are:

- Obfuscating the personal details of the account, but leaving the rest of it intact.
- Extracting the record of the account from the database, along with any other records that could/should be reasonably removed.

Below I'll detail the rough steps involved for both appraoches, as well as any other considerations that need to be made

## Obfuscating personal details

- Change email, first_name, last_name and telephone in `partysvc -> respondent`.  Also get the id for the user at this point.
- Change each enrolment to 'DISABLED' for each respondent_id that matches the id from the previous step.  This step might not be correct to do as I believe we have functionality in response-operations-ui to do this via the front door (the website) rather then the backdoor (the database)
- Change username in `auth -> user` (if using ras-rm-auth-service)
- Change email in `public -> credentials_oauthuser` (if using django-oauth service)

With obfuscation being a light touch approach to removing the record of an account, the messages that were sent between the ONS and the respondent can remain untouched as secure-message only ever dealt in uuids from party and never names.

### Things to consider

- This should be done via an endpoint in ras-party and the auth service (whichever one is being used) as opposed to a script.
- If the respondents enrolments aren't disabled then emails will be sent to the new obfuscated email address.  We would either need to code for this (probably bad, edge cases
 and special strings generally lead to an unmaintainable system down the line) or just send bad emails.  Both options seem bad so ensuring the enrolment is disabled first would be important.
- Should we have a special designated name for these obfuscated accounts like 'Deleted User' being first and last names? What about the email address?
- Obfuscating instead of removing can mess up counts for certain things (response chasing for example).
- We would need a way to inform internal users that a respondent isn't in the system anymore (i.e., when a conversation is opened, the internal user should be notified that the respondent doesn't exist anymore)
- There can be a number of side effects that we don't know about until we attempt it

## Deleting respondent entirely

I'll go service by service that could be affected by the account removal.  Each service will include what needs to be done to remove it, and what impact removing the account could have. The sample, action, collectionexercise services all have the url to the party service in their configuration.  We'll need to double check that a search that returns nothing doesn't break anything because of a deleted account doesn't break anything.

### ras-party

From the steps below, it's possible to unpick an account but the order is important because of foreign/composite/primary keys.  Once the order is determined then it should be possible to automate it.  This order looks roughly like:

- Identify which respondent needs to be removed in `partysvc -> respondent`.  Make a note of the id, party_uuid and email address.
- In the `party_svc -> business_respondent` table, identify all of the `business_id`s that have a respondent_id that is the same as the id obtained in the previous step.

- Remove all of the records from `partysvc -> enrolment` where the respondent_id matches the id (from the first step).  There is a composite key of business_id and respondent_id between enrolment and business_respondent tables.
- Remove all of the records with the id from `partysvc -> business_respondent` -> respondent_id. The 'id' of the respondant is a foreign key of respondent id in the business_respondant table.
- Remove from `partysvc -> respondent` by email address.

### ras-rm-auth-service ~~and django-oauth2-test~~

Doing the above will make it so that the user cannot log in.  To remove the record of the user from the system, you also have to do the following:

- Remove from `auth -> user` by email address (if using ras-rm-auth-service)
- ~~Remove from `public -> credentials_oauthuser` by email address (if using django-oauth service)~~ django-oauth-service has been switched off

### ras-collection-exercise

This service shouldn't have any issues if an account gets deleted. It goes to the party service for a business id.  If this has been deleted because of the previous step then it will fail.  Failure (from what I can tell) means that ultimately a SEFT survey (or collection instrument) can't be uploaded.  This is probably fine as a business would need a respondent with an account in order to be able to upload one, if there was only 1 and we deleted it, then they can't get to that screen.  If they have multiple respondents with accounts (is this possible?) then there will be a record in the business table for the other respondent so it should succeed.

- (Added PR comment about multiple respondents: Yep, it's possible that a several respondents can be enrolled onto the same survey for the same business. You're right in that it should have no effect on collex - if the business has other accounts then one of those can log in and complete a survey.)
- (Added PR comment: Deleting a user's account shouldn't result in failure in a sense - if a business doesn't have an active account then they won't be able to download a SEFT instrument in the first place.)

### rm-sample-service

Posts to the /parties endpoint.  This endpoint returns a business so this shouldn't be an issue as I think this all happens before the users account gets created at the start of the process.

### rm-action-service

This service goes to the party service to get information.  If there is never an action involving an account that's been deleted then it should be fine.  Need to find out a few things:

- What happens if it can't find the party?
- What provides the action service with stuff to do?  If we can find that out then we can see if it would ever do an action with a party that doesn't exist any more. (Added PR comment: It should always find the business because we should never remove the business details - it's the business that legally have to complete. There are action rules which say for each action (go live date, reminder etc) that if there are any active enrolments then send an email to those, if not then send a letter.)

### rm-reporting

From the looks of things, this should be unaffected.  If the party_id isn't found in the search, then it'll just come back with nothing.  Whether or not that's the desired behaviour is another question though.  This might link in with what in in the below section about whether or not we need to make sure the respondent has completed all the surveys they need to. (Added PR comment: This should be fine. If all enrolments are removed from the respondent then the search will treat as a business that has nobody enrolled to complete the surveys (like a lot of newly selected companies are))

### ras-secure-message

We should ensure that all the open conversations that the user has are closed to prevent the chance of an internal user attempting to continue a conversation with someone who doesn't have an account anymore.  We might need to add some code to check whether or not the user exists when trying to reopen a conversation and stop it happening if the party_id cannot be found.  An appropriate error message would then have to be displayed to the internal user.

Fortunately, when a party_id cannot be found for an external user, we return 'Unavailable' for the username, so deleting a user shouldn't break anything on the display front.

#### Things to consider

- Is it okay that any messages that were sent while the user had an account are still there?  If not, there is only one schema to look at (securemessage) that has 4 tables.   From the party_id gathered at the start, it's possible to delete the messages, the status states and finally the record of the thread in the conversations table.
- Do we need to ensure the respondent with the account has completed all of their surveys?  I imagine so.  How would we do that? (Added PR comment: The responsibility for completing is down to the business, not the individual user. If the only user enrolled on a survey is removed then the business should go back to receiving paper enrolment letters and reminders.)
- Script or inbuilt functionality in response-operations-ui? I would recommend to creating the feature in response-operations-ui to take as many manual steps out of the process as possible.
 Endpoints on the ras-party and auth service API should be created to do this programatically.  Doing it this way should lead to less mistakes, easier rollbacks if a mistake does occur and clearer logging output to help debug any issues.
- There's a metadata column in caseevent that records the partyId of the user that completed the survey. Will response-ops fall over of this partyid no longer exists? This should be removed.
