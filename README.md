# DropFakes Community Edition: Disposable Email Domain Database

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Update Frequency](https://img.shields.io/badge/database-updated%20monthly-brightgreen.svg)
![Domains](https://img.shields.io/badge/domains-10000%2B-orange.svg)

## Overview

DropFakes provides a comprehensive database of disposable and temporary email domains designed to safeguard your platform against automated account creation, spam, and platform abuse.

This repository contains the **Community Edition** of the DropFakes database (`blocked_domains.txt`). It serves as a static, foundational dataset containing approximately 10,000 historically active disposable domains, delayed by 60 days. This resource is provided to aid developers, researchers, and organizations in testing foundational security measures and offline development before migrating to a live production environment.

## The Threat Landscape: Why Scale Matters

The disposable email ecosystem is highly volatile. Fraudsters and automated bots constantly cycle through new, temporary domains to bypass static filters.

While the Community Edition provides a functional baseline of **~10,000 domains** for testing and integration architecture, it represents only a fraction of the active threat landscape. Organizations relying solely on static lists leave their systems vulnerable to hundreds of thousands of active threat vectors.

To achieve comprehensive protection, the **DropFakes Pro Edition** tracks and resolves **over 200,000 active disposable domains** in real-time, instantly neutralizing new threats the moment they are registered.

## Key Benefits

- **Proactive Risk Mitigation:** Identify and filter disposable email addresses during user onboarding, reducing the overhead of managing illegitimate accounts.
- **Development Flexibility:** Safely integrate and test email validation workflows in offline, staging, or CI environments without requiring external API calls.
- **Enhanced Data Integrity:** Maintain cleaner user databases, ensuring your communication resources reach authentic users.

## System Integration Guide

The `blocked_domains.txt` file should be loaded into memory at application startup using a **Hash Set** (or equivalent). This guarantees **O(1) time complexity** for all domain lookups — meaning email validation adds zero noticeable latency to your registration or authentication flows, regardless of traffic volume.

Implementation examples are provided below for Python, TypeScript, Go, Rust, and Java. Each follows the same pattern: parse line-by-line, lowercase and trim, store in a hash-based structure, check on demand.

### Python

```python
from typing import Set

def load_blocked_domains(filepath: str) -> Set[str]:
    """Loads domains into a memory-efficient Set for O(1) lookups."""
    with open(filepath, 'r', encoding='utf-8') as file:
        return {line.strip().lower() for line in file if line.strip()}

# Load once at startup
blocked_domains = load_blocked_domains('blocked_domains.txt')

def is_disposable(email: str) -> bool:
    domain = email.split('@')[-1].lower()
    return domain in blocked_domains  # O(1)
```

### TypeScript (Node.js)

```typescript
import * as fs from 'fs';

function loadBlockedDomains(filepath: string): Set<string> {
    // Load once at server startup
    const data = fs.readFileSync(filepath, 'utf-8');
    const domains = data.split('\n')
        .map(line => line.trim().toLowerCase())
        .filter(line => line.length > 0);

    return new Set(domains);
}

// Initialize once
const blockedDomains = loadBlockedDomains('blocked_domains.txt');

export function isDisposable(email: string): boolean {
    const domain = email.split('@').pop()?.toLowerCase() || '';
    return blockedDomains.has(domain);  // O(1)
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

// LoadBlockedDomains reads the file into a map for O(1) lookup performance.
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
			domains[domain] = struct{}{} // Zero-byte value; key existence = blocked
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
	_, exists := domains[domain] // O(1)
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
        return domains.contains(&domain.to_lowercase()); // O(1)
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
        // Load into a HashSet once at startup for O(1) lookups
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
        return blockedDomains.contains(domain); // O(1)
    }
}
```

## Community vs. Pro Edition

The Community Edition is a great starting point for side projects, internal tools, and early-stage products. When you need real-time protection at scale, the Pro Edition has you covered.

| Feature | Community Edition (This Repo) | Pro Edition (DropFakes.com) |
|---|---|---|
| **Database Size** | ~10,000 baseline domains | 200,000+ active domains |
| **Data Freshness** | 60-day delayed snapshot | Real-time resolution |
| **Update Frequency** | Bi-monthly static updates | Continuous, automated sync |
| **Threat Detection** | Historical | Immediate (newly registered domains) |
| **Data Maintenance** | Static list | Dynamic (auto-purges expired/re-parked domains) |
| **Support** | Community-driven | Priority support with SLA |

## Frequently Asked Questions

### How do I load blocked_domains.txt into my application?

Load it once at startup into a **Hash Set** (or hash map). Parse line-by-line, trim whitespace, lowercase every entry, and store in the set. All subsequent lookups are O(1) — the list size has no impact on validation speed. Examples for Python, TypeScript, Go, Rust, and Java are above.

### What's the difference between Community and Pro?

Community is a static `.txt` file with ~10,000 domains on a 60-day delay — great for development and early-stage use. Pro uses a live API and continuous data pipelines that monitor, verify, and resolve 200,000+ disposable domains in real-time, catching new threats as they appear.

### Why is my filter missing newly created disposable emails?

If you're on the Community Edition, you're checking against a 60-day snapshot. Fraud networks spin up new domains daily. A domain registered today will pass a static list but is immediately flagged by the DropFakes Pro real-time API.

### How do I extract the domain from an email for lookup?

Split on `@`, take the last segment, lowercase it, and check against your loaded set. Every example above demonstrates this pattern. The key is performing this once at startup so lookups stay O(1) at request time.

## Upgrading to Pro

When your project grows and you need coverage beyond the Community baseline, the Pro Edition gives you real-time data, 200,000+ domains, and continuous updates — built to scale from startup to production.

Visit [DropFakes.com](https://dropfakes.com) to get started.

## License

This project is licensed under the MIT License. Free to use, integrate, fork, and modify.

---

*For production deployments and priority support, visit DropFakes.com.*