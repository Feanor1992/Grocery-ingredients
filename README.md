# Grocery-ingredients

## 1. Import Necessary Libraries ##

The code begins by importing the required Python libraries:
- `pandas as pd`: Used for data manipulation and analysis, particularly for handling the CSV dataset.
- `numpy as np`: Provides numerical operations, such as handling arrays and mathematical computations.
- `Dash` components (`Dash`, `dcc`, `html`, `Input`, `Output`, `State`): These are part of the Dash framework for building the web application:
    - `Dash`: The core class to initialize the app.
    - `dcc` (Dash Core Components): Provides interactive components like dropdowns and graphs.
    - `html`: Allows the creation of HTML elements for the app’s layout.
    - `Input`, `Output`, `State`: Used to define interactivity through callbacks.
- `dash_ag_grid as dag`: Adds an interactive data grid to display the dataset in a tabular format.
- `plotly.graph_objs as go`: Enables the creation of interactive visualizations like radar charts, scatter plots, bar charts, and histograms.

## 2. Data Loading and Preprocessing ##

This section loads the dataset and prepares it for analysis.

**Loading the Data**
- The dataset is loaded from a file named `GroceryDB_foods.csv` using `pd.read_csv()`.
- Debugging information is printed:
    - `Initial dataset shape:` shows the number of rows and columns (e.g., (1000, 20)).
    - `Columns:` lists all column names for verification.

**Renaming Columns**
- Columns are renamed for consistency and clarity using `df.rename()`:
    - `price percal` → `price_per_cal`
    - `Total Fat` → `total_fat`
    - `Carbohydrate` → `carbohydrate`
    - `Sugars, total` → `total_sugars`
    - `Fiber, total dietary` → `total_fiber`
    - `Fatty acids, total saturated` → `total_saturated_fat`
    - `Total Vitamin A` → `total_vitamin_A`
- This ensures uniform naming conventions (e.g., lowercase with underscores).

**Handling Missing Values**
- ***Vitamins and Minerals:***
    - A list `vitamins_minerals` defines columns like `Vitamin C`, `total_vitamin_A`, `Calcium`, and `Iron`.
    - Missing values in these columns are filled with `0` using `fillna(0)`.
    - If a column is missing, a warning is printed (e.g., "Warning: Column Vitamin C not found in the dataset.").
- ***Other Numeric Columns:***
    - For all numeric columns (detected with `df.select_dtypes(include=[np.number]`)):
        - If negative values exist in critical columns (`price` or `package_weight`), they are replaced with the column’s median, and a message is printed (e.g., "Corrected negative values in column price to median value 2.5.").
        - For other numeric columns with negative values, a warning is printed without correction.
        - Missing values are filled with the column’s median using `fillna(df[col].median())`.
- ***Categorical Columns:***
    - Missing values in text columns (detected with `df.select_dtypes(include=['object'])`) are filled with 'Unknown'.

## 3. Feature Engineering ##

This section creates new features and defines nutrient categories for analysis.

**Price per Weight**
- A new column `price_per_weight` is calculated as `price / package_weight` (assuming `package_weight` is in grams).
- This feature represents the cost per unit of weight, enhancing the dataset’s analytical value.
**Nutrient Groups**
- Two lists categorize nutrients for later nutrient density calculations:
    - `beneficial`: Nutrients that are good in higher amounts:
        - `Protein`, `total_fiber`, `Vitamin C`, `total_vitamin_A`, `Calcium`, `Iron`
    - `negative`: Nutrients that are harmful in excess:
        - `total_fat`, `total_saturated_fat`, `total_sugars`, `Sodium`, `Cholesterol`
- These groups are used to evaluate the nutritional quality of foods.

## 4. Dash App Setup and Layout ##

This section initializes the Dash application and defines its user interface.

**App Initialization**
- A Dash app is created with `app = Dash(__name__)` and a server is assigned (`server = app.server`) for deployment purposes.

**Dropdown Preparation**
- ***Categories***: Unique values from `harmonized single category` are sorted into `categories` for a dropdown menu.
- ***Brands***: Unique values from `brand` are sorted into `brands` for another dropdown menu.

