# Salt

password => hasing => hash

password + salt => hasing => hash


salt도 매번랜덤


hash(password + passsalt) => passhash

``` txt
UserAccount
| useruid | username | passhash | passsalt | register_dt | lastlogin_dt |

UserToken
| useruid | refresh_token |

```