---
title: Code readability
teaching: 60
exercises: 30
---

::: questions

- Why does code readability matter?
- How can I organize my code to be more readable?
- What types of documentation can I include to improve the readability of my code?

:::

::: objectives

After completing this episode, participants should be able to:

- Import third-party libraries at the top of a script
- Choose function and variable names that help explain the purpose of the function or variable
- Organize code into reusable functions that achieve a singular purpose
- Write informative comments and roxygen2 comments to provide more detail about what the code is doing

:::

In this episode, we will introduce the concept of readable code and consider how it can help create reusable scientific software and empower collaboration between researchers.

While all developers hope their code will be stable long term, software often has to change due to changes in the real world. As requirements change, so must the relevant code. When code needs to be changed, the developer that created it or more likely a different developer needs to understand that code before they can implement the new requirements. 

Readable code facilitates the reading and understanding of the abstraction phases and, as a result, facilitates the evolution of the codebase. Readable code saves future developers' time and effort.

In order to develop readable code, we should ask ourselves: "If I re-read this piece of code in fifteen days or one year, will I be able to understand what I have done and why?"  Or even better: "If a new person who just joined the project reads my software, will they be able to understand what I have written here?"

In this episode, we will learn a few specific software best practices we can follow to help create more readable code. 

:::::: spoiler

### Code state

At this point, the code in your local software project's directory should be as in:
<https://github.com/carpentries-incubator/bbrs-software-project/tree/04-code-readability>

:::

::: spoiler

### Make sure your packages and dependencies are up to date

In the previous section we discusses using the package renv.lock to track packages and their dependencies. At this time, you should have a current renv.lock files and you should have restored the packages library. 

You can check that the project is up-to-date with 

```r
renv::status() #you can run this any time
```

If you haven't, you can restored the packages and their dependencies by running

```r
renv::restore("renv.lock")
```

:::

## Place `library` funtions at the top

Let’s look at our code again. One thing that stands out is that we’re calling library() in multiple places throughout the script. By convention, all libraries should be loaded at the top so dependencies are easy to see and not buried in the logic. This improves readability and makes the code easier to reuse and maintain.

Our code after the modification should look like the following.

```r

library()

#the rest of the code goes below 

```

Let's make sure we commit our changes.

```bash
(venv_spacewalks) $ git add eva_data_analysis.R
(venv_spacewalks) $ git commit -m "Move library calls to the top of the script"
```
## Rules for variable names in R

