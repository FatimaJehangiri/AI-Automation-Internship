# Day 9 – JavaScript for Automation (Code Node)

## Objective

The objective of this task was to learn how to use JavaScript inside the n8n Code node to transform and process data that cannot be handled efficiently using standard workflow nodes.

## Task Summary

A dataset containing 15 records with inconsistent name formatting, mixed-case email addresses, and numeric scores was processed using the Code node.

The workflow transformed the data by:

- Removing unnecessary spaces from names.
- Converting names to Title Case.
- Converting email addresses to lowercase.
- Computing a grade for each record based on the score.
- Filtering the dataset to retain only records with grades **A** and **B**.

## How the Code Node Transforms Data

The Code node receives all input records as JavaScript objects and processes them using array methods.

- **`.map()`** iterates through each record to clean and standardize the data while adding a new `grade` field.
- **String methods** such as `trim()` and `toLowerCase()` are used to normalize text.
- A custom JavaScript function determines the appropriate grade based on each student's score.
- **`.filter()`** removes records with grades lower than **B**, leaving only the required results.

The transformed records are then returned as the workflow output for use in subsequent automation steps.

## Skills Gained

- JavaScript fundamentals for automation
- Data transformation techniques
- Array manipulation using `.map()` and `.filter()`
- JSON object processing
- Building custom logic within n8n workflows

## Conclusion

This task demonstrated how the n8n Code node can extend workflow capabilities through JavaScript. By combining data cleaning, transformation, and filtering in a single node, the workflow became more efficient, flexible, and suitable for handling real-world automation scenarios.
