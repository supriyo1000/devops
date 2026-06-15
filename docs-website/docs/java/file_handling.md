# Java File Handling with File IO

### 🟢 JAVA IO

Classic approach.

Examples:

```
File
FileReader
BufferedReader
FileWriter
```


### 🔵 MODERN JAVA NIO

Examples:

```
Path
Files
```

### What is File?

File represents :
```
path/file/folder information
```

#### IMPORTANT :

```
File class itself does NOT read file content
```
It mainly :

+ checks existence
+ creates files
+ deletes files
+ gets metadata

```
new File("test.txt")
```

#### creates :

```
file reference object.

NOT actual file content.
```

```ruby
import java.io.File;
import java.io.IOException;

public class main {
    public static void main(String[] args) {
        File file = new File("test.txt");
        
        try {
            if(!file.exists()) {
            file.createNewFile();
            String name = file.getName();
            System.out.println("File " + name + " has length of " + file.length());
        }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## FileWriter

```ruby
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;

public class main {
    public static void main(String[] args) {
        File file = new File("test.txt");
        
        try {
            FileWriter writer = new FileWriter(file);
            writer.write("Helllo java");
            writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

Without :
```
writer.close()

data may not save properly.
```
#### ⚠️ Important

FileWriter writes :
```
character/text data
```

## BufferedWriter

Professional code usually uses :

👉 BufferedWriter
🧠 Why?

Instead of writing :
```
character-by-character
```

it uses :
```
memory buffer
```

FASTER.

```ruby
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;

public class main {
    public static void main(String[] args) {
        // File file = new File("test.txt");
        try {
            BufferedWriter bw = new BufferedWriter(new FileWriter("test.txt"));

            bw.write("Hello");
            bw.newLine();
            bw.write("BufferedWriter");
            bw.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## try-with-resources (VERY IMPORTANT)

Professional modern style.

```ruby
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;

public class main {
    public static void main(String[] args) {
        try(BufferedWriter bw = new BufferedWriter(new FileWriter("test.txt"))){
            bw.write("Hello");
            bw.newLine();
            bw.write("try-with-resources");
            bw.newLine();
            bw.write("VERY IMPORTANT");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### 🧠 Why Better?

Java automatically closes :
```
resource
```
No need:
```
close()
```

## FileReader

```ruby
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class main {
    public static void main(String[] args) {
        try(FileReader fr = new FileReader("test.txt")){
            int ch;
            while((ch = fr.read()) != -1) {
                System.out.println((char) ch);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
```
H
e
l
l
o



B
u
f
f
e
r
e
d
W
r
i
t
e
r
```
#### 🧠 VERY IMPORTANT
```
fr.read()
```

returns :
```
ASCII/int value
```
So :

#### (char) ch

conversion needed.

#### ⚠️ Problem with FileReader

Reads:
```
character-by-character
```
slow for large files.

## BufferedReader (VERY IMPORTANT 🔥)

Reads line by line.

```ruby
package org.example;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class App {
    public static void main(String[] args) throws IOException, InterruptedException {
        try(BufferedReader br = new BufferedReader(new FileReader("test.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}
```

## FileInputStream

Used for :
```
👉 binary data
```

Examples:
```
image
pdf
zip
```

NOT text.

#### 🧠 Difference
|Class|Purpose|
|-----|-------|
|FileReader |text|
|FileInputStream|	binary|