Check the [official documentation](https://cran.r-project.org/doc/manuals/r-release/R-intro.html#R-commands_002c-case-sensitivity_002c-etc_002e)

- Only alphanumeric characters, dot, and underscores are permitted in variable names.  
- Must start with a letter or a dot (.); if it starts with a dot, the next character cannot be a digit.
- After the first character, you can use letters, digits, dots, and underscores.  
- Cannot contain spaces or most punctuation (e.g., -, +, ?, !, @) unless you quote the name.  
- Cannot be a reserved word (keywords) such as if, else, repeat, while, function, for, in, next, break, TRUE, FALSE, NULL, NA, NaN, Inf, etc.  
- Variable names — and objects in general — are case-sensitive. So `speed_of_light` and `Speed_Of_Light` are not the same. 



### Useful things to consider when naming variables

Variables are the most common thing you will assign when coding, and it's really important that it is clear what each variable means in order to understand what the code is doing.

If you return to your code after a long time doing something else, or share your code with a colleague, it should be easy enough to understand what variables are involved in your code from their names.

Therefore we need to give them clear names, but we also want to keep them concise so the code stays readable. There are no "hard and fast rules" here, and it's often a case of using your best judgment.

> “There are only two hard things in Computer Science: cache invalidation and naming things.”  
Phil Karlton



Some useful tips for naming variables

- Short words are better than single character names. For example, if we were creating a variable to store the speed to read a file, `s` (for 'speed') is not descriptive enough but `MBReadPerSecondAverageAfterLastFlushToLog` is too long to read and prone to misspellings. `ReadSpeed` (or `read_speed`) would suffice.  
- If you are finding it difficult to come up with a variable name that is both short and descriptive, go with the short version and use an inline comment to describe it further (more on those in the next section).  
This guidance does not necessarily apply if your variable is a well-known constant in your domain - for example, *c* represents the speed of light in physics.  
- Try to be descriptive where possible and avoid meaningless or funny names like `foo`, `bar`, `var`, `thing`, etc.  
- Programming languages often have global pre-built functions, such as `input`, which you may accidentally overwrite if you assign a variable with the same name and no longer be able to access the original `input` function. In this case, opting for something like `input_data` would be preferable. 


:::::: challenge

### Rename our variables to be more descriptive (5 min)

Let's apply this to `eva_data_analysis.R`.  ## should this be my code v2.R?

a. Edit the code as follows to use descriptive (and consistent) variable names:

    - Change `data_f` to `input_file`
    - Change `data_t` to `output_file`
    - Change `g_file` to `graph_file`

    *Be sure to change all the occurrences of each variable name.*
b. What other variable names in our code would benefit from renaming? 
Rename these too. 
Hint: variables `w`, `t`, `tt` and `ttt` could also be renamed to be more descriptive.
  - **Change `w` to `csv_writer`**: makes it clear this variable is a CSV writer object. Using "w" alone would more likely be interpreted as "width" or "weight".
  - **Change `tt` to `duration_str`**: represents a string form of the duration, indicated by "_str".
  - **Change `t` to `duration_dt`**: a datetime object parsed from the string, indicated by "_dt".
  - **Change `ttt` to `duration_hours`**: the duration converted into (decimal) hours.
c. Commit your changes to your repository. Remember to use an informative commit message.


::: solution


Updated code after renaming `data_f`, `data_t` and `g_file` as well as variables `w`, `t`, `tt` and `ttt` to be more descriptive. 
      
      
```{r}

# R translation of the provided Python script
# - tidyverse-first approach
# - variable renames applied:
#   data_f -> input_file, data_t -> output_file, g_file -> graph_file
#   w -> csv_writer, tt -> duration_str, t -> duration_dt, ttt -> duration_hours

library(tidyverse)
library(lubridate)
library(jsonlite)

# Files
input_file  <- "./eva-data.json"
output_file <- "./eva-data.csv"
graph_file  <- "./cumulative_eva_graph.png"

# Expected output columns (keep in the same order as the Python "fieldnames" intent)
fieldnames <- c("EVA #", "Country", "Crew    ", "Vehicle", "Date", "Duration", "Purpose")

# ---- Read JSON lines (first 375 lines), mimicking the Python loop ----
raw_lines <- readr::read_lines(input_file, n_max = 375)

# Python does: json.loads(line[1:-1]) (drops first and last char).
# We'll do the same *conditionally* to avoid breaking valid JSON lines.
trim_one_char_each_side <- function(x) {
  if (nchar(x) >= 2) substr(x, 2, nchar(x) - 1) else x
}

json_records <- raw_lines |>
  keep(~ nzchar(.x)) |>
  map(trim_one_char_each_side) |>
  map(\(x) {
    # Robust parsing: return NULL for lines that aren't valid JSON after trimming
    tryCatch(jsonlite::fromJSON(x, simplifyVector = TRUE), error = function(e) NULL)
  }) |>
  compact()

# Convert list of records to a tibble
eva_tbl <- tibble::tibble(record = json_records) |>
  tidyr::unnest_wider(record)

# ---- Write CSV (analogous to csv_writer.writerow(data[j].values())) ----
# The Python code writes raw "values()" order, which is not stable across dicts.
# In R, we’ll write a stable, explicit column order:
csv_writer <- eva_tbl |>
  # try to match/standardize to the requested fieldnames where possible
  # (these names may differ in your modified NASA export; adjust if needed)
  rename(
    `EVA #`    = any_of(c("EVA #", "eva", "eva_number", "eva_num")),
    `Country`  = any_of(c("Country", "country")),
    `Crew    ` = any_of(c("Crew    ", "Crew", "crew")),
    `Vehicle`  = any_of(c("Vehicle", "vehicle")),
    `Date`     = any_of(c("Date", "date")),
    `Duration` = any_of(c("Duration", "duration")),
    `Purpose`  = any_of(c("Purpose", "purpose"))
  ) |>
  # ensure all expected columns exist (create missing as NA)
  mutate(across(setdiff(fieldnames, names(.)), ~ NA_character_)) |>
  select(all_of(fieldnames))

readr::write_csv(csv_writer, output_file, na = "")

# ---- Compute duration_hours and cumulative total; then plot ----
plot_tbl <- csv_writer |>
  transmute(
    date = suppressWarnings(lubridate::ymd(str_sub(`Date`, 1, 10))),
    duration_str = as.character(`Duration`)
  ) |>
  filter(!is.na(date), !is.na(duration_str), duration_str != "") |>
  mutate(
    # Parse "HH:MM" like Python's datetime.strptime(tt, "%H:%M")
    duration_dt = suppressWarnings(lubridate::hm(duration_str)),
    duration_hours = as.numeric(duration_dt) / 3600
  ) |>
  filter(!is.na(duration_hours)) |>
  arrange(date) |>
  mutate(cumulative_hours = cumsum(duration_hours))

p <- ggplot(plot_tbl, aes(x = date, y = cumulative_hours)) +
  geom_line() +
  geom_point() +
  labs(
    x = "Year",
    y = "Total time spent in space to date (hours)"
  ) +
  theme_minimal()

ggsave(
  filename = graph_file,
  plot = p,
  width = 9,
  height = 5,
  dpi = 300
)

print(p)

```
c. Let's commit our latest changes:

```bash
(venv_spacewalks) $ git add eva_data_analysis.R
(venv_spacewalks) $ git commit -m "Use descriptive variable names"
(venv_spacewalks) $ git push origin main
```

:::
::::::

As we have now updated all the variable names to be more descriptive, we can now go and close the issue on GitHub we created earlier.
To do so, we open our repository in GitHub, switch to the Issues tab, find the issue to "improve variable names" we created earlier.
There are more automated ways to close issues based on a commit/pull request that we will learn later, for now we will click the "Close issue" button at the bottom of the discussion.

## Remove unused variables and imports

Unused variables or import statements can cause confusion about what the code is doing, making it harder to read and easier to introduce mistakes. Such things may seem harmless as they do not cause immediate syntax errors - but they can potentially lead to subtle program logic errors, unexpected behavior, wrong results and issues later on making them especially tricky to detect and fix. Over time, this makes the codebase more fragile and harder to maintain and extend.

:::::: challenge

### Remove an unused variable (2 min)

Find and remove an unused variable in our code. Then, commit the updated code to the git repo.

::: solution

Variable `fieldnames` (containing column names for CSV data file) is defined but never used in the code - it should be deleted.

Updated code:

```{r}

