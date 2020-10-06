# theHarvester
## Description
A tool for open source intelligence gathering and helping to determine a company's external threat landscape on the internet. This tool gathers emails, names, subdomins, IPs, and URLs using multiple public data sources.
* Perform non-intrusive (or intrustive) recon
* If used too much, *google will block your IP*
* You can use the harvester to search for linkedin users or twitter users related to a domain
## Common Commands
`theHarvester <options>`
## Options
`-d <domain>`
`-l <limit number of results>`
`-b <search engines to use seperated by commas>`
`-c` Will brute force common domain names to find hidden domains
`-n` Will do a DNS lookup to ensure you are finding what is hosting each server
## Example Executions
`theHarvester -d nasa.gov -l 500 -b google,bing,yahoo,duckduckgo`
`theHarvester -d globomantics.com -c`
* This will brute force common subdomains to find ones used by this domain