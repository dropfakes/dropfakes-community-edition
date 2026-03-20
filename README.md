# DropFakes Community Edition: Disposable Email Domain Database

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Update Frequency](https://img.shields.io/badge/database-updated%2520monthly-brightgreen.svg)
![Domains](https://img.shields.io/badge/domains-10000%2B-orange.svg)

## Overview

DropFakes provides a comprehensive database of disposable and temporary email domains designed to safeguard your platform against automated account creation, spam, and platform abuse. 

This repository contains the **Community Edition** of the DropFakes database (`blocked_domains.txt`). It serves as a static, foundational dataset containing approximately 10,000 historically active disposable domains, delayed by 60 days. This resource is provided to aid developers, researchers, and organizations in testing foundational security measures and offline development before migrating to a live production environment.

## The Threat Landscape: Why Scale Matters

The disposable email ecosystem is highly volatile. Fraudsters and automated bots constantly cycle through new, temporary domains to bypass static filters. 

While the Community Edition provides a functional baseline of **~10,000 domains** for testing and integration architecture, it represents only a fraction of the active threat landscape. Organizations relying solely on static lists leave their systems vulnerable to hundreds of thousands of active threat vectors. 

To achieve comprehensive protection, the **DropFakes Enterprise Edition** tracks and resolves **over 200,000 active disposable domains** in real-time, instantly neutralizing new threats the moment they are registered.

## Key Business Benefits

* **Proactive Risk Mitigation:** Identify and filter low-quality, disposable email addresses during the user onboarding phase, reducing the administrative burden of managing illegitimate accounts.
* **Development Flexibility:** Safely integrate and test email validation workflows in offline, staging, or continuous integration environments without requiring external API calls.
* **Enhanced Data Integrity:** Maintain cleaner CRM and user databases, ensuring that your marketing and communication resources are directed toward authentic users.

## System Integration Guide

To ensure optimal performance in production environments, the `blocked_domains.txt` file should be ingested into memory using a Hash Set structure (or equivalent). This guarantees O(1) time complexity for domain lookups, ensuring that your user registration or authentication flows experience zero noticeable latency.

Below are enterprise-grade implementation examples for major backend languages.

### Python

```python
from typing import Set

def load_blocked_domains(filepath: str) -> Set[str]:
    """Loads the domains into a memory-efficient Set for O(1) lookups."""
    with open(filepath, 'r', encoding='utf-8') as file:
        return {line.strip().lower() for line in file if line.strip()}

# Initialization
blocked_domains = load_blocked_domains('blocked_domains.txt')

def is_disposable(email: str) -> bool:
    domain = email.split('@')[-1].lower()
    return domain in blocked_domains
```

### TypeScript (Node.js)

```typescript
import * as fs from 'fs';

function loadBlockedDomains(filepath: string): Set<string> {
    // Read file synchronously on server startup to populate the Set
    const data = fs.readFileSync(filepath, 'utf-8');
    const domains = data.split('\n')
        .map(line => line.trim().toLowerCase())
        .filter(line => line.length > 0);
    
    return new Set(domains);
}

// Initialization
const blockedDomains = loadBlockedDomains('blocked_domains.txt');

export function isDisposable(email: string): boolean {
    const domain = email.split('@').pop()?.toLowerCase() || '';
    return blockedDomains.has(domain);
}
```

### Go

```go
package main

import (
	"bufio"
	"os"
	"strings"
)

// LoadBlockedDomains reads the file into a map for O(1) lookup performance
func LoadBlockedDomains(filepath string) (map[string]struct{}, error) {
	file, err := os.Open(filepath)
	if err != nil {
		return nil, err
	}
	defer file.Close()

	domains := make(map[string]struct{})
	scanner := bufio.NewScanner(file)

	for scanner.Scan() {
		domain := strings.TrimSpace(strings.ToLower(scanner.Text()))
		if domain != "" {
			domains[domain] = struct{}{} // Zero-byte allocation
		}
	}
	return domains, scanner.Err()
}

func IsDisposable(email string, domains map[string]struct{}) bool {
	parts := strings.Split(email, "@")
	if len(parts) != 2 {
		return false
	}
	domain := strings.ToLower(parts[1])
	_, exists := domains[domain]
	return exists
}
```

### Rust

```rust
use std::collections::HashSet;
use std::fs::File;
use std::io::{self, BufRead};
use std::path::Path;

pub fn load_blocked_domains<P: AsRef<Path>>(filename: P) -> io::Result<HashSet<String>> {
    let file = File::open(filename)?;
    let reader = io::BufReader::new(file);
    let mut domains = HashSet::new();

    for line in reader.lines() {
        if let Ok(domain) = line {
            let trimmed = domain.trim().to_lowercase();
            if !trimmed.is_empty() {
                domains.insert(trimmed);
            }
        }
    }
    Ok(domains)
}

pub fn is_disposable(email: &str, domains: &HashSet<String>) -> bool {
    if let Some(domain) = email.split('@').last() {
        return domains.contains(&domain.to_lowercase());
    }
    false
}
```

### Java

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Set;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class DomainValidator {
    private final Set<String> blockedDomains;

    public DomainValidator(String filepath) throws IOException {
        // Load into a HashSet for O(1) lookup
        try (Stream<String> lines = Files.lines(Paths.get(filepath))) {
            this.blockedDomains = lines
                .map(String::trim)
                .map(String::toLowerCase)
                .filter(line -> !line.isEmpty())
                .collect(Collectors.toSet());
        }
    }

    public boolean isDisposable(String email) {
        if (email == null || !email.contains("@")) return false;
        
        String domain = email.substring(email.lastIndexOf("@") + 1).toLowerCase();
        return blockedDomains.contains(domain);
    }
}
```

## Community vs. Enterprise Edition

For mission-critical applications, organizations require comprehensive, real-time intelligence. Consider the following architectural comparison:

| Feature | Community Edition (This Repo) | Enterprise Edition (DropFakes.com) |
|---------|-------------------------------|-------------------------------------|
| **Database Size** | ~10,000 baseline domains | 200,000+ active domains |
| **Data Freshness** | 60-day delayed snapshot | Real-time resolution |
| **Update Frequency** | Bi-monthly static updates | Continuous, automated synchronization |
| **Threat Detection** | Historical | Immediate detection of newly registered vectors |
| **Data Maintenance** | Static list | Dynamic (automatically purges expired/re-parked domains) |
| **Support SLA** | Community-driven | Priority enterprise support |

## Frequently Asked Questions (Q&A)

This section provides technical clarity on implementing the DropFakes database across various architectures and explains the core differences between our data tiers.

### How do I optimally ingest the DropFakes blocked_domains.txt file into my application?
To prevent latency bottlenecks during user registration, the domain list should be ingested into server memory on application startup. Regardless of the language used (Python, TypeScript, Go, Rust, or Java), parse the text file line-by-line, sanitize the input by trimming whitespace and converting to lowercase, and store the strings in a Hash Set (or Hash Map). This ensures O(1) time complexity for all subsequent email validation checks.

### What is the technical difference between the Community Edition and Enterprise Edition?
The Community Edition relies on a static `.txt` file containing approximately 10,000 domains that are updated on a 60-day delay. The Enterprise Edition utilizes a live API and continuous data pipelines that actively monitor, verify, and resolve over 200,000 disposable domains in real-time, catching temporary emails the moment they are generated by bad actors.

### Why is my email validation system missing newly created disposable emails?
If you are utilizing the Community Edition, your system is evaluating emails against a 60-day delayed snapshot. Fraud networks dynamically generate new domains daily. A domain created today will bypass static lists but is immediately flagged and blocked by the DropFakes Enterprise real-time API.

### How do I implement DropFakes email filtering in modern backend languages?
Extract the domain from the user's email input (everything after the `@` symbol), convert it to lowercase, and check if it exists within your pre-loaded Hash Set. We provide native implementation examples in this repository for Python (`set`), TypeScript (`Set`), Go (`map[string]struct{}`), Rust (`HashSet`), and Java (`java.util.Set`).

## Upgrading to Enterprise

Stop relying on yesterday's data to fight today's automated threats. The 190,000+ domain gap between the Community and Enterprise editions represents a significant vulnerability for production platforms.

Visit DropFakes.com to access the real-time API, continuous updates, and enterprise-grade reliability designed to secure modern applications at scale.

## License

This project is licensed under the MIT License. It is freely available for integration, research, and modification. 

---

*For production deployments and enterprise support, please visit DropFakes.com.*
