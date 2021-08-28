# Chapter 1: Understanding

!!! abstract "Abstract"
    This chapter covers the work done to understand the problem to solve and the context involved, in order to define the requirements for the solution. (**3 min read**)

## Context

<!-- Write all the relevant context of the business or domain, and identify all needed resources to work on the solution -->

StarvApp is an application intended to provide suitable recipes for what the user has in the fridge. Therefore, the application should be able to select the best recipes that match with the ingredients that the user has, as well as the cooking habits collected from the usage of the application. 

The solution described in this playbook is the service responsible for processing the user input (ingredients) and the available information on the App to recommend a top of recipes. 

At the moment of starting this solution, the StarvApp already has an architecture implemented with an application interacting with backend services, so the agreed scope for this work was to create a new service that helps the existing backend to solve the recipes recommendations.

In addition, a database of recipes and ingredients was implemented and checked to ensure the available fields are enough to implement a recommendations schema. No data about users behavior was available, so the scope for the recommendations was focused on matching recipes and ingredients information.

!!! info
    Data source of recipes was changed during the project due to quality issued found. See [Chapter 3](3_executiong.md) for more details. 

The stakeholders involved in this development are mainly the Product Owner of the application and the teams that develop the other services:  

- **Frontend team**, in charge of the UI and the UX of the application.  
- **Backend team**, in charge of the services of the application (endpoints, database, integrations).  
- **Analytics team** (which develops this service), in charge of extracting data analytics to enrich the information presented to the user.  
- **DevOps team**, in charge of the infrastructure where the application is deployed.  

Finally, the application will be in Spanish language to allow a deployment in Argentina. Therefore, the text information to process will be in that language.

## Problem

<!-- Identify and document the problem to solve, after some sessions of understanding with the stakeholders -->

Based on the given context of the StarvApp development, we could state that the need was to have a service that could be called by the backend to do the task of matching a top of recipes. 

In order to understand the expected matching of recipes to have in this service, a poll was conducted with the team. The results can be found [in this Confluence page](https://matech.atlassian.net/wiki/spaces/STA/pages/609157121/Recipe+recommendation+-+Approach), where we can summarize the following insights:

- Recommended recipes should require no more than the entered ingredients 
- The following aspects are not prioritized:
    - Ability to refine recipes selection.  
    - Avoiding repeated recommendations.
    - Ability to discover new recipes.
- Preference for few but accurate recommendations rather than selecting best recipes from a list through filters.
- Recommender should be fast.

!!! tip
    Ensure this poll is filled by stakeholders from different perspectives, even outside the project's team if possible, to ensure there are opinions biased by the project flow.

## Definitions

<!-- State some definitions about the expected outcome for the solution to develop -->

Based on the understanding detailed, we defined the following required aspects on the solution to develop:

- Recommendations of 20 recipes per call.
- Recipes must match the entered ingredients.
- Recommendations should use the database of recipes and ingredients.
- Recipes should not include more than entered ingredients.
- Deploy a service that communicates with Backend through RabbitMQ
- Service must provide an output in less than 3 seconds.