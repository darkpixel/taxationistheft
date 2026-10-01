# taxationistheft.tax

Where Bob's $10,000 goes: a single-page walkthrough of a $10,000 invoice in Washington State, following every tax until the money lands in the owner's and the employee's bank accounts.

The whole site is `index.html`, served by nginx. See `CLAUDE.md` for the story, the numbers, the assumptions and deployment.

```sh
docker build -t taxationistheft .
docker run --rm -p 8080:8080 taxationistheft
# open http://localhost:8080
```
