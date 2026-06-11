---
title: "Enabling a Language Model to Search the Internet"
---

# Enabling a Language Model to Search the Internet  

Refer to my previous post about giving a language model access to a calculator for the main idea of how MCP works: [link](https://bentay85.github.io/2025/08/27/mcp.html)  

In this post, I wanted to give the Language Model the capability to search the internet. This can be done by making a API call to Google Search and passing the relevant fields in the search results to the Language Model. See [link](https://developers.google.com/custom-search/v1/overview) for documentation on how to get a Google API Key and a Google Custom Search Engine ID. This is completely free to use, up to 100 searches a day. We will set the GOOGLE_API_KEY and the GOOGLE_CSE_ID up as Environment Variables so that we do not hardcode these into our Python script.  

```python
import os
import json
from googleapiclient.discovery import build
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("google-search-server")

api_key = os.environ.get("GOOGLE_API_KEY")
cse_id = os.environ.get("GOOGLE_CSE_ID")

@mcp.tool(description="Performs a Google search and returns a JSON array of the top 3 results with title, link, and snippet fields.")
def google_search(query: str) -> list:
    if not api_key or not cse_id:
        return [{"error": "GOOGLE_API_KEY or GOOGLE_CSE_ID environment variables are not set."}]
    
    try:
        service = build("customsearch", "v1", developerKey=api_key)
        
        result = service.cse().list(
            q=query,
            cx=cse_id,
            num=3
        ).execute()

        if 'items' not in result:
            return [{"error": f"No results found for the query: '{query}'."}]

        formatted_results = []
        for item in result['items']:
            formatted_results.append({
                "title": item.get('title', 'No Title'),
                "link": item.get('link', 'No Link'),
                "snippet": item.get('snippet', 'No Snippet')
            })
            
        return formatted_results

    except Exception as e:
        return [{"error": f"An error occurred during the search: {str(e)}"}]

if __name__ == "__main__":
    if api_key and cse_id:
        mcp.run()
```  
mcp.json  
```json
{
  "mcpServers": {
    "google-search": {
      "command": "D:/mcp_search/.venv/Scripts/python.exe",
      "args": [
        "D:/mcp_search/main.py"
      ]
    }
  }
}
```  

System Prompt:  
```
You have access to a tool called google_search.

Input: { "query": "<search terms>" }

Output: JSON array of up to 3 results with title, link, and snippet.

Use this tool whenever the user asks something that requires a web search.
```  
