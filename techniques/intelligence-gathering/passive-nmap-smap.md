---
description: Port scanning without touching your target
---

# Passive nmap (smap)

{% embed url="https://github.com/s0md3v/Smap" %}

### SMAP

Smap is a useful tool that queries [https://internetdb.shodan.io/](https://internetdb.shodan.io/), a free API offered by Shodan. Its additional value is derived from the fact it can output data in nmap based formats, which can be ingested by other tools existing import options (E.g; Witnessme).

A slight issue exists in that every IP scanned is a separate GET request, so for scanning large ranges you quickly hit rate limiting resulting in unexpected output.

This can be bypassed by rotating your IPs. An easy way to do it is to set up an AWS API gateway pointing to the internetdb API, and patching out the URL within the smap binary.&#x20;

1. API Gateway&#x20;

For this you can utilise fireprox, a handy tool for quickly spinning up API gateways.

```
python3 fire.py --command create --url https://internetdb.shodan.io/
```

2\. Smap patching&#x20;

Modify the smap binary to use your API gateway - this URL is within \`shodan.go\`.

{% code title="shodan.go" %}
```go
func Query(ip string) []byte {
	url := "https://<randomstring>.execute-api.eu-west-2.amazonaws.com/fireprox/" + ip
	req, err := http.NewRequest("GET", url, nil)
	resp, err := client.Do(req)
```
{% endcode %}

3\. Scan without rate limits



### Filtering results to gain further insight

Often it can be challenging to filter useful information out of 1000s of IPs; one method I've found useful is categorising IPs into files based on the available ports. The following code will create a series of files for each open port filled with the IPs that have that port open.&#x20;

I find it makes unusual ports stand out a little easier, as one method for flagging up interesting targets.

```
grep -Po '(\d*)(?=/open)' output.gnmap | sort -u > openport.list;
for PORT in $(cat openport.list); do  awk -v port="${PORT}" "/${PORT}\/open/ {print \$2}" output.gnmap >> Live_${PORT}; done;
```

Or as a oneliner:

```
cat ip-ranges.gnmap| grep -Po '(\d*)(?=/open)' | sort -u | while read PORT ; do cat ip-ranges.gnmap | awk -v port="${PORT}" "/${PORT}\/open/ {print \$2}" >> Live_${PORT}; done

```
