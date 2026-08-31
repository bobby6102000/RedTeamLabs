# Beginner CTF Flow

This diagram shows the intended first-time learner journey for the Northbridge College Portal lab.

```mermaid
flowchart TD
    A["College homepage"] --> B["Find the unlinked /admin page"]
    B --> C["Flag 0: admin portal found"]
    C --> D["Find the exposed training file"]
    D --> E["Flag 1: fake lab password found"]
    E --> F["Sign in as the fictional department clerk"]
    F --> G["Explore the limited student-record view"]
    G --> H["Flag 2: understand the normal record limit"]
    H --> I["Test the vulnerable record search"]
    I --> J["Flag 3: complete fictional record found"]
    J --> K["Submit flags and explain impact and remediation"]
```

## Teaching sequence

1. A hidden route is not access control.
2. Web-accessible files must not contain secrets.
3. User input must be handled safely in database searches.
4. Findings should be documented with their impact and a remediation recommendation.

All flags, records, and credentials in this exercise are fictional training data inside the isolated lab VM.
