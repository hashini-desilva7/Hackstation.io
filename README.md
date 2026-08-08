SQL Injection (SQLi) — VulnHub Write-up
📌 Overview

This write-up documents the exploitation of a SQL Injection vulnerability in a vulnerable web application.

The objective was to identify the SQL injection vulnerability, determine the number of columns returned by the application's query, enumerate the database structure, extract staff credentials, and use the discovered credentials to access the administrator account and retrieve the flag.

⚠️ Disclaimer: All techniques and payloads in this write-up were performed in an authorized VulnHub/lab environment for educational purposes.

🎯 Objectives
Identify a SQL Injection vulnerability
Determine the number of columns in the vulnerable query
Identify the application's database
Enumerate database tables
Identify the staff accounts table
Enumerate relevant columns
Extract the administrator's email address
Extract the administrator's password
Authenticate as the administrator
Capture the flag
Explore UNION-based, Boolean-based blind, time-based blind, and error-based SQL Injection
