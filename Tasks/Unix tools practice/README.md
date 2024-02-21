# Batch Processing with Uinx Tools

## Nginx access log

In this task I will use unix tools to manipulate the nginx access log and save the outputs here in different files

### 1- Top 5 visited pages

```bash
awk '{print $7}' access.log | sort | uniq -c | sort --parallel=$(nproc) -n -r | head -n 5 > "1- top visited pages.txt"
```

### 2- Top 5 visited pages except '*'

```bash
cat access.log | awk '$7 != "*" {print $7}' | sort | uniq -c | sort -n -r | head -n 5 > "2- top visited pages except '*'.txt"
```

### 3- Print the number of requests in the file

```bash
cat access.log | awk 'END {print NR}' > '3- number of requests.txt'
```

### 4- extract all unique ip addresses

```bash
cat access.log | awk '{print $1}' | sort | uniq -c > "4- unique IPs"
```

### 5- count of requests made to a specifing endpoint

```bash
 cat 'access.log' | awk '$7 == "/test.txt" {print $7}' | uniq -c > "5- count of requests to path.txt"
```

### 6- print all requests made to specific page

```bash
cat 'access.log' | grep '/test.txt' > '6- requests matching pattern.txt'
```

### -7 print all requests with status code 404

```bash
cat 'access.log' | awk '$9 == 404 {print}' > '7- 404 requests.txt'
```

### 8- Count of requests per HTTP method

```bash
cat 'access.log' | awk '{print $6}' | sort | uniq -c > "8- count of requests per HTTP method"
```

### 9- Request with largest size of byetes

```bash
 cat 'access.log' | awk '{print $10 " " $0}'| sort -n -r | head -n 1 > '9- request with largest size of bytes.txt'
```
