# midwest-7-context

We are building a social media POC to demonstrate how we can leverage AI to accellerate engineering.
We have two repos:
* UI: https://github.com/slalomsunil/midwest-7-ui/tree/feature/azure-integration
* Service: https://github.com/slalomsunil/midwest-7-service/tree/feature/azure-integration

We would like to build full stack features across those two repositories.  We would like the UI to understand endpoints available in Service.  Build Service endpoints if existing endpoints do not meet what is needed to complete the feature

## Key Features
* Users are able to create their profile and login just with their username
* Users are able to post to a feed
* Friends of users are able to see other friend's posts in their feed
* Everyone is everyone's friend by default
* Users are able to post images
* Website is styled similar to facebook, but with a more radical color scheme

## What we are not doing
* No User authentication with passwords.  If you "login" with your username, that is enough to get you in.  If you are not recorgnized, then a profile is created for you
* No persistent data store.  Use an in-memory database instead.

## Feature Workflow
* Input user story
* Generate a plan
* Iterate on plan
* Generate unit tests for planned feature
* Generate integration tests for planned fature
* Generate feature based on plan output
* Run unit tests
* Run integration tests
