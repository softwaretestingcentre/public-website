---
title: "Lesson 10 - Basket Cases"
date: 2025-05-30
---
For this lesson we are going to try to add a Feature where a customer adds an item to their basket:
```gherkin
Feature: Customers can buy artwork

  Scenario: Betty chooses "Unusable Security"
    Given Betty adds "Unusable Security" to her basket
    When she checks her basket
    Then she can see her basket contains only 1 item of "Unusable Security"
```
This involves clicking the Add button on an item, then going to Checkout:

![image](https://github.com/user-attachments/assets/abd69574-a687-4fc2-a5ec-f8099ca10b73)

and confirming that the number of items and subtotal are correct:

![image](https://github.com/user-attachments/assets/caa8fa0d-ec67-4b60-8cf2-b96c51ff3e23)

Our Step Definitions will mostly rely on a new `Shop` helper class:
```java
public class ShopStepDefinitions {
    @Given("{actor} adds {string} to her basket")
    public void bettyAddsToHerBasket(Actor actor, String itemName) {
        actor.wasAbleTo(Open.browserOn().the(LandingPage.class));
        actor.attemptsTo(Shop.addItemToBasket(itemName));
    }

    @When("{actor} checks her basket")
    public void sheChecksHerBasket(Actor actor) {
        actor.attemptsTo(Shop.openBasket());
    }

    @Then("{actor} can see her basket contains only {int} item of {string}")
    public void sheCanSeeHerBasketContainsOnly(Actor actor, int itemCount, String itemName) {
        actor.attemptsTo(Shop.checkBasketContainsOnly(itemCount, itemName));
    }
}
```

```java
public class Shop {

    public static Performable addItemToBasket(String itemName) {
        return Task.where("adds " + itemName + " to basket",
                actor -> {
                    actor.remember("item cost", Text.of(ShopPage.ITEM_COST.of(itemName)));
                    actor.attemptsTo(Click.on(ShopPage.ADD_ITEM.of(itemName)));
                }
        );
    }

    public static Performable openBasket() {
        return Task.where("opens basket",
                actor -> {
                    actor.attemptsTo(Click.on(ShopPage.CHECKOUT));
                    actor.remember("basket contents", Text.of(CheckoutPage.BASKET_ITEMS));
                    actor.remember("basket subtotal", Text.of(CheckoutPage.SUB_TOTAL));
                }
        );
    }

    public static Question<Boolean> basketIsCorrect(int itemCount, String itemName) {
        return Question.about("basket contents").answeredBy(
                actor -> {
                    String itemPrice = actor.recall("item cost").toString().replaceAll("^.", "");
                    String basketContent = actor.recall("basket contents");
                    String subTotal = actor.recall("basket subtotal");
                    return basketContent.equals(itemName + "\nQuantity " + itemCount + "remove\n" + itemPrice)
                            && subTotal.equals("$" + itemPrice + ".00");
                }
        );
    }

    public static Performable checkBasketContainsOnly(int itemCount, String itemName) {
        return Ensure.that("basket has " + itemCount + " of " + itemName,
                basketIsCorrect(itemCount, itemName))
                .isTrue();
    }

}
```
The first Task definition is just a simple click on the `ADD_ITEM` button but it has also stored the item price.

The `openBasket()` method also stores the text content of the basket and the subtotal.

Finally the method to check the basket uses a compound `Question` object to check both the basket contents and subtotal.

We had to implement 2 PageObjects for the Shop and Checkout components:
```java
public class ShopPage extends PageObject {

    public static Target ADD_ITEM = Target.the("Add Item to Basket")
            .locatedBy("//*[.='{0}']//..//button");

    public static Target ITEM_COST = Target.the("item cost")
            .locatedBy("//*[.='{0}']//..//*[@class='tilePrice']");

    public static Target CHECKOUT = Target.the("Checkout button")
            .locatedBy("//*[text()='Checkout']");

}

public class CheckoutPage extends PageObject {

    public static Target BASKET_ITEMS = Target.the("Item in basket")
            .locatedBy("//*[@class='productItem']");

    public static Target SUB_TOTAL = Target.the("basket subtotal")
            .locatedBy("//*[.='subtotal']//following-sibling::*");

}
```
