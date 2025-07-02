# Documentation: RSEc Atlas

## Aim:

With this static site web application we want to demonstrate the possibilities of the RSEc.
The website is designed to help researchers and scientists explore the metadata of various scientific tools and their associated containers, packages ... you name it, with the final aim of improving
findability. The application retrieves RAW metadata from the Git repository of the Research Software Ecosystem Content (https://github.com/research-software-ecosystem/content), "compiles" them to be compact as possible and provides a user-friendly interface to view, search, and filter them along with their metadata.


## Usage:
To access the web application, visit the static site deployed using GitHub Pages. It offers the following features:

### Searching:
The search functionality allows users to perform query-based searches to find specific bio tools and containers. Searching is relevance-based, which lists relevant tools according to the search string being part of Tool Name > Tool Tags > Tool Description.

> **Notes:** Searching is case-insensitive, so "BWA" and "bwa" will return the same results. `Tags` in the context of searching are the `EDAM Topics` available on the tool details page if they exist.

**Query Search:** Searching in the web application also employs querying. Query searching allows users to search a combination of different comma-separated queries, where a query can be can either a search string or a tag. Following are some of the examples of the input string describing the use-case:
1. Single search string (Tool Name, Tool Description, or a Tag):
   - BWA
   - DeepTools
   - tag:mapping
   - tag:antimicrobial resistance
2. Combination of search strings: (Input comma separated queries like below)
   - bwa, kit
   - BWA, tag:mapping
   - tag:sequence assembly, tag:mapping
   - SALSA, tag:sequence assembly, tag:mapping
3. Input `tag:*` to list all the tags available.

Searching can be done by the search bar available on the home page of the application.

### Filtering:
The application provides various filtering options to narrow down search results based on specific criteria. This helps users quickly find the most relevant tools for their needs. The following are the filters available in the web application:
1. **Data Availability:** Filter to check if tools have Bioconda Packages or Biocontainers or if they are compatible with Galaxy.
2. **License:** Filter the tools according to the licenses fetched from the metadata.
3. **Favourite:** Filter the tools that are marked as Favourite by the user.

These filters are available on the home page of the application.

### Sorting:
The tools on the homepage can be sorted by Name, Creation Date, and Last Modified Date. A drop-down on the home page of the application provides this functionality. By default, the tools are sorted by their Names.

### Favourites:
Users can mark bio tools and containers as favorites for quick access in the future. The favorites functionality ensures that users can maintain a personalized list of frequently used tools, stored in the browser. To mark a tool as a favourite, Users must go to the tool's details page by clicking on the tool name and then toggling the star in the top-right corner.

### URLs Sharing:
If users want to share the URL with a search query, it can be done by sharing the URL and appending `/?search=SEARCH QUERY`, which will be automatically converted to `/?search=SEARCH+QUERY`. Following are some of the examples of the URLs with the search query:
   - /?search=1000Genomes+ID+history+converter `(Gives the tools with search string as 1000Genomes ID history converter)`
   - /?search=tag:antimicrobial+resistance,+fargene `(Gives the tools with result consisting Tag antimicrobial resistance and search string as fargene)`

If users want to share a link to a tool, they can directly share the URL available on the tool's details page. The following are some of the examples:
   - /tool/1000genomes_id_history_converter
   - /tool/fargene


## Setup and Installation:

To set up and run the project locally, we can follow the below procedure:

#### 1. Install the prerequisites:
Before setting up the project, ensure you have the following installed on your local machine:
- **Node.js** (v18 or higher): Required for running the Nuxt.js frontend.
- **Python** (v3.x): Required for running the Python scripts to process and merge metadata files.
- **Git**: For version control and managing repository changes.

#### 2. Clone the Repository:
To get started, clone the repository to your local machine.

#### 3. Install Python Dependencies:
Navigate to the `backend` folder and install the required Python dependencies:
1. Set up a virtual environment (optional but recommended):
   ```bash
   cd backend
   python -m venv venv
   source venv/bin/activate
   ```
2. Install the necessary Python packages from the `requirements.txt` file:
   ```bash
   pip install -r requirements.txt
   ```
   This will install all required packages, including `PyYAML` for parsing YAML files.

#### 4. Install Node.js Dependencies:
Navigate to the `frontend` directory and install the required Node.js dependencies using `npm`:
   ```bash
   cd frontend
   npm install
   ```

#### 5. Clone the RSE Content Repository:
We have to clone the RSE Content Repository to the directory `backend`, which will be later used by the python script to fetch and combine the metadata.
This will generate a folder called as `content` in the directory.
```bash
git clone https://github.com/research-software-ecosystem/content.git backend/content
```

#### 6. Run the Python Script
The Python script `merge_data_files.py` in `backend` processes and merges the metadata into `combined_metadata.json` also create a metadata file for each tool. This file is later used by the Nuxt.js frontend.
```bash
python backend/merge_data_files.py
```
This will generate the metadata files in the `frontend/public/metadata` directory.

#### 8. Generate the Static Site
Navigate to the `frontend` directory and run the following command to generate the static site:
```bash
cd frontend
npm run generate
```
This will create a fully static site in the `.output/public` directory.

#### 9. Preview the Site
To preview the generated static site, run:
```bash
npm run preview
```
You should now be able to access the site locally at `http://localhost:3000`.


## Implementation:
Following section defines the implementation of the project:

### Python Script:
The Python script is responsible for processing, merging metadata files and create metadata for each tool. It includes the following components:

- Directory Structure:
  ```
  backend/
  ├── content/                # Created after pulling the RSC Content repository
  ├── requirements.txt        # File with python dependencies, which are to be installed before running the Python script
  ├── merge_data_files.py     # Python script to merge the metadata from the RCS Content repository
  ```
- Working:
  1. The Python script traverses through all the folders in `backend/content/data` directory, where each folder represents a tool.
  2. Each folder has metadata of several files either in JSON or YAML format.
  3. To retrieve the data from these files for each specific tool, we have defined a variable called `file_patterns` in `process_files_in_folder()` function. To change the files from which we want to retrieve the information, one must modify this variable:
      ```python
      file_patterns = [
        (f"bioconda_{folder_name}.yaml", "bioconda"),
        (f"{folder_name}.biocontainers.yaml", "biocontainers"),
        (f"{folder_name}.biotools.json", "biotools"),
        (f"{folder_name}.bioschemas.jsonld", "bioschemas"),
        (f"{folder_name}.galaxy.json", "galaxy"),
       ]
      ```
      If none of these files exist, the tool is skipped.

  4. The next step is to define which data is to be fetched from each file. We use a variable called `SUMMARY_DATA_KEY_MAPPINGS` to determine which keys from the JSON or YAML files are to be fetched, and how they are supposed to be stored in our combined JSON file:
      ```json
      SUMMARY_DATA_KEY_MAPPINGS = {
       "bioconda": {
           "bioconda__name": ("package", "name"),
           "bioconda__version": ("package", "version"),
           .
           .
       },
      "biocontainers": {
           "biocontainers__name": ("name",),
           "biocontainers__identifiers": ("identifiers",),
           .
           .
       },
      .
      .
      }
      ```
      There is also another mapping (`PAGE_DATA_KEY_MAPPINGS`) for creating more data into each tool page.

      For `bioschemas.jsonld`, special parsing of the `@graph` array is handled to extract `"@type": "sc:SoftwareApplication"` entries.

  5. Finally, we have the combined JSON file with the structure shown in the following example, which would be later used by the frontend of our application:
      ```json
      [
        {
           "tool_name": "1000genomes",
           "contents": [
               "biotools",
               "bioschemas"
           ],
           "fetched_metadata": {
               "biotools__home": "http://www.internationalgenome.org",
               "biotools__summary": "The 1000 Genomes Project ran between 2008 and 2015, creating a deep catalogue of human genetic variation.",
               "biotools__addition_date": "2017-07-04T12:28:57Z",
               "biotools__last_update_date": "2022-06-30T08:53:55.709797Z",
               "biotools__tool_type": [
                   "Database portal",
                   "Web application"
               ],
               "bioschemas__name": "1000Genomes",
               "bioschemas__home": "https://bio.tools/1000genomes",
               "bioschemas__summary": "The 1000 Genomes Project ran between 2008 and 2015, creating a deep catalogue of human genetic variation.",
               "bioschemas__tool_type": "sc:SoftwareApplication"
           }
        },
        {
           "tool_name": "1000genomes_assembly_converter",
           "contents": [
               "biotools",
               "bioschemas"
           ],
           "fetched_metadata": {
               "biotools__home": "http://browser.1000genomes.org/tools.html",
               "biotools__summary": "Map your data to the current assembly.",
               "biotools__addition_date": "2015-01-29T15:47:08Z",
               "biotools__last_update_date": "2018-12-10T12:58:50Z",
               "biotools__tool_type": [
                   "Web application"
               ],
               "bioschemas__name": "1000Genomes assembly converter",
               "bioschemas__home": "https://bio.tools/1000genomes_assembly_converter",
               "bioschemas__summary": "Map your data to the current assembly.",
               "bioschemas__tool_type": "sc:SoftwareApplication"
           }
        },
      .
      .
      ]
      ```

### Frontend Project:
The user interface of the application, i.e. frontend of the project, is built using Nuxt.js and includes the following components:

- **Directory Structure**:
  ```
  frontend/
  ├── pages/               # Consists of web pages in our frontend application, like the home page to list and search tools (index.vue), and the tool description page with all the metadata of the tool (/tool/[id].vue)
  ├── plugins/             # Consists of Nuxt.js plugins used in the application. One used by our application is Vuetify, an open source UI library which provides Material UI components used in the application
  ├── public/              # Consists of metadata generated by our Python script, and the logo used in the application (logo-rsec.svg)
  ├── server/              # Consists of tsconfig.json, which specifies the root files and the compiler options for the Nuxt.js project
  ├── stores/              # We are using a Pinia store defined in tools.js, which helps to fetch the data from the JSON file and access the frontend of the application 
  ├── app.vue              # The web page which wraps our home page, it consists of header and footer for the application
  ├── nuxt.config.js       # Nuxt,js config file, which helps us to configure Vuetify, base URL of the project, and Pinia store for the project
  └── package.json         # Node.js configuration file used to manage different packages or dependencies used in our frontend application
  ```
- Working:
  1. On the home page, the user can browse through all the tools listed in the paginated window. It allows searching, sorting, and filtering of the data according to the user's criteria.
  2. Upon clicking on any tool from the home page, the user sees a tool details page containing all the metadata fetched from the JSON file.

> **Notes:**  Once the keys in the JSON or YAML files are renamed, or if the keys are to be removed or added, we have to modify the tools' details page.

