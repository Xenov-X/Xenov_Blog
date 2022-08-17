---
description: Some handy one liners for IP list parsing
---

# Large IP list handling

### Group IP list into /24s (CIDR)

Count number of unique IPs in list grouped by /24

```
cat IPs_list.txt | sort -u | awk 'BEGIN { FS = "." } ; { printf("%s.%s.%s/24\n", $1, $2, $3) }' | sort | uniq -c | sort -rn 
```

Count number of IPs in list grouped by /24 (not filtered for unique IPs). Useful if trying to understand location of high amounts of vHosting.

```
cat IP_list.txt | awk 'BEGIN { FS = "." } ; { printf("%s.%s.%s/24\n", $1, $2, $3) }' | sort | uniq -c | sort -rn
```

### Basic count of IPs

Count IPs in list to highlight common IPs, eg. if you've resolved many subdomains.

```
cat IP_list.txt | sort | uniq -c | sort -rn
```
