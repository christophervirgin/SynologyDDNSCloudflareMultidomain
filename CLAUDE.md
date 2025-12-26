# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Synology DDNS agent that updates Cloudflare DNS records (A and AAAA) when IP addresses change. Designed to be installed on Synology DSM/SRM devices and integrates with their built-in DDNS system.

## Commands

### Run Tests
```bash
./run_tests.sh
# Or directly:
composer install && vendor/bin/phpunit tests
```

### Debug Script Locally
```bash
/usr/bin/php -d open_basedir=/usr/syno/bin/ddns -f cloudflare.php "domain1.com|vpn.domain2.com" "cloudflare-api-token" "" "your-ip-address"
```

## Architecture

The entire application is in `cloudflare.php` - a single self-contained PHP script with no external dependencies (designed for Synology's constrained PHP environment).

### Key Classes

- **SynologyCloudflareDDNSAgent**: Main orchestrator. Takes hostnames (pipe-separated), API token, and IPv4. Auto-detects IPv6 via ipify.org. Matches hostnames to Cloudflare zones and updates DNS records.
- **CloudflareAPI**: HTTP client wrapper for Cloudflare API v4 (zones, DNS records, token verification)
- **Ipify**: IPv6 detection via api6.ipify.org
- **DnsRecordEntity**: Data structure for DNS record state
- **SynologyOutput**: Constants for Synology DDNS status codes (good, badauth, nohost, etc.)

### Entry Point Flow

1. Script receives 5 arguments from Synology (username=hostnames, password=API token, IPv4)
2. Validates API token with Cloudflare
3. Extracts and validates hostnames from pipe-separated input
4. Fetches all zones from Cloudflare account
5. Matches hostnames to zones (supports subdomains like `sub.example.com` matching zone `example.com`)
6. Creates DNS record entities for IPv4 (A) and IPv6 (AAAA if available)
7. Fetches existing DNS records to get IDs and current IPs
8. Updates only records where IP has changed
9. Outputs Synology status code

### Installation Target

Script installs to `/usr/syno/bin/ddns/cloudflare.php` and registers in `/etc.defaults/ddns_provider.conf`.

## Code Style

PSR-12 PHP conventions. Keep dependencies at zero for Synology compatibility.
