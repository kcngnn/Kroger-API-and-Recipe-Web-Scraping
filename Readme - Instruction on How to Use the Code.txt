### A. Get authorization to access to Kroger API
To begin with, in order to access to Kroger API, users are required to have client credentials, including Client ID and Client Secret and register redirect URL. Users can sign up an account on [Kroger API](https://developer.kroger.com/) to get the credientials as per documentation. <br/>

<ins>**Step 1:**</ins> Set up Access Information as mentioned above.

<ins>**Step 2:**</ins> Request authentication URL. <br/>
Run function `kroger_auth(client_id,redirect_uri)` to get authentication URL. Click on the link generated and sign in with your existing Kroger account. Please note to use your **Kroger grocery account**, not API account! If you don't have a grogery account yet, please sign up [here](https://www.kroger.com)! <br/>
After signing in to your account, you will be redirected to a link with authorization code, save that code to proceed. Below is an example of the redirected link, you can easily find out the code with the tag 'code' from the link. For example, the code is `tqEtjrNeDNUQt1UM1TAP2DFh9KVHZRoTIGngyWHB` from the link below. <br/>
`http://localhost:8888/lab/workspaces/auto-3?code=tqEtjrNeDNUQt1UM1TAP2DFh9KVHZRoTIGngyWHB`  

<ins>**Step 3:**</ins> Save the authorization code from previous step and run the function `kroger_access(auth_code)` to make an access token request using authozation code. The token response includes both an access and refresh token. <br/>
If access is not successfully granted, you will see the warning `"Access Unsuccessfully Granted! Please try again!"`. Otherwise, there would be no warning or notification at all. The return of the function is a list. You can retrieve access token and refresh token by accessing the second and third element of the list. <br/>
The sample code below demonstrate this step. <br/>
`auth_code = 'abc123'
access = kroger_access(auth_code)
access_token = access[1]
refresh_token = access[2]` <br/>
Warning: the request fails sometimes. If it does, repeat the process - click on the authentication URL to get a new authorization code and make a new access token request. Each authorization code can be used only once. <br/>
Also, note that access token expires after 30 minutes.

**When access token expires after 30 minutes**, use function `kroger_refresh(refresh_token)` to obtain a new access token without prompting the user. Response is exactly the same as when you obtain access token using authorizaton code. It includes a new access token and refresh token. Note that refresh token expires in 6 months, so the returned refresh token in this case can either be the same or a new one, depending on its validity. By running the funtion, if access is not successfully granted, the same warning would be issued. Otherwise, you would see no warning or notification at all. Access token and refresh token can be retrieved from the second and third element of the returned list.

### B. Integrated Code to Pull Recipes Data and Product Information from Kroger API
Integration of all the functions is a short script that makes a few function calls and can run for a set amount of time. The basis of the script is an infinite while loop that can be broken if in two ways: if set number of recipes are collected, or if a set amount of time has elapsed. The script also implements requirements for working with the Kroger API and for scraping information from allrecipes.com: the Kroger API has a 10,000 call per day limit and allrecipes.com requires a 1 second crawl delay. The following variables can be set according to preference of how the script stops:
- `max_counter`: Set this variable to the number of recipes that should be collected. Also uncomment the code at the bottom of the while loop that will stop collection when the limit is met. 
- `elapsed_time`: Set this variable to the length of time data should be collected. This should be in seconds. Uncomment the stop code that uses elapsed time. 

The ultimate goal is to get a list of suggestions for purchase at a given Kroger location for different ingredients of a number of recipes. <br/>
In order to pull suggestions to purchase from Kroger for ingredients of certain recipes, you would need to use the function `integration(elapsed_time,result_limit)` to get the result in a dictionary format, or do one more step with the function `suggestions_df(suggestions)` to convert the dictionary result to a dataframe format. <br/>
To run the function for the result in a dictionary format, you would need to input two variables - elapsed_time, and resultlimit.
- `elapsed_time`: As mentioned above, this refers to the length of time data should be collected. 
- `result_limit`: Interger value ranging from 0 to 50. Kroger API returns data with a value between this range. You can set the value to choose how many suggestions you want to get for each ingredient.

In order to get the result in a dataframe format, simply run the function `integration(elapsed_time,resultlimit)` first, then use the result as input for the function 
`suggestions_df(suggestions)`.

The returned result has the following formats:
- **Dictionary:** <br/>
        suggestions_dict = {'recipe_1': {'ingredient_1': {'upc_1': {'description': 'product_name',
                                                                    'price_regular': 123,
                                                                    'size': '1 Gal/qt/...',
                                                                    'soldBy': 'Unit/Weight/...'},
                                                          'upc_2':....}...}...}
- **DataFrame:** <br/> 
|recipe|ingredient|upc|description|price_regular|size|soldBy|
|---|---|---|---|---|---|---|
|value|value|value|value|value|value|value|

<ins>**Note:**</ins> You need to run necessary functions to get access token and refresh token, and assign the values to variables named `acess_token` and `refresh_token` respectively, before you could run function `integration()`. However, you will not need to make any new access token request once you run the function `integration()`. The integration code would request new access token when the old one has expired to continue. Thus, this code is not restricted in terms of elapsed time.

### C. Explanation of other functions defined
#### 1. Allrecipes.com Web Crawling Code
- `allowable_paths(URL)`: This function was taken from the DSCI 511 classes exercises. It intakes a URL and returns all allowable paths from the robots file for that website.
- `allowed_links(URL)`: This function was taken from the DSCI 511 class exercises. It intakes a URL and return all links a crawling program is allowed to access. 
- `find_recipe(url, recipe_numbers)`: This function searches the links on a webpage for a new recipe that has not yet been read by the function. If no recipe link is found, the function returns a URL to a webpage that contains /recipes/ in its URL. The /recipes/ page contain many links to more recipes. URL is in a string format while recipe_numbers is in a list format.
- `get_link(url, recipe_numbers)`: This function calls the `find_recipe()` function as many times as needed to get a needed URL to a recipe that has not been extracted from allrecipes.com yet. 
- `get_ingredients(url)`: This function takes in a url and returns a dictionary that contains the title of the recipe and the ingredients. The ingredients returned have been stripped of measurement or extraneous descriptive words and numbers.

#### 2. Kroger Code
- `pull_product_data(category,resultlimit)`: This function is written to be included in the main function for suggested purchase. The function uses two inputs - product names (e.g. milk, eggs, etc.), and result limit (to specify how many result to return for the product - ranging between 1 and 50). This function first pulls the raw data, and then unpack some nested information to return details on fulfillment types and prices. In order to pull prices, the request needs to tell a specific Kroger location. Generally, most of Kroger stores offer similar products and prices, a location has been defined in the code - location ID being `01400441`. This function incorporated testing validity of access token while pulling data, and would refresh token if the old one has expired.
- `get_suggestions(recipe,resultlimit,api_counter)`: This function is written to be included in the main integration function. The function uses three inputs - recipe, resultlimit, and api_counter. Recipe is inputted in a dictionary format with recipe name as a key and value being a list of ingredients. The function would utilize `pull_product_data()` to pull a determined number of suggestions (set by variable `resultlimit`) from Kroger for each ingredient. Details on suggestions include UPC, product name, regular price, size, and selling unit. `Api_counter` is a variable set to check the number of calls made against Kroger API following the limit of 10,000 calls per day.

### D. Limitations
Some limitations and challenges running the provided code and future development are forecasted as below:
- **Access to Kroger Unsuccessfully Granted:** The access token request to Kroger fails sometimes. Users must repeat the whole process - click on the link, sign in to Kroger account to get authorization code, make request - until access is successfully granted. 
- **Limited Results Returned:** Kroger only supports to return results from 1 to 50 values. Users cannot pull more results if needed.
- **Limited calls to Kroger Per Day:** Product API (the one in use for this code) has a 10,000 call per day rate limit.
- **Crawl Delay at Allrecipes.com:** Crawl delay of 1 second is required between requests.
- **Fuzzy Search Leading to Variability in Results:** The returned results for each search on Kroger do not include any criteria for filtering like top ratings or so. Also, API performs a fuzzy search based on the term provided, thus returned results are only those relevant to the search key word, and thus the results may vary from request to request.
- **Inconsistent Query Result Quality:** Lots of results were returned without necessary information like prices, sizes, selling units. Final Output sometimes contains null values for this reason.
- **Ingredients Contain Extra Words:** Measurements and frequent extraneous words are filtered out of the ingredients list. However, the filtering could be defined more in order to perfect the ingredients that are fed to the Kroger API.