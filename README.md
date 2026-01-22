# Literature Researcher AI Agent

## Overview

A multi-agent AI system for automated literature review and research paper discovery. This system uses CrewAI to coordinate specialized agents that search for, analyze, and synthesize academic papers on a given topic, generating comprehensive literature review reports.
All tools in this Projects are open-source and works on colab free CPU
## Features

- **Intelligent Search Query Generation**: Creates targeted search queries based on research topics
- **arXiv Paper Discovery**: Searches and retrieves relevant academic papers from arXiv
- **Automated Paper Analysis**: Extracts key information from paper PDFs including titles, authors, abstracts, and conclusions
- **Literature Review Generation**: Produces professional HTML reports with structured sections
- **Multi-Agent Workflow**: Sequential processing with specialized agents for each research stage

## Architecture

### Agents

1. **Search Queries Recommendation Agent**
   - Generates specific search queries for academic databases
   - Focuses on methods and technologies rather than general keywords
   - Produces up to 10 targeted search queries

2. **Researcher Agent (arXiv Search)**
   - Searches arXiv for relevant papers
   - Uses the ArxivPaperTool to fetch and download papers
   - Ranks papers based on relevance to search queries
   - Returns structured results with metadata

3. **Web Scraping Agent**
   - Extracts detailed information from paper PDFs
   - Uses ScrapeGraph.ai for intelligent PDF content extraction
   - Collects key paper details including abstract, conclusion, and keywords
   - Provides recommendation rankings and notes

4. **Literature Report Author Agent**
   - Generates professional HTML literature review reports
   - Uses Bootstrap CSS for responsive design
   - Structures reports with executive summary, methodology, findings, conclusion, and references

### Data Models

The system uses Pydantic models for structured data handling:

- **SuggestedSearchQueries**: List of search queries for paper discovery
- **SignleSearchResult**: Individual paper metadata from search results
- **SingleExtractedProduct**: Detailed paper information from PDF extraction
- **AllSearchResults/AllExtractedProducts**: Collections of results

## Installation

### Prerequisites

```bash
# Install required packages
pip install -qU crewai[tools,agentops]
pip install -qU scrapegraph-py
pip install litellm
```

### API Keys

The system requires the following API keys stored in Google Colab secrets:

1. **Mistral API Key** (`Mistral`): For LLM operations
2. **Serper API Key** (`serp`): For search operations
3. **ScrapeGraph API Key** (`scrapegraph`): For PDF content extraction

### Setup

```python
# Import required libraries
from crewai import Agent, Task, Crew, Process, LLM
from crewai.tools import tool
from google.colab import userdata
from pydantic import BaseModel, Field
from typing import List
from scrapegraph_py import Client
from crewai_tools import SerperDevTool, ArxivPaperTool
import os
import json

# Set up API keys
os.environ["MISTRAL_API_KEY"] = userdata.get('Mistral')
os.environ["SERPER_API_KEY"] = userdata.get('serp')

# Initialize components
output_dir = "./ai-agent-output"
os.makedirs(output_dir, exist_ok=True)
basic_llm = LLM(model="mistral/mistral-small-latest", temperature=0)
search_client = SerperDevTool()
scrape_client = Client(api_key=userdata.get('scrapegraph'))
```

## Usage

### Running the Complete Workflow

```python
# Initialize the crew with all agents
rankyx_crew = Crew(
    agents=[
        search_queries_recommendation_agent,
        search_engine_agent,
        scraping_agent,
        procurement_report_author_agent,
    ],
    tasks=[
        search_queries_recommendation_task,
        search_engine_task,
        scraping_task,
        procurement_report_author_task,
    ],
    process=Process.sequential
)

# Execute the workflow
crew_results = rankyx_crew.kickoff(
    inputs={
        "topic": "Your Research Topic",
        "no_keywords": 10,
        "language": "English",
        "score_th": 0.10,
        "top_recommendations_no": 5
    }
)
```

### Input Parameters

- **topic**: The research topic (e.g., "Arabic LLMs", "Transformer Neural Networks")
- **no_keywords**: Number of search queries to generate (default: 10)
- **language**: Language for papers (default: "English")
- **score_th**: Minimum relevance score threshold (default: 0.10)
- **top_recommendations_no**: Number of top papers to include in final report (default: 5)

## Output Structure

The system generates the following output files:

1. **`step_1_suggested_search_queries.json`**: Generated search queries
2. **`step_2_search_results.json`**: Initial search results from arXiv
3. **`step_3_search_results.json`**: Detailed paper information from PDF extraction
4. **`step_4_Literature_report.html`**: Final literature review report

## Example Output

### Search Queries Example
```json
{
  "queries": [
    "Arabic LLMs transformer architecture",
    "Arabic LLMs BERT",
    "Arabic LLMs fine-tuning",
    "Arabic LLMs pre-training",
    "Arabic LLMs multitask learning"
  ]
}
```

### Search Results Example
```json
{
  "results": [
    {
      "title": "A Tutorial about Random Neural Networks in Supervised Learning",
      "authors": "Sebastián Basterrech, Gerardo Rubino",
      "url": "https://arxiv.org/pdf/1609.04846v1",
      "summary": "Random Neural Networks (RNNs) are a class of Neural Networks...",
      "score": 0.95,
      "search_query": "transformer neural network"
    }
  ]
}
```

## Customization

### Modifying Search Parameters

Adjust the search query generation by modifying the agent's description:
```python
search_queries_recommendation_task = Task(
    description="Researcher is looking to do a literature review for {topic}...",
    # Modify constraints and requirements here
)
```

### Changing Output Format

Modify the Pydantic models to include different fields:
```python
class SingleExtractedProduct(BaseModel):
    # Add or remove fields as needed
    paper_doi: str = Field(None, title="DOI of the paper")
    publication_year: int = Field(None, title="Year of publication")
    # ... existing fields
```

### Adjusting Paper Sources

Replace the arXiv search with other academic databases:
```python
# Modify the search engine agent to use different tools
search_engine_agent = Agent(
    role="Researcher",
    goal="Find relevant academic papers",
    backstory="Expert at literature discovery",
    tools=[custom_search_tool],  # Add custom search tools
    # ... other parameters
)
```

## Error Handling

The system includes error handling for:

- **API Rate Limits**: Handles insufficient credits for scraping services
- **PDF Extraction Errors**: Gracefully handles malformed or inaccessible PDFs
- **Search Failures**: Continues processing with available results

## Limitations

1. **API Dependency**: Requires valid API keys for all services
2. **arXiv Focus**: Primarily searches arXiv; other databases require integration
3. **PDF Quality**: Extraction quality depends on PDF formatting and structure
4. **Credit Costs**: Scraping services may incur usage costs

## Future Enhancements

1. **Multi-Database Search**: Integrate PubMed, IEEE Xplore, and other academic databases
2. **Citation Analysis**: Include citation counts and impact metrics
3. **Trend Analysis**: Identify research trends and gaps over time
4. **Visualizations**: Add charts and graphs to literature reports
5. **Custom LLM Integration**: Support for various LLM providers beyond Mistral

## License

This project is intended for research and educational purposes. Commercial use may require additional licensing for API services and tools.

## Support

For issues and questions:
1. Check API key configurations
2. Verify internet connectivity in Colab environment
3. Ensure sufficient credits for scraping services
4. Review error messages in agent outputs

## Citation

If you use this system in your research, please acknowledge the tools and services:
- CrewAI for multi-agent orchestration
- arXiv for paper access
- Mistral for LLM services
- ScrapeGraph.ai for PDF extraction
