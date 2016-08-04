#Introduction

[![Build Status](https://travis-ci.org/bostontrader/bookwerx-core.svg?branch=master)](https://travis-ci.org/bostontrader/bookwerx-core)
[![MIT license](http://img.shields.io/badge/license-MIT-brightgreen.svg)](http://opensource.org/licenses/MIT)
[![Dependency Status](https://david-dm.org/bostontrader/bookwerx-core.svg)](https://david-dm.org/bostontrader/bookwerx-core)
[![devDependency Status](https://david-dm.org/bostontrader/bookwerx-core/dev-status.svg)](https://david-dm.org/bostontrader/bookwerx-core#info=devDependencies)

The purpose of **bookwerx-core** is to provide an API that supports multi-currency
 bookkeeping, using the double-entry bookkeeping model, slightly adapted to squeeze 
 in multiple currencies.  It uses [node](https://nodejs.org), [restify](http://restify.com/), and [mongodb](https://www.mongodb.com/).

Any application that deals with "money" (fiat, precious metals, cryptocoins) will
quickly encounter the need for bookkeeping.  Rolling your own methods is, as usual,
 easier said than done, so perhaps you can save yourself some grief and enjoy **bookwerx-core** instead.

With this API, the user can:

* Perform ordinary CRUD operations on the various bookkeeping objects,
such as accounts and transactions.

* Perform consistency checks.

* Brainwipe the db and start over.

This API is the minimum necessary to populate your db with bookkeeping information,
ensure that it is internally consistent, and nuke it from orbit and start over if necessary.

A package that provides a web-based front-end using Angular 2 can be found at [bookwerx-ui]
(https://github.com/bostontrader/bookwerx-ui) and for more sophisticated analysis, 
such as the production of financial reports and graphing, please see 
 [bookwerx-reporting](https://github.com/bostontrader/bookwerx-reporting).


##Getting Started

### Prerequisites

* You will need node and npm.

* You will need git.

* You will need mongodb.

* In order to run the example data importer, you will need Python and the requests library.
(docs.python-requests.org/en/master/user/install)

The care and feeding of these items are beyond the scope of these instructions.

### But assuming they are correctly installed...

```bash
git clone https://github.com/bostontrader/bookwerx-core.git
cd bookwerx-core
npm install
npm test
npm start
```
##Runtime Configuration

Runtime configuration is managed by [node-config](https://github.com/lorenwest/node-config)
By default, **bookwerx-core** will start the server using /config/default.json.
You may create other configurations to suit your fancy. And example to use configuration
/config/production.json:

```bash
export NODE_ENV=production
npm start
```
##Multiple Currencies

**bookwerx-core** enables the user to maintain a list of currencies relevant to their app.
Fiat, precious metals, and cryptocoins are three obvious examples of currencies that
this app could easily handle.

The "distributions" of a transaction (the debits and credits part) are all tagged
with a currency and a single transaction can include any number of different
currencies.

Any transaction which includes only a single currency should satisfy the usual
sum-of-debits = sum-of-credits constraint.

Any transaction which includes two currencies should still satisfy that constraint.
[But...](https://www.youtube.com/watch?v=FaVFuX8z26c) We can modify said constraint
a wee bit to make it fit. As you can readily imagine, the actual numbers involved
won't add up the way we ordinarily expect. Instead, you can compute an
implied exchange rate R such that sum-of-debits * R = sum-of-credits.

But be careful with this.  Although simple transactions such as currency exchanges 
can easily be recorded by debiting one currency and crediting the other, you
could easily make nonsensical transactions if you're not paying attention when you do this.

##Validation

Notice I say "you" can do this or that, not **bookwerx-core**. This is intentional and keeping
with the principal that this core is a minimalist thing.  Although there is a core of necessary 
referential integrity constaints that **bookwerx-core** enforces,
extra fancy features, such as validation belong elsewhere.  You may easily use this API to make
 non-sensensical entries into your records.  GIGO.

##Data Analysis

As mentioned earlier, this package merely records the basic bookkeeping objects.
More sophisticated analysis belongs in other packages such as
[bookwerx-reporting](https://github.com/bostontrader/bookwerx-reporting).  "What actually happened" (the transactions) belong here.
But "what does any of this mean" belongs elsewhere.


##API

Any errors returned will be JSON with a schema like {'error': {'some error message'}}
In the event that more than one error could apply, only the first error found
is returned.
Recall the earlier discussion of validation and its absence here.

Accounts

GET /accounts
Returns a JSON array of account documents.

GET /accounts/:id
Returns a JSON object containing a single account document.
Possibly error 1.

POST /accounts
Add a new account document. Returns what was just written, plus the newly assigned
_id.

PUT /accounts/:id
Modify an existing account, no upsert.
Possibly error 1.

DELETE /accounts/:id
Guess what this does?
Possibly return errors 1 or 2.

 
Possible errors:
1.  account n does not exist
2.  this account cannot be deleted because some distributions refer to it


currencies
GET /currencies get all of them
GET /currencies/:id get one of them (errors:1)
POST /currencies (add a new currency) (errors:3.1, 3.2, 4.1, 4.2)
PUT /currencies/:id (modify an existing currency, no upsert) (errors:1, 3.1, 3.2, 4.1, 4.2)
DELETE /currencies/:id (errors:1, 2)

Possible errors:
1.  currency n does not exist
2.  this currency cannot be deleted because some distributions refer to it
3.1 symbol must be truthy
3.2 symbol must be unique
4.1 title must be truthy
4.2 title must be unique


transactions
GET /transactions get all of them
GET /transactions/:id get one of them (errors:1)
POST /transactions (add a new transaction) (errors:3)
PUT /transactions/:id (modify an existing transaction, no upsert) (errors:1, 3)
DELETE /transactions/:id (errors:1, 2)

Possible errors:
1.  transaction n does not exist
2.  this transaction cannot be deleted because some distributions refer to it
3.  datetime must be a parseable datetime


distributions
GET /distributions get all of them
GET /distributions/:id get one of them (errors:1)
POST /distributions (add a new distribution) (errors:3-7)
PUT /distributions/:id (modify an existing distribution, no upsert) (errors:1, 3-7)
DELETE /distributions/:id (errors:1)

Possible errors:
1.  distribution n does not exist
3.  drcr sb "dr" or "cr"
4.  amount sb numeric
5.  transaction_id does not reference an existing transaction
6.  account_id does not reference an existing account
7.  currency_id does not reference an existing currency
