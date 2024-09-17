+++
title = 'Examples of Non-functional Requirements'
date = 2024-09-16T12:07:58-04:00
draft = false
+++

The common refrain when determining functional and nonfunctional requirements is that 
"functional requirements describe what the application does, and nonfunctional requirements 
describe how it does it."

Here are some examples of nonfunctional requirements:
- Performance
    - Latency of various actions (page loads, content loading, server processing)
- Availability
    - Uptime requirements (which may vary from system to system)
- Scalability
    - Number of users
    - Expected growth in users in 3 months, 6 months, a year, etc.
    - Throughput
    - Read vs Write heaviness (how many users read a post for each time it is written? 10:1? 50:1?)
- Observability
    - How easy it is to tell what is going on (or going wrong) with your application
- Reliability
- Technology Stack
- Security requirements
    - Authentication 
    - Authorization
- Maintainability
- Compliance
    - GDPR, COPPA, HIPAA, etc.


