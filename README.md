# ResearchMind — AI Research System

ResearchMind is a Python-based, multi-agent research assistant that searches the web, reads relevant sources, writes a structured report, and critiques the result. It provides both a polished Streamlit interface and a reusable Python pipeline for programmatic or command-line use.

> **Status:** Early-stage research prototype. Generated content should be reviewed against the original sources before being used for academic, professional, medical, legal, or other high-impact decisions.

## Features

- **Web search agent** — Finds recent information using Tavily and returns titles, URLs, and snippets.
- **Reader agent** — Selects a relevant result and extracts readable text from the source URL.
- **Writer chain** — Produces a professional report containing:
  - Introduction
  - At least three key findings
  - Conclusion
  - Source URLs found during research
- **Critic chain** — Reviews the generated report, assigns a score out of 10, identifies strengths and improvement areas, and provides a verdict.
- **Streamlit web interface** — Run research from a browser with pipeline status cards and expandable intermediate results.
- **Markdown export** — Download the final report as a `.md` file.
- **Reusable pipeline API** — Call `run_research_pipeline(topic)` from Python code or run the pipeline interactively from a terminal.

## How it works

The system processes each topic through four sequential stages:

```text
Research topic
     │
     ▼
1. Search Agent ── Tavily web search for recent information
     │
     ▼
2. Reader Agent ── Select and scrape a relevant source
     │
     ▼
3. Writer Chain ── Combine research into a structured report
     │
     ▼
4. Critic Chain ── Score and review the report
     │
     ▼
Report + critic feedback
```

The search and reader stages are LangChain agents with dedicated tools. The writer and critic stages are prompt chains using the configured OpenAI chat model.

## Architecture

| File | Responsibility |
| --- | --- |
| [`app.py`](app.py) | Streamlit UI, session state, pipeline execution, result rendering, and Markdown download. |
| [`agents.py`](agents.py) | OpenAI model configuration, search/reader agent factories, writer chain, and critic chain. |
| [`tools.py`](tools.py) | Tavily web-search tool and BeautifulSoup-based URL scraping tool. |
| [`pipeline.py`](pipeline.py) | Reusable four-stage Python pipeline and interactive command-line entry point. |
| [`requirements.txt`](requirements.txt) | Python dependencies for LangChain, OpenAI, Tavily, scraping, environment management, and supporting utilities. |
| [`.gitignore`](.gitignore) | Excludes the local `.env` file from version control. |

## Requirements

- Python **3.10+** recommended
- An OpenAI API key
- A Tavily API key
- Internet access for web search and source retrieval

The project uses:

