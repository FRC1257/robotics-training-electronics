# More About Classes

This is a lot of extra information on classes.

## Constructors

Let's look at our class from last time:

```java
public class Car {

    String model;
    int age;
    String color;
    double speed;
}
```

```java
public static void main(String[] args) {
    Car carObject = new Car();
    carObject.model = "Magic School Bus";
    carObject.age = 100;
    carObject.color = "Yellow";
    carObject.speed = 31415.9265;
}
```

You may notice that it's extremely tedious to set all of the variables. We would like to make it so that while we're creating the object with `new Car()`, we can just input our values there. We can do this with something known as constructors.

In our `Car.java`, let's define a constructor:

```java
public class Car {

    String model;
    int age;
    String color;
    double speed;

    Car(String model, int age, String color, double speed) {
        this.model = model;
        this.age = age;
        this.color = color;
        this.speed = speed;
    }
}
```

```java
public static void main(String[] args) {
    Car carObject = new Car("Magic School Bus", 100, "Yellow", 31415.9265);
}
```

In our constructor, we first define a set of input parameters. These are passed in when we create the object using `new Car(`. Then, back to our `Car` class, we have to define what we do with those. In this case, we assign our property, `this.model` to `model`. The reason why we have `this.model` and `model` is because there is a naming conflict between the constructor arguments and the properties.

In this case, to differentiate the two, `this.model` refers to the object's property and `model` refers to the parameter passed into the constructor.

Whenever we create a class and we don't explicitly define a constructor, Java automatically creates a blank constructor that does nothing. You can think of it as doing the following:

```java
public class Car {

    String model;
    int age;
    String color;
    double speed;
    
    Car() {
        // automatically creates this constructor as long as there are NO other constructors defined
    }
}
```

## Package

When we're making a project with 100s of classes, we're going to have 100s of files. As a result, we always want to be able to organize these files within folders. Within Java, these folders are called `packages`. Whenever we put a file in a folder that is not the root folder, we have to put at the top `package foldername`. For instance, if `Car.java` were in the folder `automobiles`, we would put:

```java
package automobiles;

public class Car {

}
```

If it were in the folder `automobiles/cars`, we would put:

```java
package automobiles.cars;

public class Car {

}
```

## Import

If we have two classes, and they aren't in the same folder, then we have to do something called an `import` statement. For instance, say we have `automobiles/Cars.java` and `Main.java`. We would have to `import` the `Cars.java` file.

```java
package automobiles;

public class Car {

}
```

```java
import automobiles.Car;

public class Main {

    public static void main(String[] args) {
        System.out.println("Imported Cars");
    }
}
```

## Access Modifiers

When we have a project with multiple files, there are times in which we want to limit the amount of access classes have to each other. For instance, if we had a security program, we wouldn't want just any file to access the variables and methods of another file. Additionally, to make code cleaner and more maintainable, there are various reasons why we would want to limit access to only a few files.

Whenever we have a variable or a function, we can change its access modifier. For instance, if we change the access modifier of a variable, then we can change whether it is available to other files.

Here are the access modifiers:

- `private`: Only code in the same file can access this 
- `(default)`: Only code in the same package/folder can access this (this is when you don't explicitly put an access modifier)
- `public`: Any code can access this

There is another one called `protected`, but we won't cover it. We will not need to use it, but you can research it yourself if you want.

Let's say we have a file named `Other.java`, `MainSame.java`, and `folder/MainOut`. `MainOut.java` will be inside of a separate folder than Other.java, while `MainSame.java` will be in the same folder as `Other.java`. This example is included in the `/programs` folder for `5. Objects`.

```java
public class Other {

    private int privateNumber;
    int defaultNumber;
    public int publicNumber;

    private void privatePrint() {
        System.out.println("Print 1");
    }

    void defaultPrint() {
        System.out.println("Print 2");
    }

    public void publicPrint() {
        System.out.println("publicPrint ");
    }
}
```

```java
public class MainSame {

    public static void main(String[] args) {
        Other o = new Other();
        System.out.println(o.privateNumber); // This will not work, nothing outside of the class Other can access private
        System.out.println(o.defaultNumber); // This will work, since MainSame is in the same folder
        System.out.println(o.publicNumber); // This will work, since anything can access public

        o.privatePrint(); // This will not work, nothing outside of the class Other can access private
        o.defaultPrint(); // This will work, since MainSame is in the same folder
        o.publicPrint(); // This will work, since anything can access public
    }
}
```

```java
package folder;

public class MainOut {

    public static void main(String[] args) {
        Other o = new Other();
        System.out.println(o.privateNumber); // This will not work, nothing outside of the class Other can access private
        System.out.println(o.defaultNumber); // This will not work, since MainSame is not in the same folder
        System.out.println(o.publicNumber); // This will work, since anything can access public

        o.privatePrint(); // This will not work, nothing outside of the class Other can access private
        o.defaultPrint(); // This will not work, since MainSame is not in the same folder
        o.publicPrint(); // This will work, since anything can access public
    }
}
```

Generally, it is the standard to have all variables in a class be private, and to use functions to modify/access these variables.

## Getters and Setters

When we have private variables, we use `getters` and `setters` to access these private variables from outside of the class.

```java
public class Private {

    private int number;
    private String string;

    public int getNumber() {
        return number;
    }

    public String getString() {
        return string;
    }

    public void setNumber(int number) {
        this.number = number;
    }

    public void setString() {
        this.string = string;
    }
}
```

One reason why we might prefer this over just making the variable public is that we can do execute some other code within the getters or setters to do things such as input verification.

```java
public class Private {

    private int number;
    private String string;

    private int accessCounter = 0;

    public int getNumber() {
        accessCounter++;
        return number;
    }

    public String getString() {
        accessCounter++;
        return string;
    }

    public int getAccessCounter() {
        return accessCounter;
    }

    public void setNumber(int number) {
        if(number > 0) {
            this.number = number;
        }
    }

    public void setString() {
        if(string != "") {
            this.string = string;
        }
    }
}
```

We could use the `accessCounter` variable to store the number of times the variables were accessed. We could also impose restrictions on the setters to ensure the variables are not set to bad values.
