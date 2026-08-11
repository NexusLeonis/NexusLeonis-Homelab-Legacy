# Historical Service Inventory

This is a sanitized inventory reconstructed from the original NexusLeonis documentation. The environment changed over time, so this should be read as a **historical service catalog**, not a claim that every item below was running simultaneously for the entire project.

The most complete January reference documented roughly 30 service entries. Some were placeholders or deployed but not fully configured.

## Infrastructure

|Service|Role|State|
|-|-|-|
|Portainer|Docker/container management|Running|
|Nginx Proxy Manager|Reverse proxy management|Running|
|Watchtower|Automated container update checks|Running|

## Monitoring / Network

|Service|Role|State|
|-|-|-|
|Homepage|Central service dashboard with widgets|Running|
|Uptime Kuma|Service monitoring and downtime alerts|Running|
|AdGuard Home|DNS-based filtering / network DNS|Running; configuration matured over time|
|Speedtest Tracker|Internet performance monitoring|Running|
|Glances|System/resource monitoring on secondary Linux host|Running on Mimir|

## AI / Search

|Service|Role|State|
|-|-|-|
|Open WebUI|Local AI chat interface|Running|
|SearXNG|Private metasearch|Running|
|NOMAD|Local AI-related workload/tooling|Running on Mimir|
|Ollama|Local LLM inference|Running on Windows Machine in WSL2 and Mimir|

## Utilities / Documentation

|Service|Role|State|
|-|-|-|
|Filebrowser|Web-based file/config management|Running|
|Syncthing|File synchronization|Running|
|BookStack|Documentation/wiki|Running|
|DokuWiki|Documentation/wiki|Running|
|Hoarder / Karakeep|Bookmark/archive management|Running|
|Mealie|Recipe management and scraping|Running|
|ArchiveBox|Web archival|Running on Mimir|
|MeTube|Media download utility|Running on Mimir|

## Automation / Home

|Service|Role|State|
|-|-|-|
|n8n|Workflow automation|Running|
|Home Assistant|Home automation|Running|

## Media Servers

|Service|Role|State|
|-|-|-|
|Jellyfin|Open-source media server|Running|
|Plex|Media streaming server|Running|
|Storyteller Reader|Audiobook/e-book library with AI transcription and alignment services|Running|
|Navidrome|Music server|Placeholder; service launched, never configured|

## Media Management / Automation

|Service|Role|State|
|-|-|-|
|Overseerr|Media requests and discovery|Running|
|Sonarr|TV automation|Running|
|Radarr|Movie automation|Running|
|Lidarr|Music automation|Running|
|Bazarr|Subtitle management|Running|
|Prowlarr|Indexer management|Running|
|Ombi|Additional request workflow|Placeholder; service launched, never configured|
|Readarr|E-book/audiobook automation|Placeholder; service launched, never configured|

## Remote Connectivity

|Service|Role|State|
|-|-|-|
|Tailscale|Private mesh VPN / remote access|Deployed across multiple devices; topology evolved over time|