# R translation of the provided Python script
# - tidyverse-first approach
# - variable renames applied:
#   data_f -> input_file, data_t -> output_file, g_file -> graph_file
#   w -> csv_writer, tt -> duration_str, t -> duration_dt, ttt -> duration_hours

library(tidyverse)
library(lubridate)
library(jsonlite)

# Files
input_file  <- "./eva-data.json"
output_file <- "./eva-data.csv"
graph_file  <- "./cumulative_eva_graph.png"

# ---- Read JSON lines (first 375 lines), mimicking the Python loop ----
raw_lines <- readr::read_lines(input_file, n_max = 375)

# Python does: json.loads(line[1:-1]) (drops first and last char).
# We'll do the same *conditionally* to avoid breaking valid JSON lines.
trim_one_char_each_side <- function(x) {
  if (nchar(x) >= 2) substr(x, 2, nchar(x) - 1) else x
}

json_records <- raw_lines |>
  keep(~ nzchar(.x)) |>
  map(trim_one_char_each_side) |>
  map(\(x) {
    # Robust parsing: return NULL for lines that aren't valid JSON after trimming
    tryCatch(jsonlite::fromJSON(x, simplifyVector = TRUE), error = function(e) NULL)
  }) |>
  compact()

# Convert list of records to a tibble
eva_tbl <- tibble(record = json_records) |>
  tidyr::unnest_wider(record)

# ---- Write CSV (analogous to csv_writer.writerow(data[j].values())) ----
# NOTE: We intentionally do *not* force a fixed column schema here, to mirror the
# Python behavior more closely (dict.values()) while still writing a usable CSV.
csv_writer <- eva_tbl |>
  rename(
    `EVA #`    = any_of(c("EVA #", "eva", "eva_number", "eva_num")),
    `Country`  = any_of(c("Country", "country")),
    `Crew    ` = any_of(c("Crew    ", "Crew", "crew")),
    `Vehicle`  = any_of(c("Vehicle", "vehicle")),
    `Date`     = any_of(c("Date", "date")),
    `Duration` = any_of(c("Duration", "duration")),
    `Purpose`  = any_of(c("Purpose", "purpose"))
  )

readr::write_csv(csv_writer, output_file, na = "")

# ---- Compute duration_hours and cumulative total; then plot ----
plot_tbl <- csv_writer |>
  transmute(
    date = suppressWarnings(lubridate::ymd(str_sub(`Date`, 1, 10))),
    duration_str = as.character(`Duration`)
  ) |>
  filter(!is.na(date), !is.na(duration_str), duration_str != "") |>
  mutate(
    # Parse "HH:MM" like Python's datetime.strptime(tt, "%H:%M")
    duration_dt = suppressWarnings(lubridate::hm(duration_str)),
    duration_hours = as.numeric(duration_dt) / 3600
  ) |>
  filter(!is.na(duration_hours)) |>
  arrange(date) |>
  mutate(cumulative_hours = cumsum(duration_hours))

p <- ggplot(plot_tbl, aes(x = date, y = cumulative_hours)) +
  geom_line() +
  geom_point() +
  labs(
    x = "Year",
    y = "Total time spent in space to date (hours)"
  ) +
  theme_minimal()

ggsave(
  filename = graph_file,
  plot = p,
  width = 9,
  height = 5,
  dpi = 300
)

print(p)

