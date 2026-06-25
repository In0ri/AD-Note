## 1. Install Go Programming Language
```sudo apt install golang-go```
## 2. Install httpx Using Go
```
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
export PATH=$PATH:~/go/bin
source ~/.bashrc
```
## 3. Basic httpx Usage and Probing
```
httpx -u “example.com” -probe
httpx -l hosts.txt -probe
```
- Extracting Detailed Information: 
```
httpx -l hosts.txt -server
httpx -l hosts.txt -title -tech-detect -status-code -content-length -response-time
```
- Advanced Filtering and Matchers: 
```
httpx -l hosts.txt -sc -fc 301
```
- Filter “dead” or default/error responses: 
```
httpx -l hosts.txt -sc -fep
```
- Filter status code: 
```
httpx -l hosts.txt -status-code -match-code 200
```

