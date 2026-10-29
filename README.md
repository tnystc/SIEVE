# SIEVE
 
SIEVE is a gateway that sits between your applications and the LLM providers they call, and keeps personal data from leaving your network. You run it yourself, point your app at it instead of the provider, and it checks everything that goes out.
 
We are five computer engineering students at METU in Ankara, and this is our senior project, supervised by Prof. Dr. Ertan Onur. It is a work in progress. The list below is what we are building.
 
## Why we are building it
 
Companies are plugging LLMs into everything, and a lot of customer and employee data goes along for the ride. Sometimes it is a user pasting an IBAN into a prompt. More and more it is an agent calling a tool that returns a few hundred database rows, which then get forwarded to the model as context. Once that data is at the provider, you no longer control it, which is a problem under KVKK.
 
The tools we looked at are mostly built around English and US formats. A Turkish ID number (TCKN) or tax number (VKN) is easy to miss, and just as easy to confuse with an invoice number. We want something that handles Turkish identifiers properly, and handles English just as well.
 
## What it does
 
You change the base URL and API key in your application. That is the whole integration. From then on SIEVE:
 
- scans every part of the request, including system prompts, tool calls and tool results, for sensitive values: TCKN/YKN, VKN, Turkish IBAN, phone numbers, plates, payment cards, e-mail addresses, API keys and other secrets, and (optionally) person names
- validates identifiers with their checksums, so a random 11-digit number does not get flagged just because it looks like an ID
- applies a policy you write. Each finding can be allowed, warned about, redacted or blocked, depending on the category, the data source, the application and the provider. When something goes wrong inside SIEVE, the default is to block
- replaces redacted values with typed placeholders like `[IBAN_1]` before the request leaves your network, and puts the real values back in the response, including while it streams. Your users keep seeing real data, the provider never does
- writes an audit record for every decision without storing the sensitive values themselves
- gives administrators a web dashboard (with MFA) to watch events and to edit and test policies
It runs on your own infrastructure and starts with one Docker Compose command.
 
First we are doing OpenAI Chat Completions and Anthropic Messages. The OpenAI Responses API comes later, and Gemini may follow.
 
### MCP
 
In the second half of the project we are adding the same inspection for agent traffic over the Model Context Protocol:
 
- a drop-in wrapper for local (stdio) MCP servers
- a Streamable HTTP gateway for remote MCP servers
- field-level rules for structured tool results, for example mask the e-mail field and keep the customer ID
The MCP part talks to the rest of SIEVE through an internal inspection API and shares the placeholder mapping with the LLM side, so a value gets the same placeholder whichever way it travels. It has to cope with both the older MCP versions and the 2026-07-28 revision, which dropped the handshake and sessions.
 
## What it does not cover
 
SIEVE protects programmatic API traffic that goes through it. It does not protect:
 
- consumer chat apps where you cannot change the server address
- applications that call a provider directly. To close that gap you need an egress firewall that only lets the gateway reach the provider, and we will include a hardening guide for it
 
## How it is put together
 
SIEVE is one Go program: the request pipeline, the built-in detectors, the policy engine and the masking all live in the same process, so a request never crosses the network between them. Caddy handles TLS and PostgreSQL stores the audit trail, policies and keys. Application traffic and administration use separate ports, so you can isolate the admin side.
 
Detectors that need more than pattern matching run as separate services behind a small HTTP detector API with timeouts and failure modes. That is how we plan to plug in the optional name-detection service and third-party engines such as Presidio.
 
## Running it
 
Not ready yet. The goal is:
 
```
docker compose up
```
 
and nothing else. Once there is something to run, installation, configuration and the network-hardening guide will go in `docs/`.
 
## Testing
 
We use only synthetic personal data, and only synthetic data is ever sent to a real LLM API. Most tests run against a mock provider that replays recorded responses, and local models through Ollama cover the end-to-end runs.
 
Test data comes from checksum-valid generators that we check against independent implementations (python-stdnum, among others). We will publish an English and Turkish evaluation suite that reports precision, recall, false-positive rate and added latency per category, and we will run existing tools such as Presidio and LiteLLM with Presidio against the same data.
 
## Team
 
Senior computer engineering students Tanay Sütcü, Yiğit Polat, Arda Ehi, Ali Emre Yılmaz and Deniz Yılmaz from Middle East Technical University, supervised by Prof. Dr. Ertan Onur.
 

Claude finished the response