- [LangChain](https://www.langchain.com/)
- [OpenAI](https://platform.openai.com/)
- [Tavily](https://tavily.com/)
- [Streamlit](https://streamlit.io/)
- [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/)

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/Zeny1303/AI-Research-System.git
cd AI-Research-System
```

### 2. Create and activate a virtual environment

#### macOS/Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
```

#### Windows PowerShell

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### 3. Install dependencies

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

The current dependency file does not list Streamlit explicitly, although `app.py` uses it. Install it separately if it is not already available in your environment:

```bash
pip install streamlit
```

## Configuration

Create a `.env` file in the repository root:

```dotenv
OPENAI_API_KEY=your_openai_api_key
TAVILY_API_KEY=your_tavily_api_key
```

The application loads these values with `python-dotenv`. Never commit `.env` or expose either key in source code, screenshots, logs, or public deployments.

The model is currently configured in `agents.py` as:

```text
OpenAI model: gpt-4o-mini
Temperature: 0
```

## Usage

### Streamlit interface

Start the browser-based application with:

```bash
streamlit run app.py
```

Then open the local URL printed by Streamlit, usually:

```text
http://localhost:8501
```

Enter a topic such as `quantum computing breakthroughs in 2025` and select **Run Research Pipeline**. The interface displays the search, reader, writer, and critic stages and lets you download the final report as Markdown.

### Python pipeline

The reusable pipeline returns a dictionary containing the intermediate and final outputs:

```python
from pipeline import run_research_pipeline

result = run_research_pipeline("Recent advances in fusion energy")

print(result["search_results"])
print(result["scraped_content"])
print(result["report"])
print(result["feedback"])
```

Returned keys:

| Key | Description |
| --- | --- |
| `search_results` | Tavily-backed search output with titles, URLs, and snippets. |
| `scraped_content` | Text extracted from the selected source URL. |
| `report` | The generated structured research report. |
| `feedback` | The critic's score, strengths, improvement areas, and verdict. |

### Interactive command line mode

Run the pipeline directly and enter a topic when prompted:

```bash
python pipeline.py
```

## Example topics

- `Large language model agents in 2025`
- `CRISPR gene editing developments`
- `Fusion energy progress`
- `The current state of quantum computing`
- `Renewable energy storage technologies`

## Tool behavior and limitations

### Web search

`web_search` requests up to five Tavily results and returns each result's title, URL, and a shortened content snippet.

### URL scraping

`scrape_url` uses `requests` and BeautifulSoup to fetch a page, removes `script`, `style`, `nav`, and `footer` elements, and limits extracted text to approximately 3,000 characters. Some sites may block automated requests, require JavaScript, or expose incomplete content.

### Research quality

- Search results and scraped pages can be incomplete, outdated, biased, or incorrect.
- The writer may state conclusions that are not fully supported by the available excerpts.
- The critic evaluates the generated report but does not independently verify every claim.
- Source URLs should be opened and checked manually before relying on the report.
- Do not submit confidential, personal, or sensitive information as a research topic.

### Operational considerations

- Every run makes external API and web requests and may incur provider costs.
- API rate limits, network failures, invalid URLs, robots restrictions, and provider outages can interrupt a run.
- The current UI stores results in Streamlit session state; it does not provide persistent report storage or user authentication.
- The application currently runs the stages sequentially rather than in parallel.

## Troubleshooting

### `ModuleNotFoundError: No module named 'streamlit'`

Install Streamlit explicitly:

```bash
pip install streamlit
```

### Missing API key errors

Check that `.env` is located in the project root and contains both variables:

```dotenv
OPENAI_API_KEY=your_openai_api_key
TAVILY_API_KEY=your_tavily_api_key
```

Restart the Streamlit process after changing environment variables.

### Scraping returns little or no content

The target site may require JavaScript, block automated requests, or provide content in a layout not handled by the basic scraper. Try another source URL or inspect the page manually.

### OpenAI or Tavily request failures

Confirm that the keys are valid, the accounts have available quota, and the machine has internet access. Also check provider status pages and rate limits.

## Development notes

The main extension points are:

- Add additional research tools in `tools.py`.
- Create specialized agents in `agents.py`.
- Improve report structure and fact-checking prompts in the writer and critic chains.
- Add persistence, citation extraction, retries, caching, or source validation to `pipeline.py`.
- Add automated tests for tools, pipeline state, prompt output, and failure handling.

Before deploying publicly, consider adding:

- Explicit HTTP status checking and stronger URL validation.
- Retry and timeout policies for API and scraping calls.
- Citation verification and duplicate-source handling.
- Structured output schemas for reports and critic feedback.
- Secret management appropriate to the hosting platform.
- Usage limits and authentication for public access.

## Security

- Keep API keys in environment variables or a managed secret store.
- Do not commit `.env` files.
- Treat scraped pages as untrusted input.
- Review generated reports for prompt injection, unsupported claims, and malicious links.
- Avoid sending private or regulated data to third-party model and search providers.

## Contributing

Contributions are welcome. A typical workflow is:

1. Fork the repository.
2. Create a feature branch.
3. Make a focused change.
4. Run the application or relevant checks locally.
5. Update the documentation when behavior changes.
6. Open a pull request describing the change and its validation.

## License

No license file is currently included in the repository. Until a license is added, the code should not be assumed to be available for unrestricted reuse, modification, or redistribution.

## Acknowledgements

Built with the LangChain ecosystem, OpenAI, Tavily, Streamlit, Requests, and BeautifulSoup.
