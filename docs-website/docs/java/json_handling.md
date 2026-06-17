# 🚀 JSON Handling in Java

JSON = JavaScript Object Notation

### 🚀 JSON Types
|Type |	Example|
|-----|--------|
|object	|{}|
|array |	[]|
|string	|"hello"|
|number|	100|
|boolean	|true|

### 🚀 2. Java JSON Libraries

Most common libraries:
```
Jackson ✅ (MOST IMPORTANT)
Gson
org.json
```

## 🚀 4. Convert Object → JSON

THIS is called:

#### 👉 Serialization

💥 Example Class

```ruby
class BankAccount {

    public String name;
    public double balance;

    BankAccount(String name, double balance) {
        this.name = name;
        this.balance = balance;
    }
}
```

#### 💥 Convert to JSON

```ruby
ObjectMapper mapper =
    new ObjectMapper();

BankAccount acc =
    new BankAccount("Supriyo",5000);

String json =
    mapper.writeValueAsString(acc);

System.out.println(json);
```

#### 💥 Output
```
{"name":"Supriyo","balance":5000.0}
```

🧠 Meaning
```
Java object → JSON text
```

## 🚀 5. Convert JSON → Object

THIS is called:

#### 👉 Deserialization
**cant understand**

