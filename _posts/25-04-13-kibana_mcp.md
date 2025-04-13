---
title: "Kibana MCP"
image: "assets/images/posts/25-04_kibana_mcp.gif"
---
I used a combination of Cursor, Gemini 2.5, and Claude Desktop to create a simple FastMCP server that would give an appropriately configured MCP client the ability to take some simple actions in Kibana.

The most time consuming element of this in the end was having an appropriate test environment that could be spun up with a single command. I needed a Kibana/Elasticsearch instance where the Kibana system account could authenticate with Elasticsearch, and Elasticsearch was pre-populated with some data, a detection rule, and as a result an alert. Cursor really proved its worth here by managing to get into a beautiful feedback loop where it would run the initialising bash script, inspect the logs from Kibana STDOUT to evaluate if everything had gone as planned, tear down, and start again until the errors had been resolved. Repeatedly tearing down and spinning up big Java docker containers did make my laptop a little hot.

I was hoping to use Cursor as my MCP client, however the current means by which server and client talk is STDIO. Cursor doesn't capture this, and provides some very opaque debug messages when the server fails to start. This was so frustrating that I tried to use Claude Desktop instead. Claude writes errors from STDIO to a specific logfile for the purpose of this kind of debugging which was super helpful.

Wishlist going forwards:
- Tools for detections engineering: Exception editing, detection editing
- Case management: Create cases for suspicious alerts for human analyst to review
- Expand test env. More detections, more dummy data

<a href="https://github.com/ggilligan12/kibana-mcp">https://github.com/ggilligan12/kibana-mcp</a>