**Nutrient and Axis Options**
- `nutrient_options`: A list of dictionaries for nutrient selection in visualizations, e.g., `{'label': 'Protein', 'value': 'Protein'}` for nutrients like `Protein`, `total_fat`, etc.
- `axis_options`: Options for scatter plot axes, e.g., `{'label': 'Price', 'value': 'price'}` for `price`, `price_per_cal`, `nutrient_density`, and `price_per_weight`.
- `color_options`: Same as `axis_options`, used for color-coding in the scatter plot.

**Recommended Daily Values**
- A dictionary `recommended_values` provides sample daily intake values for nutrients (used in percentage calculations):
    - `Protein`: 50g
    - `total_fat`: 70g
    - `total_fiber`: 28g
    - etc.
- These are estimates and can be adjusted.

**App Layout**
- The layout is a `html.Div` with a dark theme (`'backgroundColor': '#1a1a1a', color: 'white'`), containing:
    - ***Title:*** `Enhanced Grocery Ingredients Dashboard`.
    - ***Category Dropdown:*** Multi-select with all categories as default.
    - ***Brand Dropdown:*** Multi-select, defaulting to `DANNON`.
    - ***Nutrient Dropdown:*** Multi-select for radar chart nutrients, defaulting to `Protein` and `total_fat`.
    - ***Scatter Plot Axes:*** Dropdowns for X-axis (default: `price`) and Y-axis (default: `nutrient_density`).
    - ***Scatter Color Dropdown:*** Default: `price_per_weight`.
    - ***Histogram Nutrient Dropdown:*** Single-select, default: `Protein`.
    - ***Weight Inputs:*** Numeric inputs for beneficial (default: 1) and negative (default: 1) nutrient weights.
    - ***Radar Metric Dropdown:*** Options: `zscore`, `absolute`, `percentage`, default: `zscore`.
    - ***Graphs:*** Four `dcc.Graph` components for radar, scatter, bar, and histogram charts.
    - ***Data Grid:*** A `dag.AgGrid` displaying the dataset with filtering, sorting, and pagination.

## 5. Callbacks for Interactive Updates ##

This section defines a callback function to update the dashboard based on user inputs.

**Callback Definition**
- ***Outputs:*** Updates the data grid (`rowData`) and figures for the radar, scatter, bar, and histogram charts.
- ***Inputs:*** Values from all dropdowns, weight inputs, and radar metric selection.

**Input Handling**
- Ensures `selected_categories` and `selected_brands` are lists (converts strings to single-item lists or uses empty lists if None).

**Data Filtering**
- Filters the dataset (`dff`) to include only rows where `harmonized single category` and `brand` match the selected values.

**Nutrient Density Calculation**
- ***Z-Scores:*** For each nutrient in `beneficial` and `negative` present in `dff`, z-scores are computed on the filtered subset using `(value - mean) / std`.
- ***Density:***
    - Beneficial z-scores are multiplied by `beneficial_weight` and summed.
    - Negative z-scores are multiplied by `negative_weight` and summed.
    - `nutrient_density` = beneficial sum - negative sum (or 0 if no nutrients are present).

**Radar Chart**
- Groups data by category and calculates mean values for selected nutrients.
- Applies the selected `radar_metric`:
    - `zscore`: Normalizes values to z-scores.
    - `percentage`: Converts to percentage of `recommended_values`.
    - `absolute`: Uses raw values.
- Creates a `go.Scatterpolar` trace for each nutrient, with a dark-themed layout.

**Scatter Plot**
- Plots `scatter_x` vs. `scatter_y` with `scatter_color` as the color variable, using a dark theme and tooltips showing product names.

**Bar Chart**
- Shows average `price_per_cal` by category, with a dark theme and salmon-colored bars.

**Histogram**
- Displays the distribution of the selected nutrient (`hist_nutrient`) with 20 bins, using a dark theme and sky-blue bars.

**Return Values**
Returns updated grid data (`dff.to_dict('records')`) and figures for all four graphs.

## 6. Run the App ##
- The app runs in debug mode with app.run(debug=True) when executed as the main script.
