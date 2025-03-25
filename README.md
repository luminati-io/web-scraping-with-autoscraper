# Web Scraping with AutoScraper

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/)

This guide explains how to use AutoScraper with Python for web scraping:

- [What Is AutoScraper?](#what-is-autoscraper)
- [Setting up a Project](#setting-up-a-project)
- [Selecting a Target Website](#selecting-a-target-website)
- [Scraping Simple Data with AutoScraper](#scraping-simple-data-with-autoscraper)
- [Extracting Data from Websites with a Complex Design](#extracting-data-from-websites-with-a-complex-design)
- [Common Challenges with AutoScraper](#common-challenges-with-autoscraper)

## What Is AutoScraper?

[AutoScraper](https://github.com/alirezamika/autoscraper) is a Python library that simplifies web scraping by automatically identifying and extracting data from websites based on example queries, eliminating the need for manual HTML inspection. It efficiently handles dynamic websites with minimal setup. Learn more about scraping dynamic websites [here](https://brightdata.com/blog/how-tos/scrape-dynamic-websites-python).

## Setting up a Project

Once you have Python 3 or a later version installed, create a project directory and enter it:

```bash
mkdir auto-scrape && cd auto-scrape
```

Create a virtual environment:

```bash
python -m venv env
```

Then, activate it. On Linux and macOS:

```bash
source env/bin/activate
```

On Windows:

```powershell
venv\Scripts\activate
```

Install the `autoscraper` and `pandas` libraries:

```bash
pip install autoscraper pandas
```

## Selecting a Target Website

When scraping public websites, make sure you check the site’s Terms of Service (ToS) or `robots.txt` file to ensure the site allows scraping.

This guide assumes scraping data from [Scrape This Site’s Countries of the World: A Simple Example page](https://www.scrapethissite.com/pages/simple/), a beginner-friendly sandbox designed for testing scraping tools.

This page has a simple structure, making it an excellent starting point for learning the basics of data extraction. Once you’re comfortable with the fundamental concepts, we’ll move on to a more complex example—the [Hockey Teams: Forms, Searching, and Pagination page](https://www.scrapethissite.com/pages/forms/).

## Scraping Simple Data with AutoScraper

Use the following script to scrape a list of countries, along with their capital, population, and area:

```python
# 1. Import dependencies
from autoscraper import AutoScraper
import pandas as pd

# 2. Define the URL of the site to be scraped
url = "https://www.scrapethissite.com/pages/simple/"

# 3. Instantiate the AutoScraper
scraper = AutoScraper()

# 4. Define the wanted list by using an example from the web page
# This list should contain some text or values that you want to scrape
wanted_list = ["Andorra", "Andorra la Vella", "84000", "468.0"]

# 5. Build the scraper based on the wanted list and URL
scraper.build(url, wanted_list)

# 6. Get the results for all the elements matched
results = scraper.get_result_similar(url, grouped=True)

# 7. Display the keys and sample data to understand the structure
print("Keys found by the scraper:", results.keys())

# 8. Assign columns based on scraper keys and expected order of data
columns = ["Country Name", "Capital", "Area (sq km)", "Population"]

# 9. Create a DataFrame with the extracted data
data = {columns[i]: results[list(results.keys())[i]] for i in range(len(columns))}
df = pd.DataFrame(data)

# 10. Save the DataFrame to a CSV file
csv_filename = 'countries_data.csv'
df.to_csv(csv_filename, index=False)

print(f"Data has been successfully saved to {csv_filename}")
```

The above script begins by importing the necessary libraries: AutoScraper and pandas. Next, the target website's URL is defined, followed by the creation of an AutoScraper instance.

Here’s where AutoScraper stands out: unlike traditional scrapers that require explicit instructions, such as XPath or CSS selectors, AutoScraper allows you to provide example data. Instead of specifying where elements are located on the page, you simply list a few sample values you want to extract. This list, called the `wanted_list`, contains representative data points—such as a country’s name, capital, population, and area.

Once the `wanted_list` is set, the scraper is built using the provided URL and sample data. AutoScraper then analyzes the webpage, identifies patterns, and generates scraping rules. These rules allow the scraper to recognize and extract similar data from the same or other web pages.

Further down in the script, the `get_result_similar` method is called to retrieve all data matching the patterns AutoScraper has learned. To better understand how the scraper interprets the data structure, the script prints the rule IDs associated with the extracted data. The output should resemble this:

```
Keys found by the scraper: dict_keys(['rule_4y6n', 'rule_gghn', 'rule_a6r9', 'rule_os29'])
```

Comments eight and nine define column names and structure the extracted data into a pandas DataFrame. Comment ten then saves this data as a CSV file.

After running the script (`python script.py`), a `countries_data.csv` file will appear in the project directory with contents like:

```csv
Country Name,Capital,Area (sq km),Population
Andorra,Andorra la Vella,84000,468.0
United Arab Emirates,Abu Dhabi,4975593,82880.0
...246 collapsed rows
Zambia,Lusaka,13460305,752614.0
Zimbabwe,Harare,11651858,390580.0
```

## Extracting Data from Websites with a Complex Design

The previous technique may struggle with complex websites like the [Hockey Teams page](https://www.scrapethissite.com/pages/forms/), which features a table with many similar values. Try extracting team names, years, wins, and other fields using the same method to observe the challenge.

Fortunately, AutoScraper supports refining its model by pruning unnecessary rules during the build step. Here’s how you can do that:

```python
from autoscraper import AutoScraper
import pandas as pd

# Define the URL of the site to be scraped
url = "https://www.scrapethissite.com/pages/forms/"

def setup_model():

    # Instantiate the AutoScraper
    scraper = AutoScraper()

    # Define the wanted list by using an example from the web page
    # This list should contain some text or values that you want to scrape
    wanted_list = ["Boston Bruins", "1990", "44", "24", "0.55", "299", "264", "35"]

    # Build the scraper based on the wanted list and URL
    scraper.build(url, wanted_list)

    # Get the results for all the elements matched
    results = scraper.get_result_similar(url, grouped=True)

    # Display the data to understand the structure
    print(results)

    # Save the model
    scraper.save("teams_model.json")

def prune_rules():
    # Create an instance of Autoscraper
    scraper = AutoScraper()
    
    # Load the model saved earlier
    scraper.load("teams_model.json")

    # Update the model to only keep necessary rules
    scraper.keep_rules(['rule_hjk5', 'rule_9sty', 'rule_2hml', 'rule_3qvv', 'rule_e8x1', 'rule_mhl4', 'rule_h090', 'rule_xg34'])

    # Save the updated model again
    scraper.save("teams_model.json")
    
def load_and_run_model():
    # Create an instance of Autoscraper
    scraper = AutoScraper()
    
    # Load the model saved earlier
    scraper.load("teams_model.json")

    # Get the results for all the elements matched
    results = scraper.get_result_similar(url, grouped=True)

    # Assign columns based on scraper keys and expected order of data
    columns = ["Team Name", "Year", "Wins", "Losses", "Win %", "Goals For (GF)", "Goals Against (GA)", "+/-"]

    # Create a DataFrame with the extracted data
    data = {columns[i]: results[list(results.keys())[i]] for i in range(len(columns))}
    df = pd.DataFrame(data)

    # Save the DataFrame to a CSV file
    csv_filename = 'teams_data.csv'
    df.to_csv(csv_filename, index=False)

    print(f"Data has been successfully saved to {csv_filename}")

# setup_model()
# prune_rules()
# load_and_run_model()
```

This script defines three methods: `setup_model`, `prune_rules`, and `load_and_run_model`.

The `setup_model` method creates a scraper instance, defines a `wanted_list`, builds the scraper, scrapes data from the target URL, prints the collected rule IDs, and saves the model as `teams_model.json` in the project directory.

To run the script, uncomment the `# setup_model()` line in the script, save the file (e.g., `script.py`), and run it with `python script.py`.

The output will look like this:

```
{'rule_hjk5': ['Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks', 'Detroit Red Wings', 'Edmonton Oilers', 'Hartford Whalers', 'Los Angeles Kings', 'Minnesota North Stars', 'Montreal Canadiens', 'New Jersey Devils', 'New York Islanders', 'New York Rangers', 'Philadelphia Flyers', 'Pittsburgh Penguins', 'Quebec Nordiques', 'St. Louis Blues', 'Toronto Maple Leafs', 'Vancouver Canucks', 'Washington Capitals', 'Winnipeg Jets', 'Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks'], 'rule_uuj6': ['Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks', 'Detroit Red Wings', 'Edmonton Oilers', 'Hartford Whalers', 'Los Angeles Kings', 'Minnesota North Stars', 'Montreal Canadiens', 'New Jersey Devils', 'New York Islanders', 'New York Rangers', 'Philadelphia Flyers', 'Pittsburgh Penguins', 'Quebec Nordiques', 'St. Louis Blues', 'Toronto Maple Leafs', 'Vancouver Canucks', 'Washington Capitals', 'Winnipeg Jets', 'Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks'], 'rule_9sty': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_9nie': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_41rr': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ufil': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ere2': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_w0vo': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_rba5': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_rmae': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ccvi': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_3c34': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_4j80': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_oc36': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_93k1': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_d31n': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ghh5': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_5rne': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_4p78': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_qr7s': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_60nk': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_wcj7': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_0x7y': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_2hml': ['44', '31', '46', '49', '34', '37', '31', '46', '27', '39', '32', '25', '36', '33', '41', '16', '47', '23', '28', '37', '26', '36', '31', '31', '36'], 'rule_swtb': ['24'], 'rule_e8x1': ['0.55', '14', '0.575', '0.613', '-25', '0', '-38', '0.575', '-10', '24', '8', '-67', '32', '-15', '0.512', '-118', '0.588', '-77', '-72', '0', '-28', '-5', '-10', '-9', '21'], 'rule_3qvv': ['24', '30', '26', '23', '38', '37', '38', '24', '39', '30', '33', '45', '31', '37', '33', '50', '22', '46', '43', '36', '43', '32', '37', '37', '29'], 'rule_n07w': ['24', '30', '26', '23', '38', '37', '38', '24', '39', '30', '33', '45', '31', '37', '33', '50', '22', '46', '43', '36', '43', '32', '37', '37', '29'], 'rule_qmem': ['0.55', '0.388', '0.575', '0.613', '0.425', '0.463', '0.388', '0.575', '0.338', '0.487', '0.4', '0.312', '0.45', '0.412', '0.512', '0.2', '0.588', '0.287', '0.35', '0.463', '0.325', '0.45', '0.388', '0.388', '0.45'], 'rule_b9gx': ['264', '278', '263', '211', '298', '272', '276', '254', '266', '249', '264', '290', '265', '267', '305', '354', '250', '318', '315', '258', '288', '275', '299', '305', '236'], 'rule_mhl4': ['299', '292', '344', '284', '273', '272', '238', '340', '256', '273', '272', '223', '297', '252', '342', '236', '310', '241', '243', '258', '260', '270', '289', '296', '257'], 'rule_24nt': ['264', '278', '263', '211', '298', '272', '276', '254', '266', '249', '264', '290', '265', '267', '305', '354', '250', '318', '315', '258', '288', '275', '299', '305', '236'], 'rule_h090': ['264', '278', '263', '211', '298', '272', '276', '254', '266', '249', '264', '290', '265', '267', '305', '354', '250', '318', '315', '258', '288', '275', '299', '305', '236'], 'rule_xg34': ['35', '14', '81', '73', '-25', '0', '-38', '86', '-10', '24', '8', '-67', '32', '-15', '37', '-118', '60', '-77', '-72', '0', '-28', '-5', '-10', '-9', '21']}
```

This output shows the data collected by AutoScraper in its `get_result_similar` call from the target website. It contains duplicates because AutoScraper attempts to infer relationships and group data into rules. When it groups correctly, data can be easily extracted from similar sites.

However, AutoScraper struggles with this site due to many numbers, leading to numerous incorrect correlations and a large dataset with duplicates.

To proceed, analyze the data and select the rules with the correct data (i.e., matching the data from one column in the correct order). In this case, the following rules contained the right data (determined by verifying that each rule contained 25 data points, corresponding to the number of rows on the target page):

```
['rule_hjk5', 'rule_9sty', 'rule_2hml', 'rule_3qvv', 'rule_e8x1', 'rule_mhl4', 'rule_h090', 'rule_xg34']
```

Update the `prune_rules` method. Then, comment out the `setup_model()` line and uncomment the `prune_rules()` line in the script. When you run it, it loads the model from `teams_model.json`, removes unnecessary rules, and saves the updated model. You can inspect the contents of `teams_model.json` to verify the stored rules.

Next, to run the `load_and_run_model` method, comment out the `prune_rules` lines, uncomment the `load_and_run_model` line, and rerun the script. This will extract and save the correct data in `teams_data.csv` in the project directory, along with printing the following output:

```
Data has been successfully saved to teams_data.csv
```

Here’s what the `teams_data.csv` file looks like after a successful run:

```csv
Team Name,Year,Wins,Losses,Win %,Goals For (GF),Goals Against (GA),+/-
Boston Bruins,1990,44,0.55,24,299,264,35
Buffalo Sabres,1990,31,14,30,292,278,14
...21 more rows
Calgary Flames,1991,31,-9,37,296,305,-9
Chicago Blackhawks,1991,36,21,29,257,236,21
```

All the code developed in this article is available in [this GitHub repo](https://github.com/krharsh17/auto-scrape).

## Common Challenges with AutoScraper

AutoScraper is great for simple use cases with small datasets and distinct data points. However, it can be cumbersome for more complex scenarios, like scraping tables. It also doesn’t support JavaScript rendering, so you'll need to integrate it with tools like [Splash](https://pypi.org/project/splash/), Selenium, or Puppeteer.

If you face issues like IP blocks or need custom headers, AutoScraper allows you to specify additional request parameters for its requests module, like this:

```python
# build the scraper on an initial URL
scraper.build(
    url,
    wanted_list=wanted_list,
    request_args=dict(proxies=proxies) # this is where you can pass in a list of proxies or customer headers
)
```

For example, here’s how you can set a custom user agent and a proxy for scraping with AutoScraper:

```python
request_args = { 
  "headers: {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_5) AppleWebKit/537.36 \
            (KHTML, like Gecko) Chrome/84.0.4147.135 Safari/537.36"  # You can customize this value with your desired user agent. This value is the default used by Autoscraper.
  },
  "proxies": {
    "http": "http://user:[email protected]:3128/" # Example proxy to show you how to use the proxy port, host, username, and password values
  }
}
# build the scraper on an initial URL
scraper.build(
    url,
    wanted_list=wanted_list,
    request_args=request_args
)
```

AutoScraper uses Python's requests library and doesn’t support rate-limiting. To handle rate-limiting, you can manually implement throttling or use a prebuilt solution like the [`ratelimit`](https://pypi.org/project/ratelimit/) library.

Since AutoScraper works only with non-dynamic websites, it cannot handle CAPTCHA-protected sites. For these cases, consider using a more robust solution like the [Bright Data Web Scraping API](https://brightdata.com/products/web-scraper), which supports structured data extraction from sites like LinkedIn, Amazon, and Zillow.

Sign up now and start exploring Bright Data’s products, including a free trial!
