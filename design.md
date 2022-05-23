# Requirements

1) Automatic recording of GNUCash transactions (CSV file) from a specifed account in
- Current account to expense accounts
- Current account to investment accounts
- Current account to investement accounts of type mutual fund and shares with ability to fetch NAV/share price for the day of transaction
- Current account to current accounts
- Liability account to expense accounts

2) Resolve destination account by fuzzy matching transaction description
3) Resolve destination account by matching transaction value and description
4) When resolving the transactions to mutual funds, account for stamp duty and taxes
5) Reconcilliation status of transaction should be set to `n`

# Design

## Resolving destination account

A mapping between the transaction description and destination account is required to match a transaction to its destination account.
GNUcash accounts can be are annotated with meta data which can help resolve the transactions. The annotations can be stored as account 
descriptions in GNUcash account. The annotations can be stored in FrontMatter format.
A reverse search index is built from the keywords in the annotations which can be used to resovle the destination account from the keywords
in the transaction description.

### Sample annotations
A simple annotation for `Expenses:Eating Out` account describing the keywords to match transaction description and an optional
allow list of the souce accounts.
```
---
account_source: Assests:CurrentAssets:Savings:BankFoo, Liabilities:CreditCards:BankBaz
keywords: chats, dinner, breakfast, snacks
---
```
Sample annotation for a recurring deposit account
```
---
account_source: Assets:CurrentsAssets:Savings:BankFoo
currency: INR  ## TODO: Check if this info is already present as part of GNUCash account
keywords: fund, contribution
value_min: 2500
value_max: 2500
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
account_source: Assets:CurrentAssets:Savings:BankFoo
keywords: education
---

# Annotation for Assets:Investments:Educations Goal:Mutual Funds: FundFoo
---
account_source: Assets:CurrentAssets:Savings:BankBaz
keywords: education
---
```
### Rationale for FrontMatter format
Metadata could be stored in other popular formats like JSON or YAML too but it has its limitations. First of all, we are repurposing a field in GNUcash Account model which it was not desgined for. The `description` field should still allow adding description along with metadata.
FrontMatter supports metadata followed by free text which can be used for adding description.
If one doesn't want to use `gnucash-cmd` tool and not have any configs related to the tool in the GNUCash account description, they have to just remove the FrontMatter content.


### Resolving accounts at parent and children level
```
Expenses:Groceries:Misc
Expenses:Medical:Misc
Expenses:Medical:Self
Expenses:Medical:Spouse
Expenses:Medical:Parents
Expenses:Medical:Kids
Expenses:Gifts:Kids
Expenses:Gits:Misc
Expenses:Gifts:Spouse
Expenses:Gifts:Family and Friends
Expenses:Gifts:Parents
```
> TODO: design for account resolution with nested accounts (with similar leaf account names)
