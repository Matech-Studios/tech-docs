# Preface

!!! abstract "Abstract"
    This section introduces the most relevant information of the solution detailed in the playbook. (**3 min read**)

## Background

<!-- Describe briefly a minimum background required to set the reader in a proper context -->

StarvApp is an application intended to provide suitable recipes for what the user has in the fridge. Therefore, the application should be able to select the best recipes that match with the ingredients that the user has, as well as the cooking habits collected from the usage of the application. 

In order to make suitable recommendations, we decided to create a new service that interact with the existing services in the application to provide a selection of recommended recipes for the input ingredients entered by the user. The approach taken to recommend recipes is through a basic schema that uses the matching ingredients and their frecuencies in the database to rank the best recipes for a given input. Then, a Python service was developed that connects to a AMQP queue to recipe input and deliver recommendations in real time.

This playbook is intented to explain how this recipes recommender was developed, from the understanding of the problem to the validation of the solution integrated in the application. It is written for a reader that is interested in knowing how to obtain a recommender system for this context of the application, so the information will have technical details at different levels.

## Goal

<!-- What is the purpose of the solution -->

Develop a service that is able to recommend a top of recipes for the ingredients entered by the user and the set of recipes available in the database.

## Scope

<!-- What the solution will do and what it will not do -->

The service will receive the ingredients selected by the user, and a connection to the database with the available recipes, and it will return a top selection of recipes based on the attributes of recipes and ingredients. 

The services will not use the users' behavior to select the recommended recipes in this version.

## Technologies

<!-- Show the most relevant technologies used, mentioning the version if possible. You can group them into sections if you want -->

These are the most relevant technologies used during the development.

### Languages

|Name   | Version  | Description |
|---|---|---|
|Python | 3.8   | Main development |
|Bash   | 5.0   | Util scripts     |


### Data Processing

|Name   | Version  | Description |
|---|---|---|
|Pandas | 1.1  | Tabular data manipulation |
|Scikit-learn | 0.23   | Columns processing for scoring  |
|FastText | 0.9  | Classification of ingredients from text descriptions  |

### Messages broker

|Name   | Version  | Description |
|---|---|---|
|Aio-pika | 6.7  | Connection to RabbitMQ (AMQP connection) with asyncio capabilities |

### Testing 

|Name   | Version  | Description |
|---|---|---|
|Pytest | 6.2 | Unit and System testing of service |

## Team

<!-- Tag the users of team members, mentioning the adopted roles and the main responsibilities taken -->

- **Leandro Ferrado** ([@leferrad](https://github.com/leferrad)) - Data Scientist & Backend developer
    - Implement algorithm to recommend recipes
    - Data quality control of recipes stored in database
    - Develop services for recipes recommender