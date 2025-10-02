MongoDB Virtuals **full proper `README.md` file** with complete structure, Markdown formatting, and fully working code snippets.

````markdown
# MongoDB Virtuals (Mongoose)

Mongoose **virtuals** are document properties that you can get and set but **don’t persist to MongoDB**.  
They are typically used for computed properties or reverse population of relationships.

---

## ✨ Why Use Virtuals?
- Add **computed properties** without storing them in the database.
- Format or transform existing fields.
- Define **getters** and **setters** for custom behavior.
- Create **virtual relationships** between collections (for `populate`).

---

## 🛠️ Basic Syntax

```js
schema.virtual("fieldName")
  .get(function () {
    // return something
  })
  .set(function (value) {
    // modify document property
  });
````

---

## 📌 Example 1: Getter Virtual

```js
const userSchema = new mongoose.Schema({
  firstName: String,
  lastName: String
});

// Virtual property: fullName
userSchema.virtual("fullName").get(function () {
  return `${this.firstName} ${this.lastName}`;
});

const User = mongoose.model("User", userSchema);

(async () => {
  const user = new User({ firstName: "John", lastName: "Doe" });
  console.log(user.fullName); // "John Doe"
})();
```

🔹 Here, `fullName` is **computed on the fly**, not stored in MongoDB.

---

## 📌 Example 2: Setter Virtual

```js
const userSchema = new mongoose.Schema({
  firstName: String,
  lastName: String
});

// Virtual setter: split full name
userSchema.virtual("fullName").set(function (name) {
  const parts = name.split(" ");
  this.firstName = parts[0];
  this.lastName = parts[1];
});

const User = mongoose.model("User", userSchema);

(async () => {
  const user = new User();
  user.fullName = "Jane Smith"; // sets firstName and lastName
  console.log(user.firstName);  // "Jane"
  console.log(user.lastName);   // "Smith"
})();
```

---

## 📌 Example 3: Formatting Data (Custom Logic)

```js
const productSchema = new mongoose.Schema({
  name: String,
  price: Number
});

// Format price into currency
productSchema.virtual("priceWithCurrency").get(function () {
  return `$${this.price.toFixed(2)}`;
});

const Product = mongoose.model("Product", productSchema);

(async () => {
  const item = new Product({ name: "Shoes", price: 49.99 });
  console.log(item.priceWithCurrency); // "$49.99"
})();
```

---

## 📌 Example 4: Relationship Virtuals (`ref`, `populate`)

Use virtuals to define **reverse population** (e.g., posts belonging to a user).

```js
const userSchema = new mongoose.Schema({
  name: String,
});

const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: { type: mongoose.Schema.Types.ObjectId, ref: "User" }
});

// Virtual: connect User → Posts
userSchema.virtual("posts", {
  ref: "Post",            // The model to use
  localField: "_id",      // Find posts where `author` = `_id`
  foreignField: "author"  // field in Post
});

const User = mongoose.model("User", userSchema);
const Post = mongoose.model("Post", postSchema);

(async () => {
  const user = await User.create({ name: "Alice" });
  await Post.create({ title: "Hello", content: "World", author: user._id });

  const populatedUser = await User.findOne({ name: "Alice" }).populate("posts");
  console.log(populatedUser.posts); // Array of posts authored by Alice
})();
```

---

## 📌 Example 5: Enabling Virtuals in JSON Output

By default, virtuals **don’t show up** when you convert documents to JSON.
Enable them explicitly:

```js
const schema = new mongoose.Schema(
  { name: String },
  { toJSON: { virtuals: true }, toObject: { virtuals: true } }
);

schema.virtual("greeting").get(function () {
  return `Hello, ${this.name}`;
});

const Person = mongoose.model("Person", schema);

(async () => {
  const person = new Person({ name: "Sam" });
  console.log(person.toJSON()); // { _id: ..., name: "Sam", greeting: "Hello, Sam" }
})();
```

---

## ✅ Key Takeaways

* Virtuals = **computed fields** that don’t persist.
* Useful for **formatting**, **computed properties**, and **relationships**.
* Need `{ virtuals: true }` in `toJSON` or `toObject` to include in outputs.
* Great for **reverse population** (`populate`) of related models.

---

## 📚 References

* [Mongoose Docs: Virtuals](https://mongoosejs.com/docs/guide.html#virtuals)
* [Mongoose Populate](https://mongoosejs.com/docs/populate.html)

```

This is a **ready-to-use learning notes `README.md`** with code examples, explanations, and references.  

```
