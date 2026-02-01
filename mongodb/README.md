# MongoDB

```javascript
db.createUser({
  user: "some_user",
  pwd: "some_password",
  roles: [ { role: "readWrite", db: "articles" } ]
})
```
