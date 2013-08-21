# Analyse your PostFinance Account

Mostly designed to analyse your own post card usage but also parses all other incomming and outgoing transactions of your account.

*Supports:*

* multiple cards per account
* German PostFinance interface (trivial to add FR, IT & EN - get in touch if you need it)

## How To Get Your Data

### Step 1

Login into your PostFinance and navigate to the transaction viewer:

![PostFinance transaction form](https://raw.github.com/tpreusse/postfinance-analytics/master/guide/transaction-form.png)

### Step 2

You should get a page like this:

![PostFinance transaction page](https://raw.github.com/tpreusse/postfinance-analytics/master/guide/transaction-page.png)

Run following JS be pasting it the url bar:

```javascript
javascript:$('<textarea>').css({position:'fixed',top:0,left:0}).val($('.table-title').html()).appendTo('body');
```

A textbox with html will apprear in the top left cornern, copy/append all html to `data.html` (needs to be created the first time).

If you have more than 100 transaction you will get a next button, click it and repeat step 2 until you cycled through all transactions.

## Process Your Data

1. `cp known_entities.example.yml known_entities.yml`
1. configure `known_entities.yml`
1. `rake pc:read_data`
