---
title: Java 8 Features
tags:
  - java
  - github
date: 2018-02-04
---

I would like to introduce the Java 8 features in a systematic way. Basically, Java 8 introduces the following new features. 1. functional interface, 2. lambda expression, 3. 6 types of pre-defined functional interface in Java 8, 4. stream API, 5. default method, 6. effective final, 7. method reference.

a. `Functional Interface`
Functional Interface is a single abstract method interface.
``` 
@FunctionalInterface
public interface Payment {
    public void pay(ShoppingCart shoppingCart, 
    		Predicate<Integer> isReceiptRequired);
}
```
The annotation _FunctionalInterface_ checks against the class to comply with the rule of functional interface that it is a single abstract method interface. If we introduce another abstract method in the class, there will be compile error.

b. `Lambda Expression`
```
public void checkout() {
    Payment cashPay = (ShoppingCart shoppingCart, 
			Predicate<Integer> isReceiptRequired) -> {
        System.out.println("Cash is NOT support");
    };
    cashPay.pay(getShoppingCart(), (Integer amount) -> false);
}

```
Lambda Expression can be only used for Functional Interface. It uses the syntax _->_ to indicate the method body of the single abstract method interface. The left side of _->_ is the method parameters, and the right side of _->_ is the method implementation.


c. `6 Types of Functional Interface in Java 8`
They are all under the package _java.util.function_.
A. _Predicate, (I) -> boolean, replace if else_
B. _Function, (I) -> R, transformer, one type to another type_
C. _Consumer, (I) -> void, consumer the input data_
D. _Supplier, () -> new (), factory pattern_
E. _BinaryOperator, (I, I) -> I, reducers, accumulation_
F. _UnaryOperator, (I) -> I_
```
public default Optional<Integer> calculateTotalAmount(ShoppingCart shoppingCart) {
    Predicate<Product> filterCondition = (Product p) -> 
            StringUtils.isNotBlank(p.getId()) && p.getPrice() > 0;
    Function<Product, Integer> mapToPrice = (Product p) -> p.getPrice() * p.getQuantity();
    BinaryOperator<Integer> reduceCheckoutAmount = 
            (Integer price0, Integer price1) -> price0 + price1;
    /**
     * stream (for Collections) 1. filter, 2. map, 3. reduce.
     * fork join pool with concurrent processing
     */
    return shoppingCart.getProducts().stream().parallel()
            .filter(filterCondition).map(mapToPrice).reduce(reduceCheckoutAmount);
}
```
Here we have 3 examples for Predicate, Function, BinaryOperator functional interfaces respectively. They are all functional interfaces, so we can represent the method body as the lambda expression.
_Predicate_ is a functional interface which has a single method of return type boolean. The method implementation _StringUtils.isNotBlank(p.getId()) && p.getPrice() > 0_ will be either true or false. By the way, the _return_ key word can be omitted for the lambda expression approach.
_Function_ is a functional interface which has a single method of one type input, the different type of output. _(Product p) -> p.getPrice() * p.getQuantity();_, input is Product, but output is String which is different from the input type.
_BinaryOperator_ is a functional interface which has a single method of two inputs and one output with the same type. It is similar to the _reduce_ of the map-reduce concept, generally for accumulation. _(Integer price0, Integer price1) -> price0 + price1;_, two integers as the input, the output is an integer of the sum of the two input integers.

d. `Stream API`
```
public default String printReceipt(ShoppingCart shoppingCart) {
    StringBuilder receipt = new StringBuilder();
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    for(Category category : Category.values()) {
        receipt.append("$$$$$$$$$").append(category.name()).append("$$$$$$$$$").append("\n");
        shoppingCart.getProducts().stream()
                .filter((Product p) -> p.getCategory() == category).sorted((Product o1, Product o2)  
                    -> o1.getId().equals(o2.getId()) ? 0 : (o1.getPrice() > o2.getPrice() ? 1 : -1))
                .forEach((Product p) -> receipt.append(p.getId()).append("  ").append(p.getName())
                    .append("  ").append("X").append(p.getQuantity()).append("  ").append("SGD")
                    .append(p.getPrice()).append("\n"));
    }
    receipt.append("^^^^^^^^^^^^^^^^^^^^").append("\n");
    receipt.append("Checkout Total Amount: ").append(calculateTotalAmount(shoppingCart).get())
            .append(" SGD").append("\n");
    receipt.append("Transaction Time: ").append(sdf.format(new Date())).append("\n");
    return receipt.toString();
}
```
For collections, _Stream API_ allows to iterate each item of the collection, and enqueue each item within the pipeline to process, such as filter, for-each, map-reduce.
_shoppingCart.getProducts().stream()_, it iterates the list of Product as the stream pipeline, moreover, _parallel()_ will take use of the fork join pool for concurrent processing, then calls the corresponding stream API (the functional interfaces play the role like the anonymous inner class, but they are different in their underline behaviours), filter which input is a Predicate, (see the code in section c) map which input is a Function, reduce which input is a BinaryOperator, and so forth.

e. `Default Method`
The code snippet in section c and section d has the method with the _default_ key word. Before Java 8 release, in the interface, we can just define the abstract methods as the contract, and the class which implements the interface has the concrete implementation of the interface method.
Java 8 introduces the default method in the interface class for the default implementation.

f. `Effective Final`
```
public void checkout() {
    System.out.println("---------Checkout with Cash---------");
    int cash = 10;
    Payment cashPay = (ShoppingCart shoppingCart, Predicate<Integer> isReceiptRequired) -> {
        /**
         * cannot change, effective final;
         * local variable for lambda expression, auto change to final variable.
         * 
         * cash = 2000;
         */
        System.out.println("Cash is NOT support");
    };
    cashPay.pay(getShoppingCart(), (Integer amount) -> false);
    System.out.println();
    System.out.println("---------Checkout with Cashlessa 8---------");
    payment.pay(getShoppingCart(), this::isAskingReceipt);
}
```
Local variable for lambda expression, it will be automatically changed to final variable.
In this example, the local variable int cash cannot be modified within the lambda expression (cash = 2000) as it is changed to the final variable automatically. Otherwise, there will be compile error in line 9 (code snippet in section f).

g. `Method Reference`
```
public void checkout() {
    ...
    /**
     * method reference
     */
    payment.pay(getShoppingCart(), this::isAskingReceipt);
}
private boolean isAskingReceipt(Integer amount) {
    return amount > 10;
}
```
In java 8, we can use the syntax _::_ to refer a method body.
In this example, the second input of the pay method is a Predicate. Here we use the method reference to indicate the method body which is the implemetation of the Predicate.

In summary, Java 8 provides us some flexible ways to return the complex logic within one line, and more significant, it introduces the concept of Functional Interface for the functional programming pattern.

The code snippets above are under the package _com.snippet.java8.feature_ of the Snippets repository in [GitHub](https://github.com/xljiadahao/Snippets.git). Feel free to create any GitHub issues against the repository if you have any questions or you want to correct the code snippet errors based on your understanding. Welcome.
