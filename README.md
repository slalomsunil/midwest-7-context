# midwest-7-context

We are building a chat tool.  The chat tool does not send messages directly as written.  Rather it imparts a different tone of that message.  The user can select the tone e.g. funny, playful, serious.

We have two repos:
* UI: https://github.com/slalomsunil/midwest-7-ui/tree/feature/azure-integration
* Service: https://github.com/slalomsunil/midwest-7-service/tree/feature/azure-integration

We would like to build full stack features across those two repositories.  We would like the UI to understand endpoints available in Service.  Build Service endpoints if existing endpoints do not meet what is needed to complete the feature

## Key Features
* Users are able to create their profile and login just with their username
* Users are able to find any profile and are allowed to message anyone
* Chat interface supports text input
* Tone is selectable on each chat
* The message is rewritten to the recipient based on the tone selector

## Look and Feel
Mimic whatsapp look and feel

## What we are not doing
* No User authentication with passwords.  If you "login" with your username, that is enough to get you in.  If you are not recorgnized, then a profile is created for you

## Feature Workflow
* Input user story
* Generate a plan
* Iterate on plan
* Generate unit tests for planned feature
* Generate integration tests for planned fature
* Generate feature based on plan output
* Update swagger documentation on API creates/updates
* Run unit tests
* Run integration tests
