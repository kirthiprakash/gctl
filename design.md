# Requirements

1) Automatic recording of GNUCash transactions (CSV file) from a specifed account in
- Current account to expense accounts
- Current account to investment accounts
- Current account to investement accounts of type mutual fund and shares with ability to fetch NAV/share price for the day of transaction
- Current account to current accounts
- Liability account to expense accounts

2) Resolve destination account by fuzzy matching transaction description
3) Resolve destination account by matching transaction amount and description

# Design

## Resolving destination account

A mapping between the transaction description and destination account is required to match a transaction to its destination account.
GNUcash accounts can be are annotated with meta data which can help resolve the transactions. The annotations can be stored as account 
descriptions in GNUcash account. The annotations can be stored either in FrontMatter, Yaml or JSON.
A reverse search index is built from the keywords in the annotations which can be used to resovle the destination account from the keywords
in the transaction description.

### Sample annotations
A simple annotation for `Expenses:Eating Out` account describing the keywords to match transaction description and an optional
allow list of the souce accounts.
```
---
keywords: chats, dinner, breakfast, snacks
account_source: Assests:CurrentAssets:Savings:BankFoo, Liabilities:CreditCards:BankBaz
---
```
If a transaction gets resolved to two or more destination accounts, the allow list can be used to further filter the results.
For example, there could be a dedicated savings accounts from where all the investment transactions happen.
This scenario involves two transations and the descriptions for both can be same.
```
Savings Account (BankFoo) -> Savings Account (BankBaz) -> Investment

01/01/1970 | Kids Education | Assets:CurrentAssets:Savings:BankFoo -> Assets:CurrentAssets:Savings:BankBaz
15/01/1970 | Kids Education | Assets:CurrentAssets:Savings:BankBaz -> Assets:Investments:Education Goal:Mutual Funds: FundFoo
```
The annotations for these accounts
```
# Annotation for Assests:CurrentAssets:Savings:BankBaz
---
keywords: education
account_source: Assets:CurrentAssets:Savings:BankFoo
---

# Annotation for Assets:Investments:Educations Goal:Mutual Funds: FundFoo
---
keywords: education
account_source: Assets:CurrentAssets:Savings:BankBaz
---
```