```

Commit changes:

```bash
(venv_spacewalks) $ git add eva_data_analysis.R
(venv_spacewalks) $ git commit -m "Remove unused variable fieldname"
(venv_spacewalks) $ git push origin main
```

:::
::::::

:::::::::::::::::::::::::::::::: callout

[Linters](https://glosario.carpentries.org/en/#linter) (static analysis tools) can be very helpful with tasks like this.

A linter is a tool that automatically checks your source code for problems without running it. For Python, linters mainly flag:
	•	Style issues: formatting, naming conventions, line length, import order
	•	Potential bugs: unused imports/variables, unreachable code, dubious comparisons, shadowing built-ins
	•	Code smells / maintainability: overly complex functions, too many branches, duplicated logic
	•	Sometimes security issues (depending on the tool)

Linters usually produce warnings/errors with line numbers and (often) suggested fixes.

For R, 	lintr, the standard R linter does static code analysis for style issues and potential problems, supports many editors (including RStudio/VS Code), and is configurable via a .lintr file. Alongside lintr, R programmers commonly use styler which auto-formats R code so you eliminate many lint issues by formatting consistently.

::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::: callout

IDEs can significantly help developers enhance code readability.
They use built-in linters that automatically identify and flag issues in real-time as you write code. 
This process improves readability in several ways:

- **enforcing coding standards**: check for consistent style, formatting, and conventions, e.g. flagging inconsistent indentation, incorrect naming conventions, and excessive line length violations, which helps ensure a consistent look and feel across the entire codebase, making it easier for any developer to read and understand.
- **syntax highlighting and error detection**: use different colors and font styles for different parts of the code (keywords, variables, comments, etc.), providing visual cues that make the code easier to parse and read. 
For example, flagging syntax errors (e.g., missing brackets, semicolons, or misspelled keywords) with visual indicators like wavy red underlines ("squigglies"), allowing for immediate correction and preventing errors from becoming entrenched.
- **highlighting "code smells" and inefficiencies**: beyond simple syntax, advanced IDEs and their extensions can identify "code smells" and potential inefficiencies, such as unused variables, overly complex functions, or duplicate code blocks. 
This encourages developers to refactor the code into smaller, more meaningful functions and classes, which improves clarity and maintainability.
- **providing contextual guidance**: many IDEs provide rich, contextual help and suggestions on how to fix an issue and why it is a problem, helping developers learn and apply best practices for writing high-quality, readable code.
- **facilitating code navigation and refactoring**: features like intelligent code completion, symbol resolution, and automated refactoring support (e.g., renaming variables or extracting methods) allow developers to restructure code to be more efficient and readable without changing its core functionality. 
The IDE understands the underlying structure of the code, which makes these complex operations simple and safe.
::::::::::::::::::::::::::::::::::::::::



:::::::::::::::::::::::::::::::: callout
How to install lintr and styler for RStudio

1) Install the packages

```{r}
install.packages(c("lintr", "styler"))
```

2) Use lintr in RStudio (see issues in the Markers pane)

Enable R diagnostics in RStudio

RStudio can show lintr results in the Markers pane when diagnostics are enabled.
  - Tools → Global Options… → Code → Diagnostics
  - Check “Show diagnostics for R”

Lint a file (results appear in Markers)

From the Console:

```{r}
lintr::lint("path/to/file.R")
```
Then open Markers (in the pane that also shows Build/Git/etc.) to review findings

Use the RStudio Addins

lintr ships RStudio addins for linting the current source and the package. You can run them via:
  - Tools → Addins → Browse Addins… → search "lintr"
	-	Optional: bind keyboard shortcuts via Tools → Addins → Browse Addins → Keyboard Shortcuts (the lintr docs even suggest common bindings).

3) Use styler in RStudio (auto-format code)

RStudio Addins

styler provides RStudio addins to:
	-	style the active file
	-	style the current package
	-	style the highlighted selection  

Find them at:
	-	Tools → Addins → Browse Addins… → search “styler”


A practical “RStudio workflow”
	1.	Write code
	2.	Run styler addin (format)
	3.	Run lintr addin (lint) and fix anything meaningful
	
A minimal .lintr config file 

```
linters: lintr::linters_with_defaults(
  # Keep the signal high; tune to taste.
  line_length_linter = lintr::line_length_linter(100),

  # Common readability/style checks that work well in both .R and R chunks in .qmd/.Rmd
  object_name_linter = lintr::object_name_linter(),
  spaces_left_parentheses_linter = lintr::spaces_left_parentheses_linter(),
  spaces_inside_linter = lintr::spaces_inside_linter(),
  trailing_whitespace_linter = lintr::trailing_whitespace_linter(),

  # Catch likely mistakes without being too noisy
  unused_import_linter = lintr::unused_import_linter()
)

# Exclude rendered/output directories typical of Quarto projects
exclusions: list(
  "_site/",
  "_book/",
  "_freeze/",
  "docs/",
  "site_libs/",
  "renv/"
)
```


::::::::::::::::::::::::::::::::::::::::

#got this far 3/10/2026

## Use third-party libraries

Our script currently reads the data line-by-line from the JSON data file and uses custom code to manipulate the data.
Variables of interest are stored in lists but there are more suitable data structures (e.g. `pandas`' dataframe) to store data in our case.
By choosing custom code over popular and well-tested libraries, we are making our code less readable and understandable and more error-prone.

The main functionality of our code can be rewritten as follows using the `pandas` library to load and manipulate the data in data frames.

First, we need to install this dependency into our virtual environment.

```bash
(venv_spacewalks) $ python3 -m pip install pandas
```

Then we will edit the code to use `pandas`.
For the sake of time in the workshop, we will give you the updated code.
The code should now look like:

```python
import matplotlib.pyplot as plt
import pandas as pd

# Data source: https://data.nasa.gov/resource/eva.json (with modifications)
input_file = open('./eva-data.json', 'r', encoding='ascii')
output_file = open('./eva-data.csv', 'w', encoding='utf-8')
graph_file = './cumulative_eva_graph.png'

eva_df = pd.read_json(input_file, convert_dates=['date'], encoding='ascii')
eva_df['eva'] = eva_df['eva'].astype(float)
eva_df.dropna(axis=0, subset=['duration', 'date'], inplace=True)

eva_df.to_csv(output_file, index=False, encoding='utf-8')

eva_df.sort_values('date', inplace=True)

eva_df['duration_hours'] = eva_df['duration'].str.split(":").apply(lambda x: int(x[0]) + int(x[1])/60)
eva_df['cumulative_time'] = eva_df['duration_hours'].cumsum()
plt.plot(eva_df['date'], eva_df['cumulative_time'], 'ko-')
plt.xlabel('Year')
plt.ylabel('Total time spent in space to date (hours)')
plt.tight_layout()
plt.savefig(graph_file)
plt.show()

