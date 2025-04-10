# plagirism_test
# HackerRank Submission Scraper and Plagiarism Detector

## Overview
1. Scrape student submissions from HackerRank contest administration pages
2. Clean and process the raw code data
3. Detect potential plagiarism by identifying identical code submissions across different users

## Features

- **Automated Web Scraping**: Navigates through HackerRank's contest administration interface using Selenium
- **Robust Data Collection**: Extracts submission details including:
  - Submission ID
  - Username
  - Problem name
  - Programming language
  - Complete source code
- **Careful Resource Management**:
  - Processes submissions in batches to reduce memory usage
  - Implements smart pagination handling with error recovery
  - Uses random wait times to avoid detection
- **Code Cleaning**: Removes line numbers and normalizes code formatting
- **Plagiarism Detection**:
  - Groups submissions by problem
  - Uses hash-based comparison to efficiently identify identical code
  - Generates detailed reports of potential plagiarism cases

## Requirements

- Python 3.6+
- Chrome browser
- Selenium WebDriver for Chrome
- Required Python packages:
  - `selenium`
  - `pandas`
  - `tqdm`

## Installation

1. Clone the repository or download the source code
2. Install required packages:
   ```
   pip install selenium pandas tqdm
   ```
3. Download the Chrome WebDriver that matches your Chrome browser version from [the official site](https://sites.google.com/chromium.org/driver/) and ensure it's in your PATH

## Usage Guide

### 1. Setting Up the Scraper

The script is designed to navigate through HackerRank's administration interface. You'll need administrator access to the contest you want to analyze.

```python
# Configure Chrome options
options = Options()
options.add_argument("--start-maximized")
options.add_experimental_option("detach", True)

# Add options to bypass detection
options.add_argument("--disable-blink-features=AutomationControlled")
options.add_experimental_option("excludeSwitches", ["enable-automation"])
options.add_experimental_option("useAutomationExtension", False)

# Start Selenium
driver = webdriver.Chrome(options=options)

# Execute CDP commands to bypass detection
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
    "source": """
        Object.defineProperty(navigator, 'webdriver', {
            get: () => undefined
        })
    """
})

# Navigate to HackerRank
driver.get("https://www.hackerrank.com/dashboard")
```

### 2. Running the Scraper

After logging into HackerRank and navigating to the contest submissions page, run the scraper:

```python
scrape_hackerrank_with_pagination(
    driver=driver,
    max_pages=190,           # Adjust as needed
    batch_size=3,            # Process 3 submissions at a time
    output_dir="hackerrank_data",
    wait_between_pages=(5,8) # Random wait between pages
)
```

The scraper will:
- Process each page of submissions
- Extract data from each submission
- Save data to CSV files (one per page and a combined file)
- Log progress and any errors

### 3. Cleaning the Data

After scraping, clean the data to remove line numbers and prepare for plagiarism detection:

```python
# Clean the scraped data
input_path = "hackerrank_data/all_submissions.csv"
output_path = "hackerrank_data_cleaned/cleaned_all_submissions.csv"
process_csv_file(input_path, output_path)
```

### 4. Detecting Plagiarism

Run the plagiarism detection on the cleaned data:

```python
# Detect plagiarism
input_file = "hackerrank_data_cleaned/cleaned_all_submissions.csv"
detect_plagiarism(input_file, "plagiarism_results")
```

This will generate:
- A summary report of plagiarism statistics
- A detailed report with code samples
- A CSV file listing all detected plagiarism groups

## Output Files

### Scraper Output
- `hackerrank_data/hackerrank_submissions_page{XXX}.csv`: Individual page results
- `hackerrank_data/all_submissions.csv`: Combined results from all pages
- `hackerrank_data/scrape_log_{timestamp}.txt`: Detailed log of the scraping process

### Plagiarism Detection Output
- `plagiarism_results/plagiarism_summary.csv`: Summary statistics by problem
- `plagiarism_results/plagiarism_detailed.txt`: Detailed report with code samples
- `plagiarism_results/plagiarism_groups.csv`: Comprehensive list of all plagiarism groups

## Plagiarism Detection Methodology

The plagiarism detection works by:

1. Grouping submissions by problem name
2. Normalizing code by stripping whitespace
3. Creating MD5 hashes of the normalized code
4. Identifying submissions with identical hashes
5. Grouping and reporting identical submissions

This approach identifies exact code matches while ignoring minor formatting differences.

## Limitations

- The tool identifies exact code matches only, not partial or obfuscated plagiarism
- Requires administrator access to HackerRank contests
- Web scraping depends on HackerRank's interface, which may change
- May be subject to HackerRank's terms of service - use responsibly

## Contributing

Contributions to improve the tool are welcome! Some potential areas for enhancement:

- Advanced plagiarism detection (partial matches, algorithm fingerprinting)
- Support for other competitive programming platforms
- Performance optimizations for large contests
- Improved error handling and recovery

## License

This project is provided for educational purposes only. Use responsibly and in accordance with HackerRank's terms of service.
