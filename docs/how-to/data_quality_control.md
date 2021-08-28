# How-To: Data Quality Control

!!! abstract "Abstract"
    This guide is intended to provide some guidelines to make a data quality control over a dataset, to ensure the data has the expected content and quality.

    **Use this guide to:**

    - Conduct an efficient data quality control with Python tools in a Jupyter notebook.
    - Enhance previous data quality controls done with this presented guideline.

    **Assumptions:**

    - Data to be controlled is tabular.
    - Familiarity with Python tools and Jupyter notebooks.

## Resources

- Access to the dataset/s to be analyzed, stored in disk (e.g. in CSV, Parquet, ORC).
- Development environment with Python 3.x, having permissions to install new dependencies with pip.
- Jupyter notebook running, having access to UI.
- Jupyter notebook template for Data Quality Controls (optional)

## Setup

Before beggining the analysis, check that everything works correctly. 

- Place the dataset/s in a convenient folder, where you could also save new datasets generated in the analysis. 
    - We suggest to create a folder "data" at project level where you can place the input dataset/s and store the new ones.

- Install all the dependencies required for the analysis. 
    - It is recommended to use [Pandas](https://pandas.pydata.org/) if the datasets can entirely fit in memory or there are few aggregations to do (most probable scenario). Otherwise, you can use [Spark](https://spark.apache.org/) or [Dask](https://dask.org/) for larger data.
    - For visualizations, choose [Seaborn](https://seaborn.pydata.org) for static charts and [Plotly](https://plotly.com/) for interactive ones.

- Run Jupyter and access to UI (tipically, http://localhost:8888)

- Create a new notebook (from a template, if available) to run the analysis.

- Check that all the dependencies are correctly imported in the notebook.

- Check a correct loading of the dataset/s to control.

## Execution


### 1. Set goals and scope

It is important to describe the control to be done, to set expectation about the goals of the analysis and a scope to limit what will be done and what won't be controlled.

- Document goals and scope of the Data Quality Control to be done, in the first cells of the notebok.
    - The goals must be a summary of the outcome expected. For example, "Control correctness and coverage of columns in the dataset" and "Impute missing information with a proper strategy".
    - The scope should express what is expected to see in the notebook. For example, "Control coverage of columns", "Check distribution of values in variables", "Outliers detection", etc.
- Check both points are correctly documented, and are clear for another reader.

!!! tip
    This documentation should guide you during the analysis to avoid doing unnecessary work, as well as to help a reader to set expectations about the work to review.

### 2. Analyze data

Now it is time to analyze the data and make some check to control the content and quality. Here some points to conduct this analysis:

- Start with a quick inspection through a convenient tool that makes an automated analysis, like [Pandas profiling](https://github.com/pandas-profiling/pandas-profiling) or [Sweetviz](https://github.com/fbdesignpro/sweetviz). 
    - This will help to have a first outlook of the content and then decide the focus of the checks to be done by manually
    - Decide the tool depending on the size of the data. For instance, Pandas profiling by default does not handle large dataset, but it could be configured to optimize the processing.

!!! info
    To be continued soon...


!!! tip
    Ask for peer review in this analysis, mainly to ensure the control done is correct and it covers the needed aspects.

!!! tip
    Make focus on controling quality on the content rather than analyzing patterns or extracting insights from the data (for those goals you can conduct an Exploratory Data Analysis or EDA).

### 3. Document results and conclusions

Once your analysis it is pretty exhaustive to consider it as done, it is time to document the results.

- Collect all the insights extracted about the control done to summarize them at the end of the notebook in a section "Summary".
    - If possible, remark the treatment that the data need to ensure correct quality for later analytic tasks.
- Analyze those results obtained to document your conclusions about the quality assesed on data in a section "Conclusions"
    - Decide if the data has proper quality to proceed with later analytic task, or if some corrections must be done before.

- Document "Next steps" about work to be done, if already identified.

## Follow-up

The quality control is a never end process, so make a follow up to ensure the data sources to use will have the quality intended for a solution.

- Make corrective actions based on the conclusions extracted.
- Adjust code regarding processing of this data based on the indicated treatment.
- Load issues on the corresponding repositories that use the data.
