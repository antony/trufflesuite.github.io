# Writing Tests in Javascript

Truffle uses the [Mocha](https://mochajs.org/) testing framework and [Chai](http://chaijs.com/) for assertions to provide you with a solid framework from which to write your Javascript tests. Let's dive in and see how Truffle builds on top of Mocha to make testing your contracts a breeze.

Note: If you're unfamiliar with writing unit tests in Mocha, please see [Mocha's documentation](https://mochajs.org/) before continuing.

### Use contract() instead of describe()

Structurally, your tests should remain largely unchanged from that of Mocha: Your tests should exist in the `./test` directory, they should end with a `.js` extension, and they should contain code that Mocha will recognize as an automated test. What makes Truffle tests different from that of Mocha is the `contract()` function: This function works exactly like `describe()` except it enables Truffle's [clean-room features](/docs/getting_started/testing#clean-room-environment). It works like this:

* Before each `contract()` function is run, your contracts are redeployed to the running Ethereum client so the tests within it run with a clean contract state.
* The `contract()` function provides a list of accounts made available by your Ethereum client which you can use to write tests.

Since Truffle uses Mocha under the hood, you can still use `describe()` to run normal Mocha tests whenever Truffle clean-room features are unnecessary.

### Use contract abstractions within your tests

Contract abstractions are the basis for making contract interaction possible from Javascript (they're basically our [flux capacitor](https://www.youtube.com/watch?v=EhU862ONFys)). Because Truffle has no way of detecting which contracts you'll need to interact with within your tests, you'll need to ask for those contracts explicitly. You do this by using the `artifacts.require()` method, a method provided by Truffle that allows you to request a usable contract abstraction for a specific Solidity contract. As you'll see in the example below, you can then use this abstraction to make sure your contracts are working properly.

For more information on using contract abstractions, see the [Interacting With Your Contracts](/docs/getting_started/contracts) section.

### Using artifacts.require()

Using `artifacts.require()` within your tests is the exact same as using it within your migrations. See the [artifacts.require() documentation](./migrations#artifacts-require-) within the Migrations section for detailed usage.

# Example

Here's an example test provided to you by `truffle init`. Note the use of the `contract()` function, the `accounts` array for specifying available Ethereum accounts, and our use of `artifacts.require()` for interacting directly with our contracts.

File: `./test/metacoin.js`

```javascript
// Specifically request an abstraction for MetaCoin
var MetaCoin = artifacts.require("MetaCoin");

contract('MetaCoin', function(accounts) {
  
  it("should put 10000 MetaCoin in the first account", function () {
    return MetaCoin.deployed().then(function(instance) {
      return instance.getBalance.call(accounts[0]);
    }).then(function(balance) {
      assert.equal(balance.valueOf(), 10000, "10000 wasn't in the first account");
    });
  });

  context('Balance is transferrable', function () {
    const account_one = accounts[0];
    const account_two = accounts[1];
    const amount = 10;

    let account_one_starting_balance;
    let account_two_starting_balance;
    let account_one_ending_balance;
    let account_two_ending_balance;

    before(function () {
      return MetaCoin
        .deployed()
        .then(function (meta) {
          return Promise.all([
            meta,
            meta.getBalance.call(account_one),
            meta.getBalance.call(account_two),
            meta.sendCoin(account_two, amount, {from: account_one})
          ])
        }).then(function (results) {
          const meta = results[0];
          account_one_starting_balance = results[1].toNumber();
          account_two_starting_balance = results[2].toNumber();
          return Promise.all([
            meta.getBalance.call(account_one),
            meta.getBalance.call(account_two)
          ])
        })
        .then(function (balances) {
          account_one_ending_balance = balances[0].toNumber();
          account_two_ending_balance = balances[1].toNumber();
        });
    });

    it('should have taken amount from sender', function () {
      const expected_balance = account_one_starting_balance - amount;
      assert.equal(account_one_ending_balance, expected_balance);
    });

    it('reciever should have gained amount', function () {
      const expected_balance = account_two_starting_balance + amount
      assert.equal(account_two_ending_balance, expected_balance);
    });
  });
});
```

This test will produce the following output:

```
Using network 'development'.

  Contract: MetaCoin
    ✓ should put 10000 MetaCoin in the first account
    Balance is transferrable
      ✓ should have taken amount from sender
      ✓ reciever should have gained amount

  3 passing (113ms)
```

# Advanced

Truffle gives you access to Mocha's configuration so you can change how Mocha behaves. See the [project configuration](/docs/advanced/configuration#mocha) section for more details.