```

Once we have replaced the  code in our Python script `eva_data_analysis.py` with the above code, we need to make sure that we capture the changes in our virtual development environment too.

```bash
(venv_spacewalks) $ python3 -m pip freeze > requirements.txt
```

Now, we need to commit the changes we have made. We can add multiple files to the same commit by listing all of them. Remember to use an informative commit message.

```bash
(venv_spacewalks) $ git add eva_data_analysis.py requirements.txt
(venv_spacewalks) $ git commit -m "Refactor code and add Pandas to venv"
(venv_spacewalks) $ git push origin main
```

We have committed the code and the environment changes together since they are related and form one logical unit of change.

## Use comments to explain functionality

Commenting is a very useful practice to help convey the context of the code.
It can be helpful as a reminder for your future self or your collaborators as to why code is written in a certain way, 
how it is achieving a specific task, or the real-world implications of your code.

There are several ways to add comments to code:

- An **inline comment** is a comment on the same line as a code statement. 
Typically, it comes after the code statement and finishes when the line ends and 
is useful when you want to explain the code line in short. 
Inline comments in Python should be separated by at least two spaces from the statement; they start with a # followed
by a single space, and have no end delimiter.
- A **single-line comment** or **prologue comment** is a comment that comes the line before a block of code to explain it.
- A **multi-line** or **block comment** can span multiple lines and has a start and end sequence.
To comment out a block of code in Python, you can either add a # at the beginning of each line of the block or 
surround the entire block with three single (`'''`) or double quotes (`"""`).

```python
x = 5  # In Python, inline comments begin with the `#` symbol and a single space.

# this is a single-line comment
y = x + 10
z = y*2 + x

'''
This is a multiline
comment
in Python.
'''
```

Here are a few things to keep in mind when commenting your code:

- Focus on the **why** and the **how** of your code - avoid using comments to explain **what** your code does. 
If your code is too complex for other programmers to understand, consider rewriting it for clarity rather than adding comments to explain it.
- Make sure you are not reiterating something that your code already conveys on its own. Comments should not echo your code.
- Keep comments short and concise. Large blocks of text quickly become unreadable and difficult to maintain.
- Comments that contradict the code are worse than no comments. Always make a priority of keeping comments up-to-date when code changes.

### Examples of unhelpful comments

```python
statetax = 1.0625  # Assigns the float 1.0625 to the variable 'statetax'
citytax = 1.01  # Assigns the float 1.01 to the variable 'citytax'
specialtax = 1.01  # Assigns the float 1.01 to the variable 'specialtax'
```

The comments in this code simply tell us what the code does, which is easy enough to figure out without the inline comments.

### Examples of helpful comments

```python
statetax = 1.0625  # State sales tax rate is 6.25% through Jan. 1
citytax = 1.01  # City sales tax rate is 1% through Jan. 1
specialtax = 1.01  # Special sales tax rate is 1% through Jan. 1
```

In this case, it might not be immediately obvious what each variable represents, so the comments offer helpful, real-world context.
The date in the comment also indicates when the code might need to be updated.

::: challenge

### Add comments to our code (10 min)

a. Examine `eva_data_analysis.py`.
Add as many comments as you think is required to help yourself and others understand what that code is doing.
b. Commit your changes to your repository. Remember to use an informative commit message.

::: solution

### Solution

Some good comments may look like the example below.

``` python
import matplotlib.pyplot as plt
import pandas as pd


# https://data.nasa.gov/resource/eva.json (with modifications)
input_file = open('./eva-data.json', 'r', encoding='ascii')
output_file = open('./eva-data.csv', 'w', encoding='utf-8')
graph_file = './cumulative_eva_graph.png'

print("--START--")
print(f'Reading JSON file {input_file}')
# Read the data from a JSON file into a Pandas dataframe
eva_df = pd.read_json(input_file, convert_dates=['date'], encoding='ascii')
eva_df['eva'] = eva_df['eva'].astype(float)
# Clean the data by removing any rows where duration is missing
eva_df.dropna(axis=0, subset=['duration', 'date'], inplace=True)

print(f'Saving to CSV file {output_file}')
# Save dataframe to CSV file for later analysis
eva_df.to_csv(output_file, index=False, encoding='utf-8')

# Sort dataframe by date ready to be plotted (date values are on x-axis)
eva_df.sort_values('date', inplace=True)

# Plot cumulative time spent in space over years
print(f'Plotting cumulative spacewalk duration and saving to {graph_file}')
eva_df['duration_hours'] = eva_df['duration'].str.split(":").apply(lambda x: int(x[0]) + int(x[1])/60)
eva_df['cumulative_time'] = eva_df['duration_hours'].cumsum()
plt.plot(eva_df['date'], eva_df['cumulative_time'], 'ko-')
plt.xlabel('Year')
plt.ylabel('Total time spent in space to date (hours)')
plt.tight_layout()
plt.savefig(graph_file)
plt.show()
print("--END--")
```

Note that we have also added some useful print statements, to let us know what stage the analysis is in.

Commit changes:

```bash
(venv_spacewalks) $ git add eva_data_analysis.py
(venv_spacewalks) $ git commit -m "Add inline comments to the code"
(venv_spacewalks) $ git push origin main
```

:::
:::

## Separate units of functionality

Functions are a fundamental concept in writing software and are one of the core ways you can organise your code to 
improve its readability.
A function is an isolated section of code that performs a single, *specific* task that can be simple or complex.
It can then be called multiple times with different inputs throughout a codebase, but its definition only needs to 
appear once.

Breaking up code into functions in this manner benefits readability since the smaller sections are easier to read 
and understand.
Since functions can be reused, codebases naturally begin to follow the [Don't Repeat Yourself principle][dry-principle] 
which prevents software from becoming overly long and confusing.
The software also becomes easier to maintain because, if the code encapsulated in a function needs to change, 
it only needs updating in one place instead of many.
As we will learn in a future episode, testing code also becomes simpler when code is written in functions.
Each function can be individually checked to ensure it is doing what is intended, which improves confidence in 
the software as a whole.

::: callout
Decomposing code into functions helps with reusability of blocks of code and eliminating repetition, 
but, equally importantly, it helps with code readability and testing.
:::

Looking at our code, you may notice it contains different pieces of functionality:

1. reading the data from a JSON file
2. converting and saving the data in the CSV format
3. processing/cleaning the data and preparing it for analysis 
4. data analysis and visualising the results

Let's refactor our code so that reading the data in JSON format into a dataframe (step 1) and converting it and saving 
to the CSV format (step 2) are extracted into separate functions.
Let's name those functions `read_json_to_dataframe` and `write_dataframe_to_csv` respectively. 
The main part of the script should then be simplified to invoke these new functions, while the functions themselves 
contain the complexity of each of these two steps. We will continue to work on steps 3 and 4 above later on.

:::::::::: instructor

You will need to share the code below with the learners via copy-and-paste either in shared notes or chat in a virtual training. Then you can highlight and describe the changes.

:::::::::::::::::::::

After the initial refactoring, our code may look something like the following.

```python
import matplotlib.pyplot as plt
import pandas as pd

def read_json_to_dataframe(input_file):
    print(f'Reading JSON file {input_file}')
    # Read the data from a JSON file into a Pandas dataframe
    eva_df = pd.read_json(input_file, convert_dates=['date'], encoding='ascii')
    eva_df['eva'] = eva_df['eva'].astype(float)
    # Clean the data by removing any rows where duration is missing
    eva_df.dropna(axis=0, subset=['duration', 'date'], inplace=True)
    return eva_df


def write_dataframe_to_csv(df, output_file):
    print(f'Saving to CSV file {output_file}')
    # Save dataframe to CSV file for later analysis
    df.to_csv(output_file, index=False, encoding='utf-8')


# Main code

print("--START--")

input_file = open('./eva-data.json', 'r', encoding='ascii')
output_file = open('./eva-data.csv', 'w', encoding='utf-8')
graph_file = './cumulative_eva_graph.png'

# Read the data from JSON file
eva_data = read_json_to_dataframe(input_file)

# Convert and export data to CSV file
write_dataframe_to_csv(eva_data, output_file)

# Sort dataframe by date ready to be plotted (date values are on x-axis)
eva_data.sort_values('date', inplace=True)

# Plot cumulative time spent in space over years
print(f'Plotting cumulative spacewalk duration and saving to {graph_file}')
eva_data['duration_hours'] = eva_data['duration'].str.split(":").apply(lambda x: int(x[0]) + int(x[1])/60)
eva_data['cumulative_time'] = eva_data['duration_hours'].cumsum()
plt.plot(eva_data['date'], eva_data['cumulative_time'], 'ko-')
plt.xlabel('Year')
plt.ylabel('Total time spent in space to date (hours)')
plt.tight_layout()
plt.savefig(graph_file)
plt.show()

print("--END--")
```

We have chosen to create functions for reading in and writing out data files since this is a very common task within research software.
While these functions do not contain that many lines of code due to using the `pandas` in-built methods that do all the complex data reading, converting and writing operations, it can be useful to package these steps together into reusable functions if you need to read in or write out a lot of similarly structured files and process them in the same way.

We can further simplify the main part of our code by extracting the code to plot a graph into a separate function `plot_cumulative_time_in_space`.
Let's do that as an exercise.

::: challenge

### Extract functionality into a function (5 min)

Extract the code to plot a graph into a separate function `plot_cumulative_time_in_space(df, graph_file)`.
The function should take the following two arguments: 

1. a dataframe `df`, and 
2. a file object or a file path string `graph_file` where to save the plot.

Make sure to commit and push your changes.

::: solution

After extracting the code to plot a graph into a separate function, our code may look something like the following:

```python
import matplotlib.pyplot as plt
import pandas as pd

def read_json_to_dataframe(input_file):
    print(f'Reading JSON file {input_file}')
    # Read the data from a JSON file into a Pandas dataframe
    eva_df = pd.read_json(input_file, convert_dates=['date'], encoding='ascii')
    eva_df['eva'] = eva_df['eva'].astype(float)
    # Clean the data by removing any rows where duration is missing
    eva_df.dropna(axis=0, subset=['duration', 'date'], inplace=True)
    return eva_df


def write_dataframe_to_csv(df, output_file):
    print(f'Saving to CSV file {output_file}')
    # Save dataframe to CSV file for later analysis
    df.to_csv(output_file, index=False, encoding='utf-8')


def plot_cumulative_time_in_space(df, graph_file):
    print(f'Plotting cumulative spacewalk duration and saving to {graph_file}')
    df['duration_hours'] = df['duration'].str.split(":").apply(lambda x: int(x[0]) + int(x[1])/60)
    df['cumulative_time'] = df['duration_hours'].cumsum()
    plt.plot(df['date'], df['cumulative_time'], 'ko-')
    plt.xlabel('Year')
    plt.ylabel('Total time spent in space to date (hours)')
    plt.tight_layout()
    plt.savefig(graph_file)
    plt.show()

# Main code

print("--START--")

input_file = open('./eva-data.json', 'r', encoding='ascii')
output_file = open('./eva-data.csv', 'w', encoding='utf-8')
graph_file = './cumulative_eva_graph.png'

# Read the data from JSON file
eva_data = read_json_to_dataframe(input_file)

# Convert and export data to CSV file
write_dataframe_to_csv(eva_data, output_file)

# Sort dataframe by date ready to be plotted (date values are on x-axis)
eva_data.sort_values('date', inplace=True)

# Plot cumulative time spent in space over years
plot_cumulative_time_in_space(eva_data, graph_file)

print("--END--")
```
:::

:::

## Use docstrings to document functions

Now that we have written some functions, it is time to document them so that we can quickly recall 
(and others looking at our code in the future can understand) what the functions do without having to read
the code.

*Docstrings* are a specific type of documentation that are provided within functions and [Python classes][python-classes].
A function docstring should explain what that particular code is doing, what parameters the function needs (inputs)
and what form they should take, what the function outputs (you may see words like 'returns' or 'yields' here), 
and errors (if any) that might be raised.

Providing these docstrings helps improve code readability since it makes the function code more transparent and aids 
understanding.
Particularly, docstrings that provide information on the input and output of functions makes it easier to reuse them 
in other parts of the code, without having to read the full function to understand what needs to be provided and 
what will be returned.

Python docstrings are defined by enclosing the text with 3 double quotes (`"""`).
This text is also indented to the same level as the code defined beneath it, which is 4 whitespaces by convention.

### Example of a single-line docstring

```python
def add(x, y):
    """Add two numbers together"""
    return x + y
```

### Example of a multi-line docstring

```python
def divide(x, y):
    """
    Divide number x by number y.

    Args:
        x: A number to be divided.
        y: A number to divide by.

    Returns:
        float: The division of x by y.
        
    Raises:
        ZeroDivisionError: Cannot divide by zero.
    """
    return x / y
```

Some projects may have their own guidelines on how to write docstrings, such as [numpy][numpy-docstring].
If you are contributing code to a wider project or community, try to follow the guidelines and standards they provide 
for code style.

As your code grows and becomes more complex, the docstrings can form the content of a reference guide allowing 
developers to quickly look up how to use the APIs, functions, and classes defined in your codebase.
Hence, it is common to find tools that will automatically extract docstrings from your code and generate a website where people can learn about your code without downloading/installing and reading the code files - such as [MkDocs][mkdocs-org].

Let's write a docstring for the function `read_json_to_dataframe` we introduced in the previous exercise using the 
[Google Style Python Docstrings Convention][google-doc-string]. 
Remember, questions we want to answer when writing the docstring include:

- What the function does?
- What kind of inputs does the function take? Are they required or optional? Do they have default values?
- What output will the function produce?
- What exceptions/errors, if any, it can produce?

Our `read_json_to_dataframe` function fully described by a docstring may look like:

```python
def read_json_to_dataframe(input_file):
    """
    Read the data from a JSON file into a Pandas dataframe.
    Clean the data by removing any rows where the 'duration' value is missing.

    Args:
        input_file (file or str): The file object or path to the JSON file.

    Returns:
         eva_df (pd.DataFrame): The cleaned and sorted data as a dataframe structure
    """
    print(f'Reading JSON file {input_file}')
    # Read the data from a JSON file into a Pandas dataframe
    eva_df = pd.read_json(input_file, convert_dates=['date'], encoding='ascii')
    eva_df['eva'] = eva_df['eva'].astype(float)
    # Clean the data by removing any rows where duration is missing
    eva_df.dropna(axis=0, subset=['duration', 'date'], inplace=True)
    return eva_df
```

:::::: challenge

### Writing docstrings (5 min)

Write docstrings for the functions `write_dataframe_to_csv` and `plot_cumulative_time_in_space` we introduced earlier.

::: solution

### Solution

Our `write_dataframe_to_csv` function fully described by a docstring may look like:

```python
def write_dataframe_to_csv(df, output_file):
    """
    Write the dataframe to a CSV file.

    Args:
        df (pd.DataFrame): The input dataframe.
        output_file (file or str): The file object or path to the output CSV file.

    Returns:
        None
    """
    print(f'Saving to CSV file {output_file}')
    # Save dataframe to CSV file for later analysis
    df.to_csv(output_file, index=False, encoding='utf-8')
```

Our `plot_cumulative_time_in_space` function fully described by a docstring may look like:

```python
def plot_cumulative_time_in_space(df, graph_file):
    """
    Plot the cumulative time spent in space over years.

    Convert the duration column from strings to number of hours
    Calculate cumulative sum of durations
    Generate a plot of cumulative time spent in space over years and
    save it to the specified location

    Args:
        df (pd.DataFrame): The input dataframe.
        graph_file (file or str): The file object or path to the output graph file.

    Returns:
        None
    """
    df['duration_hours'] = df['duration'].str.split(":").apply(lambda x: int(x[0]) + int(x[1])/60)
    df['cumulative_time'] = df['duration_hours'].cumsum()
    plt.plot(df['date'], df['cumulative_time'], 'ko-')
    plt.xlabel('Year')
    plt.ylabel('Total time spent in space to date (hours)')
    plt.tight_layout()
    plt.savefig(graph_file)
    plt.show()
```

:::

::::::

Finally, our code may look something like the following:

``` python
import matplotlib.pyplot as plt
import pandas as pd

def read_json_to_dataframe(input_file):
    """
    Read the data from a JSON file into a Pandas dataframe.
    Clean the data by removing any rows where the 'duration' value is missing.

    Args:
        input_file (file or str): The file object or path to the JSON file.

    Returns:
         eva_df (pd.DataFrame): The cleaned and sorted data as a dataframe structure
    """
    print(f'Reading JSON file {input_file}')
    # Read the data from a JSON file into a Pandas dataframe
    eva_df = pd.read_json(input_file, convert_dates=['date'], encoding='ascii')
    eva_df['eva'] = eva_df['eva'].astype(float)
    # Clean the data by removing any rows where duration is missing
    eva_df.dropna(axis=0, subset=['duration', 'date'], inplace=True)
    return eva_df


def write_dataframe_to_csv(df, output_file):
    """
    Write the dataframe to a CSV file.

    Args:
        df (pd.DataFrame): The input dataframe.
        output_file (file or str): The file object or path to the output CSV file.

    Returns:
        None
    """
    print(f'Saving to CSV file {output_file}')
    # Save dataframe to CSV file for later analysis
    df.to_csv(output_file, index=False, encoding='utf-8')

def plot_cumulative_time_in_space(df, graph_file):
    """
    Plot the cumulative time spent in space over years.

    Convert the duration column from strings to number of hours
    Calculate cumulative sum of durations
    Generate a plot of cumulative time spent in space over years and
    save it to the specified location

    Args:
        df (pd.DataFrame): The input dataframe.
        graph_file (file or str): The file object or path to the output graph file.

    Returns:
        None
    """
    print(f'Plotting cumulative spacewalk duration and saving to {graph_file}')
    df['duration_hours'] = df['duration'].str.split(":").apply(lambda x: int(x[0]) + int(x[1])/60)
    df['cumulative_time'] = df['duration_hours'].cumsum()
    plt.plot(df['date'], df['cumulative_time'], 'ko-')
    plt.xlabel('Year')
    plt.ylabel('Total time spent in space to date (hours)')
    plt.tight_layout()
    plt.savefig(graph_file)
    plt.show()

# Main code

print("--START--")

input_file = open('./eva-data.json', 'r', encoding='ascii')
output_file = open('./eva-data.csv', 'w', encoding='utf-8')
graph_file = './cumulative_eva_graph.png'

# Read the data from JSON file
eva_data = read_json_to_dataframe(input_file)

# Convert and export data to CSV file
write_dataframe_to_csv(eva_data, output_file)

# Sort dataframe by date ready to be plotted (date values are on x-axis)
eva_data.sort_values('date', inplace=True)

# Plot cumulative time spent in space over years
plot_cumulative_time_in_space(eva_data, graph_file)

print("--END--")
```

Do not forget to commit any uncommitted changes you may have and then push your work to GitHub.

```bash
(venv_spacewalks) $ git add <your_changed_files>
(venv_spacewalks) $ git commit -m "Your commit message"
(venv_spacewalks) $ git push origin main
```

## Summary

Good code readability brings many benefits to software development. 
It makes code easier to understand, maintain, and debug. 
This benefits collaborators and future developers as well as the original author. 
Readable code reduces the risk of errors, speeds up onboarding of new team members, and simplifies code reviews. 
It also supports long-term sustainability, as clear code is more adaptable and easier to extend or refactor over time.

Integrated Development Environments (IDEs) significantly enhance code readability by using built-in static analysis tools (linters) that automatically identify and flag issues in real-time as you write code.
This proactive approach allows developers to identify and fix many issues as they write code, rather than later in the development cycle.

:::::: spoiler

### Code state

At this point, the code in your local software project's directory should be as in:
<https://github.com/carpentries-incubator/bbrs-software-project/tree/05-code-structure>

:::

## Further reading

We recommend the following resources for some additional reading on the topic of this episode:

- [7 tell-tale signs of unreadable code](https://www.index.dev/blog/7-tell-tale-signs-of-unreadable-code-how-to-identify-and-fix-the-problem)
- ['Code Readability Matters' from the Guardian's engineering blog][guardian-code-readability]
- [PEP 8 Style Guide for Python][pep8-comments]
- [Coursera: Inline commenting in Python][coursera-inline-comments]
- [Introducing Functions from Introduction to Python][python-functions-intro]
- [W3Schools.com Python Functions][python-functions-w3schools]

Also check the [full reference set](learners/reference.md#litref) for the course.

::: keypoints

- Readable code is easier to understand, maintain, debug and extend (reuse) - saving time and effort.
- Choosing descriptive variable and function names will communicate their purpose more effectively.
- Using comments and docstrings to describe parts of the code will help transmit understanding and context.
- Use libraries or packages for common functionality to avoid duplication.
- Creating functions from the smallest, reusable units of code will make the code more readable and help.
compartmentalise which parts of the code are doing what actions and isolate specific code sections for reuse.

:::
