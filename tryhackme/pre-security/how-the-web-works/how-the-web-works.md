# TryHackMe — Pre-Security Path (Part 3: How The Web Works)

**Status:** Completed  
**Date Completed:** *16/11/2025*  
**Modules Covered:**  
- DNS in Detail  
- HTTP in Detail  
- How Websites Work  
- Putting It All Together  

---

### Objective
Gain a deep understanding of how the web functions behind the scenes — from DNS lookups to HTTP communication, webpage structure, and how servers deliver content. This module builds the foundation for analysing web traffic, spotting vulnerabilities, and understanding how websites operate at every layer.

---

### Key Takeaways

#### **Module 1 — DNS in Detail**
- Learned what DNS is and why it acts as the “phonebook of the internet.”  
- Understood the full **domain hierarchy** (root → TLD → domain → subdomain).  
- Explored common **record types** (A, AAAA, MX, CNAME, TXT).  
- Studied how DNS requests are made and resolved.  
- Completed practical exercises performing real DNS lookups.

#### **Module 2 — HTTP in Detail**
- Learned how HTTP/HTTPS enables communication between browsers and servers.  
- Understood the structure of **requests and responses**.  
- Explored key **HTTP methods** (GET, POST, PUT, DELETE, etc.).  
- Learned how to interpret **status codes** (2xx success, 3xx redirects, 4xx client errors, 5xx server errors).  
- Studied **headers** and their role in authentication, content type, caching, and security.  
- Learned how **cookies** store session data and how they impact security.  
- Practiced making HTTP requests manually.

#### **Module 3 — How Websites Work**
- Learned the complete lifecycle of loading a webpage.  
- Understood the roles of **HTML**, **CSS**, and **JavaScript** in building websites.  
- Studied how client-side scripts work.  
- Identified web risks like **Sensitive Data Exposure** and why insecure coding exposes information.  
- Practiced **HTML Injection**, gaining insight into how client-side vulnerabilities arise.

#### **Module 4 — Putting It All Together**
- Brought together all concepts learned (DNS → HTTP → web rendering).  
- Explored additional components like CDNs, caching layers, and load balancers.  
- Learned how web servers handle requests and deliver responses.  
- Completed a final quiz consolidating the full module.

---

### Reflection
This module helped me understand exactly how websites function from the ground up. I now know how DNS resolves domains, how HTTP transfers data, how webpages are built, and how servers respond behind the scenes. This knowledge is essential for cybersecurity work — especially for analysing logs, spotting vulnerabilities, understanding attacks, and interpreting network traffic.

---

### Evidence
Screenshots demonstrating the hands-on practical work completed in this module are stored in the **`evidence/`** folder. These include:

- **DNS Lookup Practical**  
  - Performing CNAME, TXT, MX and A record lookups (`nslookup` simulator)  
  - *File:* `Practical - DNS in Detail - How The Web Works - Module 3.png`

- **HTTP Requests Practical**  
  - Making GET, POST, PUT and DELETE requests and analysing server responses  
  - *File:* `Making Requests - HTTP in Detail - How The Web Works - Module 3.png`

- **Sensitive Data Exposure**  
  - Viewing leaked credentials in the page’s source code  
  - *File:* `Sensitive Data Exposure - How Websites Work - How The Web Works.png`

- **HTML Injection**  
  - Injecting malicious HTML into an unsanitised input field  
  - *File:* `HTML Injection - How Websites Work - How The Web Works.png`

These screenshots provide evidence of interacting with DNS tools, crafting HTTP requests, analysing responses, inspecting source code, and exploiting client-side vulnerabilities.

---

### Next Step
Progressing to **Pre-Security Part 4 — Linux Fundamentals**, while also moving onto **Google Cybersecurity Certificate (Course 3 — Connect and Protect: Networks and Network Security).**
