1. Накатить бэкап базы
```javascript
mongoimport --db usersDB ./users.json
```

2. Найти средний возраст людей в системе
```javascript
// request
let cursor = db.users.aggregate([{
 $group: {
  _id: null,
  totalAvgAge: {
   $avg: "$age"
  }
 }
}]).toArray();
// response
{
 "_id": null,
 "totalAvgAge": 30.38862559241706
}
let avg = cursor[0].totalAvgAge;
```
3. Найти средний возраст в штате Аляска
```javascript
// request
cursor = db.users.aggregate([{
 $match: {
  address: {
   $regex: "Alaska"
  }
 }
}, {
 $group: {
  _id: null,
  avgAgeInAlaska: {
   $avg: "$age"
  }
 }
}]).toArray();
// response
{
 "_id": null,
 "avgAgeInAlaska": 31.5
}
let avg_alaska = cursor[0].avgAgeInAlaska;
```
4. Начиная от Math.ceil(avg + avg_alaska) найти первого человека с другом по имени Деннис
```javascript
// request
let index = Math.ceil(avg + avg_alaska);
db.users.findOne({
 $and: [{
  friends: {
   $elemMatch: {
    name: {
     $regex: "Dennis"
    }
   }
  }
 }, {
  index: {
   $gte: index
  }
 }]
});
// response
{
 "index": 306,
 "name": "Esperanza Blevins",
 "address": "659 Oceanic Avenue, Collins, Utah, 6277"
}
```
5. Найти людей из того же штата, что и предыдущий человек и посмотреть какой фрукт любят больше всего в этом штате (аггрегация)
```javascript
// request
db.users.aggregate(
 [{
  $match: {
   address: {
    $regex: "Utah"
   }
  }
 }, {
  $group: {
   _id: "$favoriteFruit",
   count: {
    $sum: 1
   }
  }
 }]
);
// response
{ "_id" : "strawberry", "count" : 3 }
{ "_id" : "apple", "count" : 6 }
{ "_id" : "banana", "count" : 6 }
```
6. Найти саммого раннего зарегистрировавшегося пользователя с таким любимым фруктом
```javascript
// request
db.users.find({
 favoriteFruit: {
  $regex: "apple"
 }
}).sort({
 registered: 1
}).toArray()[0];
// response
{
 "_id": ObjectId("5adf3c1544abaca147cdd568"),
 "name": "Magdalena Compton",
 "registered": "2014-01-02T10:16:56 -02:00",
 "favoriteFruit": "apple"
}
```
7. Добавить этому пользовелю свойтво: { features: 'first apple eater' }
```javascript
// request
db.users.update({
 "_id": ObjectId("5adf3c1544abaca147cdd568")
}, {
 $set: {
  "features": "first apple eater"
 }
});
// response
WriteResult({
 "nMatched": 1,
 "nUpserted": 0,
 "nModified": 1
})
```
8. Удалить всех любителей клубники (написать количество удаленных пользователей)
```javascript
// request
db.users.remove({
 favoriteFruit: {
  $regex: "strawberry"
 }
});
// response
WriteResult({ "nRemoved" : 253 })
```