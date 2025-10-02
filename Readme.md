MONGODB VIRTUALS NOTES 💻 **LEARNING NOTES**
---

````markdown
# 📘 Mongoose Virtuals Cheat Sheet

Virtuals in **Mongoose** are document properties that **don’t persist in MongoDB**.  
They are calculated values or relationships derived from existing schema fields.

---

## 🚀 Why Use Virtuals?
- 🧮 **Computed values**: Derive values without storing duplicate data.  
- 🎨 **Formatting**: Return human-readable versions of stored values.  
- 🔗 **Relationships**: Simulate joins (populate virtuals).  
- 📦 **Cleaner data**: Keep DB normalized but still provide useful extra fields.  

---

## 🛠️ 1. Basic Virtual Getter

```js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  firstName: String,
  lastName: String,
});

// Virtual Getter → fullName
userSchema.virtual("fullName").get(function () {
  return `${this.firstName} ${this.lastName}`;
});

userSchema.set("toJSON", { virtuals: true });
userSchema.set("toObject", { virtuals: true });

const User = mongoose.model("User", userSchema);

(async () => {
  const u = new User({ firstName: "Ahmed", lastName: "Ali" });
  console.log(u.fullName); // "Ahmed Ali"
})();

✅ **When to use**: Need a calculated field like `fullName`.

---

## 🛠️ 2. Virtual Setter

```js
userSchema.virtual("name").set(function (v) {
  const [first, last] = v.split(" ");
  this.firstName = first;
  this.lastName = last;
});

const u = new User();
u.name = "Ali Khan"; 

console.log(u.firstName); // "Ali"
console.log(u.lastName);  // "Khan"
```

✅ **When to use**: Want to assign multiple schema fields in one go.

---

## 🛠️ 3. Formatting / Derived Values

Example: Store `minutes` but expose `HH:MM`.

```js
const routeSchema = new mongoose.Schema({
  arrivalMinutes: Number,
});

routeSchema.virtual("arrivalTime").get(function () {
  const h = String(Math.floor(this.arrivalMinutes / 60)).padStart(2, "0");
  const m = String(this.arrivalMinutes % 60).padStart(2, "0");
  return `${h}:${m}`;
});

const Route = mongoose.model("Route", routeSchema);

(async () => {
  const r = new Route({ arrivalMinutes: 125 });
  console.log(r.arrivalTime); // "02:05"
})();
```

✅ **When to use**: Need to convert stored raw values into human-friendly formats.

---

## 🛠️ 4. Relationship Virtuals (Populate)

Virtuals can define **reverse references** (not stored in DB).

```js
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
});

const commentSchema = new mongoose.Schema({
  postId: { type: mongoose.Schema.Types.ObjectId, ref: "Post" },
  text: String,
});

// Virtual relationship: all comments for a post
postSchema.virtual("comments", {
  ref: "Comment",
  localField: "_id",
  foreignField: "postId",
});

const Post = mongoose.model("Post", postSchema);
const Comment = mongoose.model("Comment", commentSchema);

(async () => {
  const post = await Post.create({ title: "My Post", content: "Hello" });
  await Comment.create({ postId: post._id, text: "Nice post!" });

  const populatedPost = await Post.findById(post._id).populate("comments");
  console.log(populatedPost.comments); // Array of comments
})();
```

✅ **When to use**: To connect collections without embedding documents.

---

## 🛠️ 5. Combine Getter + Setter

```js
const productSchema = new mongoose.Schema({
  priceCents: Number,
});

// Getter: return price in dollars
productSchema.virtual("price").get(function () {
  return (this.priceCents / 100).toFixed(2);
});

// Setter: allow setting price in dollars
productSchema.virtual("price").set(function (v) {
  this.priceCents = v * 100;
});

const Product = mongoose.model("Product", productSchema);

(async () => {
  const p = new Product();
  p.price = 19.99; 
  console.log(p.priceCents); // 1999
  console.log(p.price);      // "19.99"
})();
```

✅ **When to use**: Useful for unit conversion (cents ↔ dollars, seconds ↔ time).

---

## ⚡ Key Notes

* Virtuals are **not stored** in MongoDB.
* They are included in `toObject()` / `toJSON()` **only if enabled**:

  ```js
  schema.set("toJSON", { virtuals: true });
  schema.set("toObject", { virtuals: true });
  ```
* Great for APIs: send clients **derived or related values** without extra logic in controllers.

---

## 📌 When to Use Virtuals

✔ Derived values (e.g., fullName, formatted date)
✔ Unit conversions (cents ↔ dollars, minutes ↔ time)
✔ Reverse relationships (`Post` → `Comments`)
✔ Cleaner code (business logic stays in schema)

❌ Don’t use them if you actually need to **query/filter** by the value → store in DB instead.

---

## ⚠️ Limitations & Best Practices

* Virtuals **cannot be used in queries** (`.find({ virtualField: ... })` won’t work).
* They add **no DB overhead** since they aren’t stored.
* Use **indexes** on real fields if you need query speed.
* Keep formatting logic in virtuals, not controllers → **separation of concerns**.

---

📚 **Recommended**: Use virtuals for **presentation and relationships**, not for essential persisted values.

```

---

```
