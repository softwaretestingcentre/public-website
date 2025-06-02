---
title: "Lesson 11 - Multiple Basket Cases"
date: 2025-06-03
---
In the [previous lesson](/public-website/2025/05/30/Lesson-10-Basket-Cases.html) we saw how to add single items to the basket.

Since most of the methods we created to test the result need to know how many items there are in the basket, it should be easy to create another test for bulk buying a single item:

```gherkin
  Scenario: Betty chooses multiple copies of "Experimental"
    Given Betty adds 3 items of "Experimental" at $128 each to her basket
    When she checks her basket
    Then she can see her basket contains only 3 items of "Experimental"
```

And we add a new step definition and helper:
```java
// ShopStepDefinitions.java
    @Given("{actor} adds {int} items of {string} at ${int} each to her basket")
    public void bettyAddsToHerBasket(Actor actor, int itemCount, String itemName, int itemPrice) {
        actor.wasAbleTo(Open.browserOn().the(LandingPage.class));
        actor.attemptsTo(Shop.addItemsToBasket(itemCount, itemName, itemPrice));
    }

// Shop.java
    public static Performable addItemsToBasket(int itemCount, String itemName, int itemPrice) {
        return Task.where("adds " + itemName + " to basket",
                actor -> {
                    actor.remember("item price", itemPrice);
                    for (int i = 0; i < itemCount; i++) {
                        actor.attemptsTo(Click.on(ShopPage.ADD_ITEM.of(itemName)));
                    }
                }
        );
    }
```

And we are going to change the way we predict and check the subtotal for the `Then` step:
```java
    public static Question<Boolean> basketIsCorrect(int itemCount, String itemName) {
        return Question.about("basket contents").answeredBy(
                actor -> {
                    int itemPrice = actor.recall("item price");
                    String itemTotal = "$" + (itemCount * itemPrice) + ".00";
                    String basketContents = actor.recall("basket contents");
                    String subTotal = actor.recall("basket subtotal");

                    return basketContents.equals(itemName + "\nQuantity " + itemCount + "remove\n" + itemPrice)
                            && subTotal.equals(itemTotal);
                }
        );
    }
```

# Refactoring
We notice now that there is very little difference (just the addition of a loop) between `addItemToBasket()` and `addItemsToBasket()` helper methods.

Adding one item is just a special cases of adding multiple items, so we can remove both the redundant helper method `addItemToBasket()` and stepDefinition and update our original Scenario to:
```gherkin
  Scenario: Betty chooses "Unusable Security"
    Given Betty adds 1 item of "Unusable Security" at $2 each to her basket
    When she checks her basket
    Then she can see her basket contains only 1 item of "Unusable Security"
```

And allow the step to be written as "item" or "items"
```java
    @Given("{actor} adds {int} item/items of {string} at ${int} each to her basket")
    public void bettyAddsToHerBasket(Actor actor, int itemCount, String itemName, int itemPrice) {
        actor.wasAbleTo(Open.browserOn().the(LandingPage.class));
        actor.attemptsTo(Shop.addItemsToBasket(itemCount, itemName, itemPrice));
    }
```

So now we have only 3 step definitions to cover tests for single and multiple item purchases.

> ℹ️ It may be tempting to eliminate the single item test case here but it is always useful to prove that the simplest case works before moving onto more complicated examples
>
> ℹ️ If we only have complicated tests failing, it is easy to blame the test failures on their complexity, hiding the fact that the basic functionality may be broken - so always get evidence for that first
>
> ❗ Note the I have changed the item for the second test - we need to vary our test data to prove that the application can cope with more than example of the input data

# Mixing it up
Another check that baskets are created and totalled correctly is to add a mixture of different items.

For this requirement, we need a data structure to represent the customer's choices, e.g:

| Item | Count | Price |
| ---- | ----- | ----- |
| Docker Tooling | 1 | 8 |
| Docker Babies | 3 | 64 |
| Docker for Developers | 5 | 256 |

```gherkin
  Scenario: Betty chooses a mix of items
    Given Betty adds items to her basket
      | Item                  | Count | Price |
      | Docker Tooling        | 1     | 8     |
      | Docker Babies         | 3     | 64    |
      | Docker for Developers | 5     | 256   |
    When she checks her basket
    Then she can see her basket contains only her chosen items

```

We can see that we need new step definitions and ways of remembering the basket items to check the basket contents.

