# Chapter 3: Execution

!!! abstract "Abstract"
    This chapter covers the execution of the work based on the previous definitions done, in terms of developing, testing, and validating the solutions based on the agreed acceptance criteria. (**10 min read**)

## Initial analysis

Once the repository was prepared, and the initial requirements were defined, we should start from the analysis of the available data to check the following aspects:
- Available information to be used for the recommender schema
- Quality of data, and corrections to be done
- Pre-processing steps required to use the data

The first data source implemented was [Spoonacular](https://spoonacular.com/) through a paid subscription to [RapidAPI](https://rapidapi.com/). Since the involved recipes were in English, a translator service was implemented as well to have Spanish data. 

In order to ensure the database of recipes and ingredients as proper quality to implement the given approach, a Data Quality Control was conducted. Check the [How-to: Data Quality Control](how-to/data_quality_control.md) document for details. Through [this notebook](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/notebooks/20210130_data_quality_control.ipynb), the analysis was done where in general aspects the data observed was fine. However, at that point and in later iterations there were data issues found like:
- Thousands of ingredients, making hard to achieve proper matching rules for the recommender (check [these issues identified](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/develop/notebooks/ingredients_issues.yml
))
- Bad translations of the implemented service (also noticed in the previous issues identified)
- Missing information in some fields, especially for instructions of recipes

Therefore, the data sources was marked as **not reliable**, and an extra work was required to identify a new source and scrape recipes data from it. 

## New data source

After reviewing several potential data sources (having fields required by the StarvApp, and without legal restrictions), a new one was found. Therefore, a [scraper script was developed](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/scripts/scraper_plasticwebs.py) to get datasets from there with [some defined configuration](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/scripts/plasticwebs_config.yaml). Here some **important limitations** to know about the scraper:
- Execution must be done manually, with a proper update of the YAML configuration mentioned
- Paths to resulting datasets must be updated in the code as well

The resulting datasets in CSV format were controlled in a new data quality check conducted in [this notebook](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/notebooks/20210630_data_quality_control.ipynb), and then shared to the Backend team to a proper upload to the Database of the StarvApp. In addition, the snapshots of datasets obtained were stored in One Drive for availability.
 
!!! info
    This data extraction process was part of a **workaround task** handled after finding data issues in Spoonacular. Later work about automation of data extraction from these sources will be required in further steps.

It is important to mention that ingredients in recipes extracted from this source were not in a normalized basis as in Spoonacular, but they were in format of free text description that required to be processed and classified as described below.

## Ingredients classification

Given a free text description of an ingredient, the challenge is to give it a normalized name to make easier the retrieval of recipes for a query of entered ingredients. For example, if you have an ingredient with description "2 cucharadas de aceite de oliva" you expect to classify it as "Aceite de oliva". However, this is not an easy task due to the given aspects that may happen on the text:

- Information about quantity, size and units that may variate for each kind of recipe (e.g. "1 kg de ajíes morrones rojos grandes")
- Information about how the ingredient should be prepared (e.g. "1 cebolla mediana picada fina")
- Personal comments about the quality of the ingredient (e.g. "2 hojas de laurel fileteadas (mejor fresco)")
- Some ingredients share very similar names (e.g. "Cebolla" & "Cebolla de verdeo", "Aceite" & "Aceite de oliva")
- Some ingredients even change of name depending on the region (e.g. "Calabacin" means "Zucchini" in other countries)

Therefore, the approach taken was to train a classifier to process a text description and tell which is the corresponding ingredient. To tackle the given complexity in the text, a [FastText model](https://fasttext.cc/docs/en/supervised-tutorial.html) was defined due to be easy to adjust but powerful as it is a neural network designed for this purpose. The first step was to prepare a labeled dataset to train and validate the model with 2 fields per record (a text description, and a "label" indicating the expected ingredient classification), which was built as it follows:

1. Label some few records (e.g. 5 per class) by manually indicating which ingredient to put for a given text description
2. Remove those records for ingredients very specific and hard to find in usual recipes. This means that **only recipes with supported ingredients will be valid in the database**.
3. Train a model and use it to classify new recors (e.g. 10 per class)
4. Correct some mislabeled records by manually and loop until you have a bigger labeled dataset (e.g. 100 records per class)

A labeled dataset was obtained and stored in One Drive, so then it was used to train a FastText model. A [script](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/starvapprecom/classifier/fasttext.py) was developed to do that, which does the following steps
- Load labeled dataset
- Pre-process the text descriptions (e.g. apply lowercase, remove special characters and numbers, remove words from a blacklist)
- Split train-test sets for [cross-validation](https://scikit-learn.org/stable/modules/cross_validation.html#cross-validation)
- Train model with some defined parameters
- Get results about train and test sets to validate the performance of the model


Once the model was obtained, then it is ready to be used to [classify the extracted recipes](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/scripts/scraper_plasticwebs.py#L234) to build the database. As a result, we can have the datasets with proper fields and ingredients ready to be used for the application.

## Recommender approach

<!-- Document the approach taken to develop the solution based on the defined requirements and knowledge about the project -->

Given an input from the user with ingredients, and a database with recipes, the idea was to have a recommender system to decide the top of recipes to return with some criteria. In this project, there were two approaches explored in this order:

1. Basic scores
2. Weight ingredients

Both of them were intended for a ["cold start"](https://medium.com/yusp/the-cold-start-problem-for-recommender-systems-89a76505a7), where no users history is available. Therefore, recommendations are obtained only based on the recipes information, not from users.

!!! tip
    Following the [KISS principle](https://en.wikipedia.org/wiki/KISS_principle), it is suggested to always start with a very simple approach (even if it is not Machine Learning), and once it is implemented you can collect feedback and data about usage and then refine it.

### Approach 1: Basic Scores

Based on the available data, the decision was to start with a simple approach of basic scoring, in order to accelerate integration and collect early feedback to justify the need for a more complex approach

#### Idea
In this "naive" approach, the idea is to get the recipes with the best score based on the following aspects:

- Matching of ingredients entered
- Ratings of recipes
- Cooking time required

By tuning the score functions of these aspects, as well as the way to aggregate them, we could have a very quick way to select the best recipes to recommend, in order to be able to start the integration with other services and then have the structure needed to try a better approach.

#### Metrics

In order to assess if a set of recommended recipes is appropriate for a set of entered ingredients, these are the aspects to be evaluated:

- Quality of matched ingredients, by checking average matching ratio of ingredients > 0.5 
- Popularity of recipes, by checking average rating > 4.0
- Complexity of recipes, by checking average cooking time < 60

The problem of this approach is that it does not take into account some concept of relevancy on ingredients involved in a recommended recipe. So, as a result, some "not common" recipes may be selected when having few matches of ingredients for a given input, and that was a not desired aspect that was analyzed to propose a new solution.

More details about approach in [this Jupyter notebook](https://github.com/Matech-Studios/starvapp_recipes_recommender/blob/main/notebooks/20201228_basic_approach_scores.ipynb).

### Approach 2: Weight Ingredients

To tackle the problem of recommendations with not common ingredients when there are few matched ingredients, a new approach was analyzed to "weight" ingredients to achieve a selection of "feasible" recipes on the recommendations delivered.

#### Idea

Given that context, we could compute some "weights" for the given ingredients on the database, where the idea is to express in a score with range [0, 1] how "common" is an ingredient. Therefore, the recommendations that will select additional ingredients that are not part of the input, could prefer "common" ingredients to be recommended instead of "not common" ones. 

So, through an analysis on the data available, ingredients were processed to get frecuency and scale it into a range of [0,1] with a logarithmic scale. The result was saved into a [YAML file](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/data/ingredients_metadata.yml), and the approach implemented was simply recommend recipes that:
- Are prone to contain the input ingredients (with high score)
- Are prone to contain "common" ingredients (with the explained score)

!!! info
    Notice that the YAML file indicates which is the list of supported ingredients, and the relevance they have in the given database.

Finally, **this was the approach selected to go production**.

## Service development

Once the recommendation approach was defined and developed, the final step is to develop and deploy a service that receives requests from the StarvApp with input of users and returns recipes recommendations. Therefore, a [script](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/scripts/service.py) was developed with [asyncio](https://docs.python.org/3/library/asyncio.html) to do the following steps:

1. Set an asynchronous connection to an existing RabbitMQ queue with [aio-pika](https://aio-pika.readthedocs.io/en/latest/) 
2. When a message is received, the service calls the function to process the input ingredients and return the recommended recipes
3. A response is delivered with the recommendations, on the same RabbitMQ queue. 

For more details about the integration between services in the application, check the architecture shown in [Chapter 2](../2_preparation).

To test the service before the deployment, another [script](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/scripts/task_random_recipes.py) was developed to emulate requests with a random input to ensure the service will respond correctly.


To deploy this service, a [Dockerfile](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/Dockerfile) was developed to install the system and run the service as entrypoint. At the same time, another [Dockerfile](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/Dockerfile_rabbitmq) was defined to also run the RabbitMQ service for the application. Notice that both services are managed by the DevOps team in their given infrastructure.

## Testing

<!-- Describe the implemented tests for the solution -->

For this solution, there where two types of testing implemented:

- **Unit testing:** To validate each module or block of code on the [starvapprecom](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/tree/main/starvapprecom) package mostly for code coverage purposes, which are implemented as [simple functions](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/tree/main/tests/unit) executed by `pytest`.
- **System testing:** To validate some *functional* [tests](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/tests/system/test_scenarios_approaches_weight_ingredients.py), as checks over the recommender with a dataset regarding some specific [scenarios](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/tests/system/scenarios.yml), each of them having [corresponding checks](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/tests/system/checks.py).

!!! info
    To run the system tests, a dataset must be available and with a configured valid path

## Deployment

<!-- Document the work done to deploy the implemented solution in a testing environment for testing purposes -->

The StarvApp has a Continuous Integration (CI) pipeline configured in terms of [Azure pipelines](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/azure_pipelines.yml) and handled by the DevOps team, so the CI is already managed in this work and the files explained in the Environment section are the contract to ensure correct integration. However, additionally a [Github Actions workflow](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/.github/workflows/ci_pipeline.yml) was configured for this repository to make the following checks:

- Smoke testing based on configured tests 
- Code coverage along the releases
- Correct linting  

You can check the configuration and current status in the [project's README](https://github.com/Matech-Studios/Matech.Starvapp.RecipesRecomender.Analytics/blob/main/README.md).

