### My Methodology for Recon:

1. **Starting with Subdomain Enumeration:**
   - I use tools like: **subfinder**, **sublist3r**, **amass**, **assetfinder**, **crt.sh**.
   - Command to sort and filter out duplicates:
     ```bash
     subfinder -d target.com | sublist3r -d target.com -o - | amass enum -passive -d target.com | assetfinder --subs-only target.com | sort -u > subdomains.txt
     ```

2. **Next, I move on to using the httpx tool to bring all the active subdomains:**
   - Command to get active subdomains:
     ```bash
     cat subdomains.txt | httpx -ports 443,80,8080,8000,8888 -status-code -mc 200,403,400,500 | sort -k2 | uniq | tee httpx_results.txt
     ```

3. **Then, I use waybackurls, katana, and gau to gather all the parameters, URLs, old URLs, and filter them:**
   - Command to gather and filter URLs:
     ```bash
     cat httpx_results.txt | waybackurls; cat httpx_results.txt | gau; cat httpx_results.txt | katana -silent -jc -d 5) | httpx -ports 443,80,8080,8000,8888 -status-code -mc 200,403,400,500 | sort -k2 | uniq | tee cleanurls.txt
     ```

4. **Next, I grep for JavaScript URLs and save them using the following command:**
   - Command to extract JS URLs:
     ```bash
     cat cleanurls.txt | grep ".js" | sort -u | tee js_urls.txt
     ```

5. **Afterward, I use automation tools like:**
   - **Nuclei**, **Dalfox**, **sqlmap**, **secretfinder** for JS URLs.

6. **Then, I start manual hunting and try to grep all parameters that may be vulnerable:**
   - Command to find parameters:
     ```bash
     cat cleanurls.txt | grep "?"
     ```

7. **I use gf-patterns tools to bring only the subs I want to test. For example:**
   - Command for XSS:
     ```bash
     cat cleanurls.txt | gf xss | tee xss_params.txt
     ```
   - Command for SQLi:
     ```bash
     cat cleanurls.txt | gf sqli | tee sqli_params.txt
     ```
   - Command for LFI:
     ```bash
     cat cleanurls.txt | gf lfi | tee lfi_params.txt
     ```

8. **I start checking each category one by one, looking for vulnerabilities such as XSS, LFI, SSRF, SQLi, Command Injection, and Path Traversal.**
   - I inject each parameter with a payload for one of these vulnerabilities. If there is any CSP, I try to bypass it. If I can't, I move on to the next parameter.

9. **To gather important extensions and save them in one file:**
   - Command to extract extensions:
     ```bash
     cat urls.txt | grep -Eo '\.(php|aspx|jsp|json|zip|tar|bak|log|conf|ini|sql|xml|csv|yaml|env|config)(\?|$)' | sort -u > extensions.txt
     ```

10. **I don't forget the Arjun tool to search for hidden parameters in the extensions.txt file:**
    - Command to run Arjun:
      ```bash
      arjun -i urls.txt -o arjun_results.txt
      ```

11. **After all that, I fire up Burp Suite to look for:**
    - File Upload, BAC, IDOR, Business Logic, Race Condition, OTP Bypass, and any other vulnerabilities that you couldn't find with tools or that require deep thinking and patience.

12. **If I encounter 403, 400, or 500 status code pages, I move on to fuzzing. I only use dirsearch for fuzzing:**
    - Command to run fuzzing:
      ```bash
      dirsearch -u target.com
      ```

13. **I have a lot of things in my mind, but I am tired. Perhaps one day I can write my complete and ultimate methodology.**