We will convert Cucumber's DataTable object to a List of Maps:
```java
    @Given("{actor} adds items to her basket")
    public void bettyAddsItemsToHerBasket(Actor actor, DataTable itemChoices) {
        List<Map<String, String>> choices = itemChoices.asMaps(String.class, String.class);
        actor.remember("choices", choices);
        actor.wasAbleTo(Open.browserOn().the(LandingPage.class));
        for (Map<String, String> choice: choices) {
            actor.attemptsTo(Shop.addItemsToBasket(
                    Integer.parseInt(choice.get("Count")),
                    choice.get("Item"),
                    Integer.parseInt(choice.get("Price"))
            ));
        }
    }
```
And we see that the correct items have been added:

![image](https://github.com/user-attachments/assets/f4348742-50f3-484f-b797-ee0bb82dfdf4)

At the moment, our `she checks her basket` step definition only stores the first row of the basket, so we need to update its helper method to get each row:
```java
    public static Performable openBasket() {
        return Task.where("opens basket",
                actor -> {
                    actor.attemptsTo(Click.on(ShopPage.CHECKOUT));
                    actor.remember("basket contents", Text.ofEach(CheckoutPage.BASKET_ITEMS));
                    actor.remember("basket subtotal", Text.of(CheckoutPage.SUB_TOTAL));
                }
        );
    }
```
Now when we ask for the basket contents later, we get an `ArrayList`:

![image](https://github.com/user-attachments/assets/05d8f659-327f-4378-aabe-86213e5a88b0)

And we want to compare this to our `choices` collection:

![image](https://github.com/user-attachments/assets/caff821f-ce52-499d-9a34-e46b4e271737)

So we refactor our `basketIsCorrect()` helper again to to fail if any of these conditions are false:
- The number of different items chosen matches the rows in the basket
- Each row matches the item, quantity and price we expect
- The basket subtotal equals the cost of all items

```java
    public static Question<Boolean> basketIsCorrect(List<Map<String, String>> choices) {
        return Question.about("basket contents").answeredBy(
                actor -> {
                    ArrayList<String> basketContents = actor.recall("basket contents");
                    if (basketContents.size() != choices.size()) {
                        return false;
                    }
                    int row = 0;
                    int totalPrice = 0;
                    for (Map<String, String> choice: choices) {
                        String basketRow = basketContents.get(row);
                        String itemName = choice.get("Item");
                        int itemCount = Integer.parseInt(choice.get("Count"));
                        int itemPrice = Integer.parseInt(choice.get("Price"));
                        totalPrice += itemCount * itemPrice;
                        if (!basketRow.equals(itemName + "\nQuantity " + itemCount + "remove\n" + itemPrice)) {
                            return false;
                        }
                        row++;
                    }
                    return actor.recall("basket subtotal").equals("$" + totalPrice + ".00");
                }
        );
    }

    public static Performable checkBasketContainsOnly(List<Map<String, String>> choices) {
        return Ensure.that("basket has expected Contents",
                        basketIsCorrect(choices))
                .isTrue();
    }
```

> ❗This test passes but we have now broken our existing tests
> 
> ✔️ Rather than create other versions of the helpers to cover the previous tests, we will refactor the scenarios
> 
> ✔️ This makes it clear that the scenarios are a sequence of increasingly complex test conditions

```gherkin
  Scenario: Betty chooses a single copy of "Unusable Security"
    Given Betty adds items to her basket
      | Item              | Count | Price |
      | Unusable Security | 1     | 2     |
    When she checks her basket
    Then she can see her basket contains only her chosen items

  Scenario: Betty chooses multiple copies of "Experimental"
    Given Betty adds items to her basket
      | Item         | Count | Price |
      | Experimental | 3     | 128   |
    When she checks her basket
    Then she can see her basket contains only her chosen items

  Scenario: Betty chooses a mix of items
    Given Betty adds items to her basket
      | Item                  | Count | Price |
      | Docker Tooling        | 1     | 8     |
      | Docker Babies         | 3     | 64    |
      | Docker for Developers | 5     | 256   |
    When she checks her basket
    Then she can see her basket contains only her chosen items
```
So now the scenarios have better names and are a clear progression that checks:
- an item is added to the basket correctly
- bulk item costs are calculated correctly
- mixed items are added to the basket correctly
- mixed item costs are calculated correctly

And this is clearly shown in the test report:

![image](https://github.com/user-attachments/assets/79ccb79d-804e-4496-8656-b5da8f74bf85)

> ✔️ We only have 3 step definitions and 4 helper methods to maintain to cover all these scenarios
>
> ✔️ In addition, we are only relying on 4 page elements, giving us some resilience against UI changes

# Closing thoughts

> ℹ️ When I originally started writing the tests, I scraped the item price from the website.
>
> ❗ This violates the _Do Not Trust The UI_ rule
>
> ✔️ All of your input test data should come from an indisputable source external to the application

> ❗ If the `basketIsCorrect()` check fails, it gives you no information about where the failure is
>
> ✔️ You might want to implement the checks as separate step definitions for more granular failure messages
