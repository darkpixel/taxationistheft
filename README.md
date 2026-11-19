# taxationistheft.tax

Where Bob's $10,000 goes: a single-page walkthrough of a $10,000 invoice in Washington State, following every tax until the money lands in the owner's and the employee's bank accounts.

See it live at [taxationistheft.tax](https://taxationistheft.tax).

## Help make it accurate

The point of this page is to show what really happens to a paycheck, so the numbers need to hold up. If you spot a problem, please [open an issue](https://github.com/darkpixel/taxationistheft/issues). Useful reports include:

- **A bug in the data:** a wrong rate, a math error, or a figure that doesn't match its source. Say which line is wrong, what it should be, and link a source if you have one.
- **A missing tax:** something that comes out of this money that the page doesn't show. Say who collects it, the rate, and whether it applies to the business, the employee, or both.
- **A suggestion:** clearer wording, a better explanation, a design fix, or an idea for something new.

Pull requests are welcome too.

## Running it locally

The whole site is `index.html`, served by nginx. See `CLAUDE.md` for the story, the numbers and the assumptions.

```sh
docker build -t taxationistheft .
docker run --rm -p 8080:8080 taxationistheft
# open http://localhost:8080
```
