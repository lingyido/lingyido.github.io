# todo

* use checkwitness instead of assure user==tx.sender
* because of checkwitness's way, you should check that user can not make your contract address as the user, otherwise it will pass the check
* do not use `verify`
