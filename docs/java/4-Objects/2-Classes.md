# Classes

Previously, we discussed how objects can be made to group together properties and actions into a single entity. Here, we will discuss how we implement objects and these custom datatypes in our programs.

## Classes Analogy

Think of how objects are made in today's world. We have countless objects, such as cars, computers, books, etc. These are all made by some factory that follows some set of instructions. Each specific car/computer/book is different from each other in that the exact materials might be different, the age might be different, but each and every single one is made from a specific set of instructions that define how it should be manufactured and how it should act. In programming, this set of instructions, or the blueprint, is called a **`class`**.

## Java Classes

In Java, each class, or each blueprint, is in its own file. The class name should match up with the file name. For instance, a class called `Car` should be in a file named `Car.java`. All classes should be in UpperCase, with no spaces and the first letter of each word capitalized.

Once we have this new file called `Car.java`, here is how we make a new class in that file:

```java
public class Car {

}
```

That's all we need to make the most basic class, or blueprint. Now let's see how we can use that class to construct an object. Notice that in this new file, we don't put `public static void main(String[] args)`. This is because we can only have one of those in the program.

Back in our `public static void main(String[] args)` function, we can use this new class/blueprint to make an object.

```java
public static void main(String[] args) {
    Car carObject = new Car();
}
```

That's it, we made a simple, barebones object that has 0 properties or actions. Now let's see how we can extend this so that we can make more advanced objects that actually have a purpose.

## Properties

One of the key things that is a part of objects, as we said before, is that they can have properties. For instance, each book can have a different name or a different number of pages. Therefore, it is essential for us to be able to introduce these properties into the class we have made so far.

So, in our `Car.java` class, we want to introduce the `model`, `age`, `color`, and `speed` property.

```java
public class Car {

    String model;
    int age;
    String color;
    double speed;
}
```

Then, in our `main` function, we can set these properties in our `Car` object.

```java
public static void main(String[] args) {
    Car carObject = new Car();
    carObject.model = "Magic School Bus";
    carObject.age = 100;
    carObject.color = "Yellow";
    carObject.speed = 31415.9265;

    System.out.println(carObject.model);
    System.out.println(carObject.age);
    System.out.println(carObject.color);
    System.out.println(carObject.speed);
}
```

## Actions

The other key components of objects is that they can have actions. For instance, we want to be able to drive our car. We can do this by adding functions to our class.

In our `Car` class, we want to be able to drive our car at the specified speed for a certain amount of time. We do this by making a function.

```java
public class Car {
    String model;
    int age;
    String color;
    double speed;

    void driveFor(double time) {
        System.out.println("Drove " + (speed * time));
    }

    int distance(double time) {
        return speed * time;
    }
}
```

```java
public static void main(String[] args) {
    Car carObject = new Car();
    carObject.model = "Magic School Bus";
    carObject.age = 100;
    carObject.color = "Yellow";
    carObject.speed = 31415.9265;

    System.out.println(carObject.model);
    System.out.println(carObject.age);
    System.out.println(carObject.color);
    System.out.println(carObject.speed);

    car.driveFor(12.0);
    System.out.println("Distance: " + car.driveFor(24.0));
}
```

## Static vs Nonstatic

You may have noticed that before, when we made functions, we always made them `static`. For instance, `public static void print()`. What `static` means is that in order to call this function, we do not need an object. By default, all functions that you create have to be called from an object. For instance, before, when we made functions, we did not need an object for that function. Instead, we just called the function `print()`. However, if we want that function to be part of an object as an action, we do not want to put `static`.

This concept of `static` also extends to variables, or properties, as well. Just like with functions, static variables do not need objects. Nonstatic variables do. To see this in action, let's see an example.

In `Car.java`

```java
public class Car {
    String manufacturer;

    static String type = "Car";
}
```

In `main()`

```java
public static void main(String[] args) {
    Car car1 = new Car();
    car1.manufacturer = "BMW";


    Car car2 = new Car();
    car2.manufacturer = "Toyota";

    System.out.println(car1.manufacturer); // Needs an object

    // Below does not work because `manufacturer` requires a object to be used (manufacturer is different for each object)
    // System.out.println(Car.manufacturer);


    System.out.println(Car.type); // Does not need an object
    // Each car has the same type, so you can make that static. This will be the same for all of the `Car` variables and is shared
}
```
